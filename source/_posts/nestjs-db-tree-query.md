---
title: nestjs-typeorm树形查询
date: 2023-09-01 20:30:39
categories:
- typescript
- nestjs
tags:
- typeorm
- tree
---
### 背景
树形结构是一种常用的存储查询模型，比如后台菜单或评论列表，展开就是一棵树，对于数据库的表来讲，意味着自引用，即有一个外键字段关联本表的主键，该外键字段一般称为`parent_id`，由该字段串联起整颗树的结构，在数据量不大的场景中，单纯一个`parent_id`即可胜任树形数据的查询，只需对全表数据循环遍历组装，这种方案在前者后台菜单中比较常用

当数据超过一定量级，一次性load全部数据并循环遍历对于io和cpu都是很大的负担也不现实，此时一般增加辅助字段来提升查询性能(空间换时间)，以及满足更多样性的查询需求。这里只介绍一种常用的方案-`path`，类似于unix的文件夹层级，该字段存储从根节点到该节点的整条路径，比如`100.120.200.`：代表这棵树根节点为100，该节点为200，该节点的父节点为120，最大的好处是比单纯的`parent_id`更容易获取任意节点的所有子节点

很多orm都内置了树形模型方案，包括typeorm，具体可查看文档[Tree Entities](https://typeorm.io/tree-entities)，而`Materialized Path`就是上述`path`方案的实现，开箱即用

### 问题
不同场景有不同的方案需求，我们限定当前的需求是评论列表，结合当前各大平台如知乎的交互方案，更细化的需求是**二级评论列表(非完整树形)，第一层为所有的一级评论，一级评论分页加载所有的子评论**

继续看typeorm提供的查询方案，有几个相关的方法：`findTrees`、`findRoots`、`findDescendants`、和`findDescendantsTree`：
* `findTrees`：一次性返回所有的评论树
* `findRoots`：一次性返回所有的根节点
* `findDescendants`/`findDescendantsTree`：返回一棵树
<!-- more -->
貌似无法直接处理子评论分页加载的逻辑？

### 方案
通过`Materialized Path`生成的表结构可看到字段`mpath`，同样为了便于直接查询出节点的父节点，增加一个字段`parentId`，整个Entity的关键代码如下：

```typescript
# comment.entity.ts
import {
    BaseEntity,
    TreeParent,
    Tree,
    Entity,
} from 'typeorm';

@Tree('materialized-path')
@Entity('content_comments')
export class CommentEntity extends BaseEntity {
    @TreeParent({ onDelete: 'CASCADE' })
    parent: CommentEntity | null;
}
```

目前的问题是单纯根据`typeorm`暴露的方法无法对子节点进行分页加载，我们的解决方案围绕隐藏字段`mpath`进行:
1. 查询当前页的根节点列表
2. 根据`mpath`字段查询出上述根节点的子节点列表(记得CommentEntity也需要添加该字段)

关键代码如下：
```typescript
@Injectable()
export class CommentService {
    protected repository: CommentEntity;
    constructor(protected repository: CommentRepository) {
        this.repository = repository;
    }

    /**
     * 查询一级评论，附带固定数量的二级评论(可自定义排序规则)
     */
    async getPostComments(
        post: PostEntity,
        loginUserId: string | null,
        page = 1,
        limit = 10,
        childrenCommentCount = 3,
    ) {
        // 1. 查询当前页的根节点列表
        const query = CommentEntity.createQueryBuilder('comment')
            .where(`comment.postId = :postId`, { postId: post.id })
            .andWhere('comment.parentId IS NULL')
            .orderBy('comment.id', 'DESC')
            .skip((page - 1) * limit)
            .take(limit);
        const data = await query.getManyAndCount();

        const rootCommentIds = data[0].map((v: CommentEntity) => v.id);
        if (rootCommentIds.length > 0) {
            // 2. 根据`mpath`字段查询出上述根节点的子节点列表
            const sql = `SELECT id, p FROM (
                SELECT *, substring(mpath, 1, instr(mpath, '.')) as p, ROW_NUMBER() OVER (PARTITION BY substring(mpath, 1, instr(mpath, '.')) ORDER BY id DESC) AS n
                FROM content_comments
                where parentId is not null and (${rootCommentIds
                    .map((v: string) => ` mpath like '${v}.%' or `)
                    .join('')} false)
            ) AS x WHERE n <= ${childrenCommentCount}`;
            const children = await CommentEntity.getRepository().manager.query(sql);
            const rootChildrenIds = groupBy(children, 'p');
            const childrenComments = keyBy(
                await CommentEntity.find({
                    where: {
                        id: In(children.map((v: CommentEntity) => v.id)),
                    },
                    relations: ['user', 'parent', 'parent.user'],
                    order: {
                        id: 'DESC',
                    },
                }),
                'id',
            );
            data[0] = data[0].map((v) => {
                const k = `${v.id}.`;
                v.children = [];
                if (!isNil(rootChildrenIds[k]) && rootChildrenIds[k].length !== 0) {
                    v.children = rootChildrenIds[k].map((c: CommentEntity) => {
                        return childrenComments[c.id];
                    });
                }
                return v;
            });
        }

        // 其他处理略
        return $data;
    }
}
```
关键逻辑在第2步，使用`mysql`的**窗口函数**和`mpath`字段获取固定数量的自节点

但是执行会报错，因为`mpath`是隐藏字段，不可直接对其做查询操作，需要设置为可见：
```typescript
@Injectable()
export class CommentService {
    protected repository: CommentEntity;
    constructor(protected repository: CommentRepository) {
        // 设置mpath可见
        repository.metadata.columns = repository.metadata.columns.map((x) => {
            if (x.databaseName === 'mpath') {
                x.isVirtual = false;
            }
            return x;
        });
        this.repository = repository;
    }
}
```

查询速度取决于数据量和窗口函数的性能，后续的优化方案比如`mpath`加索引等不再赘述；当数据量继续增长时单纯使用`mysql`也是远远不够的，需要引入其他性能更高的技术栈，也不在本文讨论范围

### 尾声
欢迎批评指正出文中的错误及不合理之处，欢迎评论交流