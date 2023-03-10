---
title: 数据库事务与并发控制
date: 2017-06-21 16:28:20
categories:
- 笔记
tags:
- mysql
- acid
---

### 数据库事务
> 数据库管理系统执行过程中的一个逻辑单位，由一个有限的数据库操作序列构成.

一个数据库事务通常包含了一个序列的对数据库的读/写操作。它的存在包含有以下两个目的：

* 为数据库操作序列提供了一个从失败中恢复到正常状态的方法，同时提供了数据库即使在异常状态下仍能保持一致性的方法。
* 当多个应用程序在并发访问数据库时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰。

当事务被提交给了DBMS（数据库管理系统），则DBMS（数据库管理系统）需要确保该事务中的所有操作都成功完成且其结果被永久保存在数据库中，如果事务中有的操作没有成功完成，则事务中的所有操作都需要被回滚，回到事务执行前的状态;同时，该事务对数据库或者其他事务的执行无影响，所有的事务都好像在独立的运行。
 
但在现实情况下，失败的风险很高。在一个数据库事务的执行过程中，有可能会遇上事务操作失败、数据库系统/操作系统失败，甚至是存储介质失败等情况。这便需要DBMS对一个执行失败的事务执行恢复操作，将其数据库状态恢复到一致状态（数据的一致性得到保证的状态）。为了实现将数据库状态恢复到一致状态的功能，DBMS通常需要维护**事务日志(基于日志的REDO/UNDO机制)**以追踪事务中所有影响数据库数据的操作。

#### ACID
* 原子性（Atomicity）：事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行。
* 一致性（Consistency）：事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束。
* 隔离性（Isolation）：多个事务并发执行时，一个事务的执行不应影响其他事务的执行。
* 持久性（Durability）：已被提交的事务对数据库的修改应该永久保存在数据库中。
<!-- more -->
#### 示例
用一个常用的“A账户向B账号汇钱”的例子来说明如何通过数据库事务保证数据的准确性和完整性。熟悉关系型数据库事务的都知道从帐号A到帐号B需要6个操作：

1. 从A账号中把余额读出来（500）。
2. 对A账号做减法操作（500-100）。
3. 把结果写回A账号中（400）。
4. 从B账号中把余额读出来（500）。
5. 对B账号做加法操作（500+100）。
6. 把结果写回B账号中（600）。

##### 原子性
保证1-6所有过程要么都执行，要么都不执行。一旦在执行某一步骤的过程中发生问题，就需要执行回滚操作。 假如执行到第五步的时候，B账户突然不可用（比如被注销），那么之前的所有操作都应该回滚到执行事务之前的状态。

##### 一致性
在转账之前，A和B的账户中共有500+500=1000元钱。在转账之后，A和B的账户中共有400+600=1000元。也就是说，数据的状态在执行该事务操作之后从一个状态改变到了另外一个状态。同时一致性还能保证账户余额不会变成负数等。

##### 隔离性
在A向B转账的整个过程中，只要事务还没有提交（commit），查询A账户和B账户的时候，两个账户里面的钱的数量都不会有变化。
如果在A给B转账的同时，有另外一个事务执行了C给B转账的操作，那么当两个事务都结束的时候，B账户里面的钱应该是A转给B的钱加上C转给B的钱再加上自己原有的钱。

##### 持久性
一旦转账成功（事务提交），两个账户的里面的钱就会真的发生变化（会把数据写入数据库做持久化保存）

### 读现象
> ANSI/ISO SQL 92标准涉及三种不同的一个事务读取另外一个事务可能修改的数据的“读现象”。
#### 脏读
当一个事务允许读取另外一个事务修改但未提交的数据时，就可能发生脏读。
#### 不可重复读
在一次事务中，当一行数据获取两遍得到不同的结果表示发生了“不可重复读”。
#### 幻影读
在事务执行过程中，当两个完全相同的查询语句执行得到不同的结果集。

### 隔离级别
在数据库事务的ACID四个属性中，隔离性是一个最常放松的一个。为了获取更高的隔离等级，数据库系统的锁机制或者多版本并发控制机制都会影响并发。 应用软件也需要额外的逻辑来使其正常工作。

#### 可序列化
最高的隔离级别。
在基于锁机制并发控制的DBMS实现可序列化，要求在选定对象上的读锁和写锁保持直到事务结束后才能释放。在SELECT 的查询中使用一个“WHERE”子句来描述一个范围时应该获得一个“范围锁”（range-locks）。这种机制可以避免“幻影读”（phantom reads）现象（详见下文）。
当采用不基于锁的并发控制时不用获取锁。但当系统探测到几个并发事务有“写冲突”的时候，只有其中一个是允许提交的。这种机制的详细描述见“快照隔离”
#### 可重复读
在可重复读（REPEATABLE READS）隔离级别中，基于锁机制并发控制的DBMS需要对选定对象的读锁（read locks）和写锁（write locks）一直保持到事务结束，但不要求“范围锁”，因此可能会发生“幻影读”。
#### 提交读
在提交读（READ COMMITTED）级别中，基于锁机制并发控制的DBMS需要对选定对象的写锁一直保持到事务结束，但是读锁在SELECT操作完成后马上释放（因此“不可重复读”现象可能会发生，见下面描述）。和前一种隔离级别一样，也不要求“范围锁”。
#### 未提交读
未提交读（READ UNCOMMITTED）是最低的隔离级别。允许“脏读”（dirty reads），事务可以看到其他事务“尚未提交”的修改。
通过比低一级的隔离级别要求更多的限制，高一级的级别提供更强的隔离性。标准允许事务运行在更强的事务隔离级别上。(如在可重复读隔离级别上执行提交读的事务是没有问题的)

### 并发控制
并发控制描述了数据库事务隔离以保证数据正确性的机制。为了保证并行事务执行的准确执行，数据库和存储引擎在设计的时候着重强调了并发控制这一点。典型的事务相关机制限制数据的访问顺序（执行调度）以满足可序列化 和可恢复性。限制数据访问意味着降低了执行的性能，并发控制机制就是要保证在满足这些限制的前提下提供尽可能高的性能。经常在不损害正确性的情况下，为了达到更好的性能，可序列化的要求会减低一些，但是为了避免数据一致性的破坏，可恢复性必须保证。

两阶段锁是关系数据库中最常见的提供了可序列化和可恢复性的并发控制机制，为了访问一个数据库对象，事务首先要获得这个对象的锁。对于不同的访问类型（如对对象的读写操作）和锁的类型，如果另外一个事务正持有这个对象的锁，获得锁的过程会被阻塞或者延迟。

### 锁机制
封锁是一项用于多用户同时访问数据库的技术，是实现并发控制的一项重要手段，能够防止当多用户改写数据库时造成数据丢失和损坏。当有一个用户对数据库内的数据进行操作时，在读取数据前先锁住数据，这样其他用户就无法访问和修改该数据，直到这一数据修改并写回数据库解除封锁为止。

封锁也有不同的分类与概念:

#### 使用方式区分
##### 乐观并发控制(乐观锁)
> 在关系数据库管理系统里，乐观并发控制（又名“乐观锁”，Optimistic Concurrency Control，缩写“OCC”）是一种并发控制的方法。它假设多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，正在提交的事务会进行回滚。

** 乐观并发控制多数用于数据争用不大、冲突较少的环境中，这种环境中，偶尔回滚事务的成本会低于读取数据时锁定数据的成本，因此可以获得比其他并发控制方法更高的吞吐量。 **

相对于悲观锁，在对数据库进行处理的时候，乐观锁并不会使用数据库提供的锁机制。一般的实现乐观锁的方式就是记录**数据版本**。

数据版本
>为数据增加的一个版本标识。当读取数据时，将版本标识的值一同读出，数据每更新一次，同时对版本标识进行更新。当我们提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的版本标识进行比对，如果数据库表当前版本号与第一次取出来的版本标识值相等，则予以更新，否则认为是过期数据。

实现数据版本有两种方式，第一种是使用**版本号**，第二种是使用**时间戳**。

###### 版本号实现乐观锁
使用版本号时，可以在数据初始化时指定一个版本号，每次对数据的更新操作都对版本号执行+1操作。并判断当前版本号是不是该数据的最新的版本号。

``` sql
// 1.查询出商品信息
select (status,status,version) from t_goods where id=#{id}
// 2.根据商品信息生成订单
// 3.修改商品status为2
update t_goods set status=2, version=version+1 where id=#{id} and version=#{version};
```

###### 时间戳实现乐观锁
同版本号类似，增加一列updated_at字段标识该行数据修改的时间，修改前读出时间戳，更新时update 时间戳并在where中判断时间戳是否为开始取出的值。

###### 优点与不足

乐观并发控制相信事务之间的数据竞争(data race)的概率是比较小的，因此尽可能直接做下去，直到提交的时候才去锁定，所以不会产生任何锁和死锁。但如果直接简单这么做，还是有可能会遇到不可预期的结果，例如**两个事务都读取了数据库的某一行，经过修改以后写回数据库，这时就遇到了问题**(这种酌情可用悲观锁处理解决)。

##### 悲观并发控制(悲观锁)
> 在关系数据库管理系统里，悲观并发控制（又名“悲观锁”，Pessimistic Concurrency Control，缩写“PCC”）是一种并发控制的方法。它可以阻止一个事务以影响其他用户的方式来修改数据。如果一个事务执行的操作读某行数据应用了锁，那只有当这个事务把锁释放，其他事务才能够执行与该锁冲突的操作。

悲观并发控制主要用于数据争用激烈的环境，以及发生并发冲突时使用锁保护数据的成本要低于回滚事务的成本的环境中。

###### 悲观锁加锁流程
1. 在对任意记录进行修改前，先尝试为该记录加上排他锁（exclusive locking）。
2. 如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。 具体响应方式由开发者根据实际需要决定。
3. 如果成功加锁，那么就可以对记录做修改，事务完成后就会解锁了。
4. 其间如果有其他对该记录做修改或加排他锁的操作，都会等待我们解锁或直接抛出异常。

###### Innodb悲观锁
> 要使用悲观锁，我们必须关闭mysql数据库的自动提交属性，因为MySQL默认使用autocommit模式，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。set autocommit=0;

``` sql
//0.开始事务(三者选一就可以)
begin;/begin work;/start transaction; 
//1.查询出商品信息
select status from t_goods where id=1 for update;
//2.根据商品信息生成订单
insert into t_orders (id,goods_id) values (null,1);
//3.修改商品status为2
update t_goods set status=2;
//4.提交事务
commit;/commit work;
```

上面的查询语句中，我们使用了**select…for update**的方式，这样就通过开启排他锁的方式实现了悲观锁。此时在t_goods表中，id为1的 那条数据就被我们锁定了，其它的事务必须等本次事务提交之后才能执行。这样我们可以保证当前的数据不会被其它事务修改。

> 上面我们提到，使用select…for update会把数据给锁住，不过我们需要注意一些锁的级别，MySQL InnoDB默认行级锁。行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住，这点需要注意。

###### 优点与不足
悲观并发控制实际上是“先取锁再访问”的保守策略，为数据处理的安全提供了保证。但是在效率方面，处理加锁的机制会让数据库产生额外的开销，还有增加产生死锁的机会；另外，在只读型事务处理中由于不会产生冲突，也没必要使用锁，这样做只能增加系统负载；还有会降低了并行性，一个事务如果锁定了某行数据，其他事务就必须等待该事务处理完才可以处理那行数


#### 级别区分
##### 共享锁(读锁)
共享锁又称读锁，是读取操作创建的锁。其他用户可以并发读取数据，但任何事务都不能对数据进行修改（获取数据上的排他锁），直到已释放所有共享锁。

如果事务T对数据A加上共享锁后，则其他事务只能对A再加共享锁，不能加排他锁。获准共享锁的事务只能读数据，不能修改数据。

###### 用法

    SELECT ... LOCK IN SHARE MODE;

在查询语句后面增加LOCK IN SHARE MODE，Mysql会对查询结果中的每行都加共享锁，当没有其他线程对查询结果集中的任何一行使用排他锁时，可以成功申请共享锁，否则会被阻塞。其他线程也可以读取使用了共享锁的表，而且这些线程读取的是同一个版本的数据。


##### 排它锁(写锁)
排他锁又称写锁，如果事务T对数据A加上排他锁后，则其他事务不能再对A加任任何类型的封锁。获准排他锁的事务既能读数据，又能修改数据。

###### 用法

    SELECT ... FOR UPDATE;

在查询语句后面增加FOR UPDATE，Mysql会对查询结果中的每行都加排他锁，当没有其他线程对查询结果集中的任何一行使用排他锁时，可以成功申请排他锁，否则会被阻塞。

##### 意向锁(InnodB自动加的锁)
意向锁是表级锁，其设计目的主要是为了在一个事务中揭示下一行将要被请求锁的类型。

###### 意向共享锁（IS）
事务准备给数据行加入共享锁，也就是说一个数据行加共享锁前必须先取得该表的IS锁

###### 意向排他锁（IX）
类似上面，表示事务准备给数据行加入排他锁，说明事务在一个数据行加排他锁前必须先取得该表的IX锁

对于insert、update、delete，InnoDB会自动给涉及的数据加排他锁（X）；对于一般的Select语句，InnoDB不会加任何锁，事务可以通过以下语句给显示加共享锁或排他锁。

#### 粒度区分
##### 表级锁
表级锁是MySQL中锁定粒度最大的一种锁，表示对当前操作的整张表加锁，它实现简单，资源消耗较少，被大部分MySQL引擎支持。最常使用的MYISAM与INNODB都支持表级锁定。

###### 特点

开销小，加锁快；不会出现死锁；锁定粒度大，发出锁冲突的概率最高，并发度最低。

##### 行级锁
行级锁是Mysql中锁定粒度最细的一种锁，表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，但加锁的开销也最大。行级锁分为共享锁 和 排他锁。

###### 特点
开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

##### 页级锁
页级锁是MySQL中锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。所以取了折衷的页级，一次锁定相邻的一组记录。BDB支持页级锁

###### 特点
开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

##### MySQL常用存储引擎的锁机制
> MyISAM和MEMORY采用表级锁(table-level locking)

> BDB采用页面锁(page-level locking)或表级锁，默认为页面锁

> InnoDB支持行级锁(row-level locking)和表级锁,默认为行级锁

###### Innodb
InnoDB行锁是通过给索引上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：**只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁！**

在实际应用中，要特别注意InnoDB行锁的这一特性，不然的话，可能导致大量的锁冲突，从而影响并发性能。

* 在不通过索引条件查询的时候,InnoDB 确实使用的是表锁,而不是行锁。
* 由于 MySQL 的行锁是针对索引加的锁,不是针对记录加的锁,所以虽然是访问不同行 的记录,但是如果是使用相同的索引键,是会出现锁冲突的。应用设计的时候要注意这一点。
* 当表有多个索引的时候,不同的事务可以使用不同的索引锁定不同的行,另外,不论是使用主键索引、唯一索引或普通索引,InnoDB 都会使用行锁来对数据加锁。
* 即便在条件中使用了索引字段,但是否使用索引来检索数据是由 MySQL 通过判断不同 执行计划的代价来决定的,如果MySQL认为全表扫效率更高,比如对一些很小的表,它就不会使用索引,这种情况下InnoDB 将使用表锁,而不是行锁。因此,在分析锁冲突时,别忘了检查SQL的执行计划,以确认是否真正使用了索引。

###### 行级锁与死锁
MyISAM中是不会产生死锁的，因为MyISAM总是一次性获得所需的全部锁，要么全部满足，要么全部等待。而在InnoDB中，锁是逐步获得的，就造成了死锁的可能。

在MySQL中，行级锁并不是直接锁记录，而是锁索引。索引分为主键索引和非主键索引两种，如果一条sql语句操作了主键索引，MySQL就会锁定这条主键索引；如果一条语句操作了非主键索引，MySQL会先锁定该非主键索引，再锁定相关的主键索引。 在UPDATE、DELETE操作时，MySQL不仅锁定WHERE条件扫描过的所有索引记录，而且会锁定相邻的键值，即所谓的next-key locking。

当两个事务同时执行，一个锁住了主键索引，在等待其他相关索引。另一个锁定了非主键索引，在等待主键索引。这样就会发生死锁。

发生死锁后，InnoDB一般都可以检测到，并使一个事务释放锁回退，另一个获取锁完成事务。

有多种方法可以避免死锁，这里只介绍常见的三种

1. 如果不同程序会并发存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁机会。
2. 在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率；
3. 对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过表级锁定来减少死锁产生的概率；


### 概念总结
> 在事务处理的ACID属性中，**一致性是最基本的属性**，其它的三个属性都为了保证一致性而存在的。

所谓一致性，指的是数据处于一种有意义的状态，这种状态是语义上的而不是语法上的。最常见的例子是转帐。例如从帐户A转一笔钱到帐户B上，如果帐户A上的钱减少了，而帐户B上的钱却没有增加，那么我们认为此时数据处于不一致的状态。

在数据库实现的场景中，一致性可以分为数据库外部的一致性和数据库内部的一致性。前者由外部应用的编码来保证，即某个应用在执行转帐的数据库操作时，必须在同一个事务内部调用对帐户A和帐户B的操作。如果在这个层次出现错误，这不是数据库本身能够解决的，也不属于我们需要讨论的范围。后者由数据库来保证，即在同一个事务内部的一组操作必须全部执行成功（或者全部失败）。这就是事务处理的原子性。

> 为了实现原子性，需要通过日志：将所有对数据的更新操作都写入日志，如果一个事务中的一部分操作已经成功，但以后的操作，由于断电/系统崩溃/其它的软硬件错误而无法继续，则通过回溯日志，将已经执行成功的操作撤销，从而达到“全部操作失败”的目的。最常见的场景是，数据库系统崩溃后重启，此时数据库处于不一致的状态，必须先执行一个crashrecovery的过程：读取日志进行REDO（重演将所有已经执行成功但尚未写入到磁盘的操作，保证持久性），再对所有到崩溃时尚未成功提交的事务进行UNDO（撤销所有执行了一部分但尚未提交的操作，保证原子性）。crashrecovery结束后，数据库恢复到一致性状态，可以继续被使用。

原子性并不能完全保证一致性。在多个事务并行进行的情况下，即使保证了每一个事务的原子性，仍然可能导致数据不一致的结果。例如，事务1需要将100元转入帐号A：先读取帐号A的值，然后在这个值上加上100。但是，在这两个操作之间，另一个事务2修改了帐号A的值，为它增加了100元。那么最后的结果应该是A增加了200元。但事实上，事务1最终完成后，帐号A只增加了100元，因为事务2的修改结果被事务1覆盖掉了。

为了保证并发情况下的一致性，引入了隔离性，即保证每一个事务能够看到的数据总是一致的，就好象其它并发事务并不存在一样。用术语来说，就是多个事务并发执行后的状态，和它们串行执行后的状态是等价的。所谓**隔离性的实现方式就是各种类型的锁**。

<br/><br/><br/>
*哈哈哈哈哈哈 是不是看的头大 我也是 :)*

<br/><br/><br/><br/><br/><br/>

资料：
> [事务隔离](https://zh.wikipedia.org/wiki/%E4%BA%8B%E5%8B%99%E9%9A%94%E9%9B%A2)
> 
> [MySQL中的行级锁,表级锁,页级锁](http://www.hollischuang.com/archives/914)
> 
> [数据库事务原子性、一致性是怎样实现的](https://www.zhihu.com/question/30272728)
