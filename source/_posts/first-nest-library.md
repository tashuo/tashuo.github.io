---
title: 从零开始编写Nest库
date: 2023-04-23 18:00:58
categories:
- typescript
- nestjs
tags:
- npm
- jest
- javascript
---
### 十万个为什么
`nest`库是什么？`nest`是一个`typescript`构建的`node`框架，那`nest`库可以理解为依赖`nest`框架的`node`库？`node`是一个`javascript`服务端的运行环境，那又要怎样用`typescript`编写`javascript`库呢？做为一只从其他语言(`js`几乎零基础只会简单的`jQuery`)转来学习`nest`的菜鸟刚开始总会在某些时刻冷不丁得被这些概念绕的云里雾里，不过是因为直接跨过`js`、`node`、`ts`直接啃`nest`的原因，底层建筑不牢固直接搬砖终究是雾里看花，而从零手撸一个`nest`库不妨可以视为一个敲门砖，可以带我们看一探`js`世界的冰山一角.

### 撸起袖子开干
#### 给它一个家
`nest`库终究也是一个独立的项目，和其他开源库一样我们使用`github`来做版本管理，新建一个空白仓库`clone`到本地.

```bash
$: git clone https://github.com/tashuo/nestjs-config
$: cd nestjs-config
```

#### 给它一支指挥棒
> `npm` is the world's largest software registry. Open source developers from every continent use npm to share and borrow packages, and many organizations use npm to manage private development as well.
<!-- more -->
*本项目使用`npm`做为`js`的依赖管理工具*

`npm`使用`package.json`作为整个项目的配置，包括包名、版本号、依赖、测试等等，当前直接初始化配置文件，初始化过程中可以交互式的填入一些基本配置，也可以一路回车后面手动编辑配置.

```bash
$: npm init
```

具体配置可参考：[https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html](https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html)

`ts`项目需要重点关注`main`和`types`配置，当前项目的场景只需要配置`main`为`dist/index.js`，因为`dist`目录编译出的`index.d.ts`也可用于声明导出的文件
> If your package has a main .js file, you will need to indicate the main declaration file in your package.json file as well. Set the types property to point to your bundled declaration file.
> 
> Also note that if your main declaration file is named index.d.ts and lives at the root of the package (next to index.js) you do not need to mark the types property, though it is advisable to do so.

#### 告诉它如何变身
> `TypeScript` is a language that is a superset of `JavaScript`: JS syntax is therefore legal TS.

`typescript`作为`javascript`的超集并不能直接运行在浏览器或`node`运行时中，`ts`把`js`封装得连它妈(浏览器、`node`)都不认识了，要想进门必须把这身皮脱掉，这个活儿由`tsc`负责，`tsc`把ts编译为原生的`js`代码，而说到编译就会涉及到不同的编译参数及配置，`tsconfig.json`挺身而出.
> The presence of a tsconfig.json file in a directory indicates that the directory is the root of a TypeScript project. The tsconfig.json file specifies the root files and the compiler options required to compile the project.

具体配置可参考 [https://www.typescriptlang.org/docs/handbook/tsconfig-json.html](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)

#### 告诉它存在的意义
本项目只实现一个简易的`nest`配置模块，项目启动时自动获取指定**config**目录的所有配置，这份配置可以在项目任何其他模块使用. 

为什么会存在依赖`nest`的库呢，比如像`lodash`、`axios`这些库可以直接在`nest`中使用，并不依赖于`nest`(它们在江湖上大杀四方的时候`nest`还在吃奶吧)，那我们要写的这个库可以这么酷吗？不好意思这个头必须得低，因为有求于人，我们要在项目启动时初始化所有配置，而且配置获取的方式也依赖框架(provider)，它们那么酷是因为没有包袱，如果你也能对社会对外界对欲望少一些依赖，你也可以这么酷:)

如前文所讲，我们要依赖`nest`框架的启动来加载配置，需要引入**动态模块**的[概念](https://docs.nestjs.com/fundamentals/dynamic-modules)，提供一个`ConfigModule`用于`import`，提供一个`ConfigService`作为`Provider`供其他模块引入使用，所有代码均在项目的[`src`目录下](https://github.com/tashuo/nestjs-config/tree/main/src).


#### 告诉它要靠谱
> Jest is a delightful JavaScript Testing Framework with a focus on simplicity.

> It works with projects using: Babel, TypeScript, Node, React, Angular, Vue and more!

俗话说得好，没有提供单元测试的代码都是耍花枪，而且编写单元测试时总会时不时得告诉你的代码有多屎，比如现在写的`ConfigService`无法脱离`ConfigModule`单独测试，因为`constructor`依赖了从`ConfigModule`注入的参数，所以出现了三个单元测试文件的奇观，如果够优雅的话一个文件就可以覆盖所有的测试，后面再优化，下次一定.

单元测试代码在[`test`目录](https://github.com/tashuo/nestjs-config/tree/main/test)

#### 告诉它做一个对社会有用的人
项目开发完需要发布到公有的`npm`仓库才可以使用，官方仓库是`https://registry.npmjs.org/`，国内一般使用淘宝、清华或其他境内的源，在进行发布流程中如果设置了非官方源，**需要在下面执行命令时显式指定官方的源**(--registry https://registry.npmjs.org)

1. 编译代码(`$: tsc`)
1. 注册npm账号(https://www.npmjs.com/)
2. 登录(`$: npm login --registry https://registry.npmjs.org)`)
3. 发布到远程仓库(`$: npm publish --registry https://registry.npmjs.org)`)

需要注意的是，`npm`直接使用`package.json`中的`name`作为全局包名，如果包名已存在就会报错.

本来以为像`php`的`composer`一样包名会带前缀，起码没那么容易冲突，结果`npm`这操作整的，会不会很多很屌的包名已经被人恶意占坑了.


### 仓库
所有的代码都在[https://github.com/tashuo/nestjs-config](https://github.com/tashuo/nestjs-config)，欢迎提出问题一起交流
