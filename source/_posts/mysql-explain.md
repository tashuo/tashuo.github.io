---
title: MySQL的explain文档
date: 2017-08-04 15:12:48
categories:
- 笔记
tags:
- mysql
---

文档原地址：[EXPLAIN Output Format](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)

> EXPLAIN提供一些MySQL执行SQL语句的信息，包括SELECT, DELETE, INSERT, REPLACE和UPDATE等语句。
> 
> **MySQL 5.6.3之前只支持SELECT**。

## 简介
EXPLAIN作用于SELECT语句时会返回一列信息。SELECT语句会按照一定顺序输出表的内容。MySQL使用循环嵌套的方式处理所有的连表操作。这意味着MySQL会从第一张表中读取一行数据，然后在第二张表中读出相匹配的数据，然后第三张表等等。当所有的表都被执行到，MySQL会输出选中的列并且回溯表的数据直到找到匹配更多行的表。从找到的这个表里面读取下一行数据，然后下一个表。

EXPLAIN的输出包含不同的分割开来的数据。另外，对于SELECT语句，EXPLAIN还会生成一些额外的信息与`SHOW WARNING`的输出一起显示。

## 输出内容
下面介绍`EXPLAIN`命令的输出内容，稍后会补充`type`,`Extra`两个字段的信息。

`EXPLAIN`输出的每行信息都对应于相应的一张表。每行输出的内容都总结在下文的表格中，表格最后一列是更加详细的说明。表格第一列是字段名，第二列是当加了`FORMAT=JSON`参数时跟第一列相同概念的字段名。
<!-- more -->
**Table 8.1 EXPLAIN Output Columns**

Column       |JSON Name     |Meaning
-----        |---------     |-------
id           |select_id     |`SELECT`操作的唯一标识
select_type  |None          |`SELECT`类型
table        |table_name    |操作的表名
partitions   |partitions    |匹配的分区
type         |access_type   |联表类型
possible_keys|possible_keys |可能用到的索引
key          |key           |用到的索引
key_len      |key_length    |用到的索引长度
ref          |ref           |与索引比较的值
rows         |rows          |预估查询的列数
filtered     |filtered      |被条件过滤掉的百分比
Extra        |None          |附加信息

<br/>
> **注意**
> 
> JSON格式的NULL值属性不会输出

<br/>

* id (JSON Name: select_id)

    该`SELECT`语句的唯一标识。这个数字是有序产生的。如果数据是其他查询union而来的话这个值可以为null。在这种情况喜爱，`table`属性的值将会是`<union M,N>`格式，用来表明数据是由M和N查询组合而成。

* select_type (JSON Name: None)

    `SELECT`操作的类型，可以是下表中的任意值。JSON格式的输出会将该字段值作为`query_block`输出，除非是`SIMPLE`或`PRIMARY`类型。JSON格式的名称同样在表中体现。

    `DEPENDENT`通常表示关联子查询的使用，查看文档 [关联子查询](https://dev.mysql.com/doc/refman/5.7/en/correlated-subqueries.html)

    `DEPENDENT SUBQUERY`同`UNCACHEABLE SUBQUERY`的评估是不同的。对于`DEPENDENT SUBQUERY`，对于每组从外部查询上下文中而来的变量的不同值，子查询仅重新评估一次。而`UNCACHEABLE SUBQUERY`对外部查询上下文的每一行都需要重新评估子查询。

    子查询的可缓存性与常说的sql查询缓存不一样([查询缓存](https://dev.mysql.com/doc/refman/5.7/en/query-cache-operation.html))。子查询缓存发生在语句执行期间，而常说的查询缓存则是在语句执行完毕后存储结果。

    当使用JSON格式输出EXPLAIN结果时没有单独的`select_type`字段，而是输出另一个`query_block`字段。相当于大多数子查询的类型（一个示例 MATERIALIZED的materialized_from_subquery），并在适当时显示。SIMPLE和PRIMARY两种类型没有对应的JSON格式的值。

    在非SELECT语句中`select_type`的值展示的是语句类型。如`DELETE`语句的`select_type`的值是delete。

select_type         | JSON Name         | Meaning
----                | ----              | ----
SIMPLE              | None              | 没有连表或自查询的简单查询
PRIMARY             | None              | 最外层的查询
UNION               | None              | 第二个或之后被union的查询
DEPENDENT UNION     | dependent (true)  | 依赖于外部查询结果的第二个或之后被union的查询
UNION RESULT        | union_result      | union而来的查询结果
SUBQUERY            | None              | 第一个子查询
DEPENDENT SUBQUERY  |  dependent (true) | 依赖于外部查询结果的第一个个子查询
DERIVED             | None              | `from`语句中的查询
MATERIALIZED        | materialized_from_subquery  | 子查询
UNCACHEABLE SUBQUERY| cacheable (false) | 不能缓存结果，并且必须对外部查询获取的每一行进行重新评估的子查询
UNCACHEABLE UNION   | cacheable (false) | 第二个或之后一个属于不可缓存结果子查询的UNION的查询 （见 UNCACHEABLE SUBQUERY）


* table (JSON Name: table_name)

    操作到的表名，可以是以下的任何值
 
    * <unionM,N\>: 数据是由其他几个查询union而来, M和N为查询id
   
    * <derivedN\>: id为N的查询中from子句是一个select语句，而当前查询的from子句是id为N的查询结果
   
    * <subqueryN\>: The row refers to the result of a materialized subquery for the row with an id value of N. See Section 8.2.2.2, “Optimizing Subqueries with Materialization”.

* partitions (JSON Name: partitions)

    语句操作到的分区。无分区的表将会返回null。

* type (JSON Name: access_type)
    
    联表类型，下文会有详细介绍。

* possible_keys (JSON Name: possible_keys)

    possible_keys指出MySQL可以选择查询时用的所有索引。不过独立于实际输出的顺序。这意味着其中的索引并不一定会用到。
 
    如果此列是NULL（或未经JSON格式输出定义），则不存在相关索引。在这种情况下，您就可以通过检查WHERE子句来检查是否引用了适用于索引的某个或某些列通过建立索引来优化性能。

    要查看表的索引，请使用。 SHOW INDEX FROM tbl_name。

* key (JSON Name: key)

    该列指出MySQL实际使用的索引。如果MySQL决定使用`possible_keys`中其中一个索引来查询数据，这个索引就会出现这一列。
 
    该列的值没有在`possible_keys`中是有可能的。这发生在`possible_keys`中所有的索引都不适合用来查表，但是选取的列又适用于其他的索引。也就是说，该索引作用于选中的列，虽然它并不用于确定要检索的行，但索引扫描效率要高于实际数据扫描。

    对于Innodb来说，一个辅助索引可能处理选中的列，即使MySQL也选择了主键，由于Innodb聚簇索引的特性，所有的辅助索引都依赖于主键。如果该列的值为null，是因为MySQL没有找到索引用来提高sql执行效率。

    让MySQL强制使用或不使用某个索引可以通过参数`FORCE INDEX`,` USE INDEX`或`IGNORE INDEX`实现。具体可查看[文档](https://dev.mysql.com/doc/refman/5.7/en/index-hints.html)。

    对于MyISAM类型的表，可通过执行`ANALYZE TABLE`来帮助选择合适的索引用来提升性能，`myisamchk --analyze`也可以。

* key_len (JSON Name: key_length)
    
    该字段标识MySQL决定使用该索引的长度, 可以帮助查明MySQL实际使用了一个多列索引的哪些部分。如果字段`key`的值为null，那么该字段的值也为null。

    由于索引的实际存储结构，可以为null的字段要比不可以为null字段索引长度长。

* ref (JSON Name: ref)

    该字段展示的是哪一列或常量拿来与上面`key`字段中的索引做比较用于从表中取数据。

    如果值是`func`，说明该列使用的是某个函数的值。要查看是哪个函数，可以在`explain`命令后使用 `SHOW WARNING` 参数获取。这个函数很可能是算数运算符等运算。

* rows (JSON Name:rows)

    该字段标识MySQL认为执行这条sql语句需要查询到的行数。

    对于`Innodb`引擎数据库的表，这个值只是一个估值，也很可能是不准确的。

* filtered (JSON Name: filtered)

    该字段标识将由where语句过滤掉的行数占总行数的百分比估值。因此，`rows`字段展示查询到行数的估值而 `rows`*`filtered`/100 的值就是将被联接到前面表的行数。

* Extra (JSON Name: none)

    该字段展示MySQL如果执行SQL语句的附加信息。下文会介绍不同值的含义。

    JSON格式的输出没有一个对应于该字段的字段。然而这些值可能会出现在另外一些JSON属性中或作为`message`属性的值输出。


### type(联表/查询类型)
`type`字段展示不同表之间如何联接，对应于JSON格式中的`access_type`属性。下面列出不同的值类型，从优到劣的排序。

* system

    该表只有一行，是下面`const`类型的特殊情况。

* const

    该表最多只匹配到一行，并且在查询的开始就命中。因为只有一行，所以该行的数据对于MySQL查询优化器可以看作是常量。`const`类型非常高效因为只读一次。

    该类型出现在使用一个常量与主键或唯一键比较时。下面的例子中，`tbl_name`表将作为`const`类型的表使用:

    ```sql
        SELECT * FROM tbl_name WHERE primary_key=1;

        SELECT * FROM tbl_name
        WHERE primary_key_part1=1 AND primary_key_part2=2;
    ```

* eq_ref

    数据是通过循环对比前面表中的值获取。除了`system`和`const`，这个是最高效的联表类型。该类型出现在联表使用的索引都是`主键`或`非NULL值唯一键`时。

    `eq_ref`用于索引列使用‘=’运算符的场景。比较的值可以是常量也可以是前面的表中读出的值。下面是示例:

    ```sql
        SELECT * FROM ref_table, other_table
            WHERE ref_table.key_column=other_table.column;

        SELECT * FROM ref_table, other_table
            WHERE ref_table.key_column_part1=other_table.column
            AND ref_table.key_column_part2=1;
    ```

* ref

    当与前面表中取出的数据对比时都能命中索引。该类型出现在联表时使用了前缀索引或索引不是主键或唯一键(即不能跟根据索引命中唯一一行数据)。对于一个数据量不大的表来说，该类型效率也比较高。

    `ref`用于索引列使用‘=’、‘<=’和‘>=’运算符的场景。下面是示例:

    ```sql
        SELECT * FROM ref_table WHERE key_column=expr;

        SELECT * FROM ref_table, other_table
            WHERE ref_table.key_column=other_table.column;

        SELECT * FROM ref_table, other_table
            WHERE ref_table.key_column_part1=other_table.column
            AND ref_table.key_column_part2=1;
    ```


* fulltext

    使用全文索引时的联表类型。

* ref_or_null

    类似于`ref`类型，但是多一个NULL值的判断。常用于多条件查询，示例如下:

    ```sql
        SELECT * FROM ref_table
            WHERE key_column=expr OR key_column IS NULL;
    ```

* index_merge

    该类型说明MySQL使用了[索引合并优化](https://dev.mysql.com/doc/refman/5.7/en/index-merge-optimization.html)。这种情形下，`key`字段的值是使用的索引列表，`key_len`的值是所使用索引的最长关键部分的列表。

* unique_subquery

    该类型是在含有某些类似下面示例中IN条件查询时对于`eq_ref`类型的补充:

    ```sql
        value IN (SELECT primary_key FROM single_table WHERE some_expr)
    ```

    该类型只是一种索引查询功能，用于替换掉原有的子条件查询以获取更高的性能。

* index_subquery

    该类型类似于`unique_subquery`，唯一不同的是IN子查询的索引不一定是NOT NULL。

* range

    使用索引时仅在给定的范围查找。这种情况下，`key`字段的值展示使用的索引，`key_len`是实际使用到索引最长的关键部分的长度，`ref`字段是NULL。

    当索引列跟一个常量做 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, 或 IN() 其中任意运算时该类型将会被使用。示例如下:

    ```sql
        SELECT * FROM tbl_name
            WHERE key_column = 10;

        SELECT * FROM tbl_name
            WHERE key_column BETWEEN 10 and 20;

        SELECT * FROM tbl_name
            WHERE key_column IN (10,20,30);

        SELECT * FROM tbl_name
            WHERE key_part1 = 10 AND key_part2 IN (10,20,30);
    ```

* index

    该类型等同于下面的`ALL`类型，只是索引树被扫描。在两种情形下会发生:

    * 如果索引是查询的覆盖索引并且可以用于查询到表中所有需要的数据，索引树就会被扫描。这种情况下，`Extra`字段的值将是`Using index`。索引扫描会比全表扫描快因为索引要比实际数据小。
    * 利用索引做全表扫描用以索引的排序查询数据。这种情况下`Extra`字段的值不是`Using index`。

* ALL

    全表扫描用于先前表返回每行数据的联合查询。如果该表是第一个未被标记为`const`类型的表，那这种情况通常是低效的，并且在其它任何情形下也是不好的。一般可以通过增加索引来检索基于常量或先前的表返回的数据用以避免全表扫描这种类型。

### Extra(附件信息)
该字段输出一些关于MySQL实际如何执行语句的额外信息，下文会介绍该字段可能出现的值以及相应的JSON格式的输出。

如果你希望SQL语句执行的更快，可以去探究输出了`Using filesort`或`Using temporary`值的该字段，对应于JSON格式则using_filesort和using_temporary_table属性的值都为true。

* Child of 'table' pushed join@1 (JSON: message text)

* const row not found (JSON property: const_row_not_found)

* Deleting all rows (JSON property: message)

* Distinct (JSON property: distinct)

* FirstMatch(tbl_name) (JSON property: first_match)

* Full scan on NULL key (JSON property: message)

* Impossible HAVING (JSON property: message)

* Impossible WHERE (JSON property: message)

* Impossible WHERE noticed after reading const tables (JSON property: message)

* LooseScan(m..n) (JSON property: message)

* No matching min/max row (JSON property: message)

* no matching row in const table (JSON property: message)

* No matching rows after partition pruning (JSON property: message)

* No tables used (JSON property: message)

* Not exists (JSON property: message)

* Plan isn't ready yet (JSON property: none)

* Range checked for each record (index map: N) (JSON property: message)

* Scanned N databases (JSON property: message)

* Select tables optimized away (JSON property: message)

* Skip_open_table, Open_frm_only, Open_full_table (JSON property: message)

* Start temporary, End temporary (JSON property: message)

* unique row not found (JSON property: message)

* Using filesort (JSON property: using_filesort)

    为了确定如何按照一定的排序检索数据，MySQL必须做一个额外的转换。排序通过遍历所有行和存储所有匹配WHERE子句条件的行的键和指针完成。这些键是有序的，然后所有数据也会按照一定的顺序来检索。

* Using index (JSON property: using_index)
    
    仅使用索引树来检索表中的数据，而不需要进行额外的对实际数据的搜索。当查询只使用作为单个索引一部分的列时会使用此策略。

    对于含有用户自定义聚簇索引的Innodb表，即使该字段不是`Using index`值也可能使用索引，这种情形出现在`type`字段的值是index并且`key`字段的值是PRIMARY时。

* Using index condition (JSON property: using_index_condition)

* Using index for group-by (JSON property: using_index_for_group_by)

* Using join buffer (Block Nested Loop), Using join buffer (Batched Key Access) (JSON property: using_join_buffer)

* Using MRR (JSON property: message)

* Using sort_union(...), Using union(...), Using intersect(...) (JSON property: message)

* Using temporary (JSON property: using_temporary_table)

* Using where (JSON property: attached_condition)

* Using where with pushed condition (JSON property: message)

* Zero limit (JSON property: message)


### TODO














