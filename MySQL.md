# MySQL

## 基本架构

- 客户端

- Server层：客户端连接器、分析器、优化器、执行器，涵盖 MySQL 的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

- 引擎：存储引擎

### 连接器

负责跟客户端建立连接，验证密码，确认权限（即使你修改用户权限，只能在下一次连接中生效）。

使用 `show processlist` 查看连接的客户端列表。

使用 wait_timeout 参数控制客户端空闲自动断开时长。

建立连接的过程通常是比较复杂的，所以我建议你在使用中要尽量减少建立连接的动作，也就是尽量使用长连接。

全部使用长连接后，有些时候 MySQL 占用内存涨得特别快，这是因为 MySQL 在执行过程中临时使用的内存是管理在连接对象里面的。这些资源会在连接断开的时候才释放。所以如果长连接累积下来，可能导致内存占用太大，被系统强行杀掉（OOM），从现象看就是 MySQL 异常重启了。

定期断开长连接。使用一段时间，或者程序里面判断执行过一个占用内存的大查询后，断开连接，之后要查询再重连。

如果你用的是 MySQL 5.7 或更新版本，可以在每次执行一个比较大的操作后，通过执行 mysql_reset_connection 来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。

### 查询缓存

查询缓存的弊端是：如果写频繁，缓存就会频繁失效。

MySQL 8.0 版本直接将查询缓存的整块功能删掉了，之前也可以手动制定：

```sql
select SQL_CACHE * from T where ID=10；
```

### 分析器

词法分析，关键词识别，表名识别，列表识别。

语法分析，是否满足 MySQL 语法。

### 优化器

最佳的执行计划。

### 执行器

首先判断执行权限。

然后掉用引擎提供的存取接口，每次掉用一行，然后下一行，以此类推。

## 日志系统

更新语句牵涉到 redo log（重做日志）和 binlog（归档日志）。

redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。

redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。

redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

redo log 和 binlog 使用的 两阶段提交，保证事务性。

### redo log

redo log 是 InnoDB 引擎特有的日志，有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为crash-safe。

因为在大体量文件里找数据 磁盘IO的成本很高，类似于一个写缓存，而且追加到文件末尾，效率更高。然后在空闲的时候，合并到数据库里。

redo log 是环形结构，类似于记账的黑板擦，写满了就需要合并到主账本里。

WAL 的全称是 Write-Ahead Logging，它的关键点就是先写日志，再写磁盘，也就是先写粉板，等不忙的时候再写账本。

你可以设置4个文件，每个文件1GB。

### 备份频率

频率越高，好处是“最长恢复时间”更短。

在一天一备的模式里，最坏情况下需要应用一天的 binlog。比如，你每天 0 点做一次全量备份，而要恢复出一个到昨天晚上 23 点的备份。

一周一备最坏情况就要应用一周的 binlog 了。

## 事务隔离

ACID 和 隔离级别 参见 《SQL必知必会》和 SQL笔记

开启一个事务并且获得一致性视图：

```sql
START TRANSACTION  WITH CONSISTENT SNAPSHOT；
```

查看隔离级别：

```sql
show variables like 'transaction-isolation'
```

===============

** 更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读”（current read），更新操作一旦当前读之后成功之后，后面的读都会以更新的版本为准，如果不成功呢？那后面也不拿到当前读的数据。** 

案例：为什么修改表数据，结果再次查看没变化？1、在修改语句之前，条件已经发生变化，导致修改语句失效了。2、之后的查询语句，因为Update没有成功，版本ID不是当前的版本，所以拿的还是之前版本的值。

** MVCC 为每个事务给一个版本号，每次查询，都去比对一下什么该读，什么不该读，当Update的时候，才会把新数据更新当前的版本号，这样后面的查询才会以更新的为准。**

除了 update 语句外，select 语句如果加锁，也是当前读。

```sql
select k from t where id=1 lock in share mode;
select k from t where id=1 for update;
```

===============

表结构不支持“可重复读”？这是因为表结构没有对应的行数据，也没有 row trx_id，因此只能遵循当前读的逻辑。

而 MySQL 8.0 已经可以把表结构放在 InnoDB 字典里了，也许以后会支持表结构的可重复读。

===============

不要使用长事务

长事务意味着系统里面会存在很老的事务视图（MVCC）。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间。

在 MySQL 5.5 及以前的版本，回滚日志是跟数据字典一起放在 ibdata 文件里的，即使长事务最终提交，回滚段被清理，文件也不会变小。我见过数据只有 20GB，而回滚段有 200GB 的库。最终只好为了清理回滚段，重建整个库。

除了对回滚段的影响，长事务还占用锁资源，也可能拖垮整个库。

===============

避免长事务影响

1. 确认是否使用了 set autocommit=0。这个确认工作可以在测试环境中开展，把 MySQL 的 general_log 开起来，然后随便跑一个业务逻辑，通过 general_log 的日志来确认。一般框架如果会设置这个值，也就会提供参数来控制行为，你的目标就是把它改成 1。

2. 确认是否有不必要的只读事务。有些框架会习惯不管什么语句先用 begin/commit 框起来。我见过有些是业务并没有这个需要，但是也把好几个 select 语句放到了事务中。这种只读事务可以去掉。

3. 业务连接数据库的时候，根据业务本身的预估，通过 SET MAX_EXECUTION_TIME 命令，来控制每个语句执行的最长时间，避免单个语句意外执行太长时间。？？？

4. 监控 information_schema.Innodb_trx 表，设置长事务阈值，超过就报警 / 或者 kill；

5. Percona 的 pt-kill 这个工具不错，推荐使用；

6. 在业务功能测试阶段要求输出所有的 general_log，分析日志行为提前发现问题；

7. 如果使用的是 MySQL 5.6 或者更新版本，把 innodb_undo_tablespaces 设置成 2（或更大的值）。如果真的出现大事务导致回滚段过大，这样设置后清理起来更方便。

===============

## 索引

索引可能因为删除，或者页分裂等原因，导致数据页有空洞，重建索引的过程会创建一个新的索引，把数据按顺序插入，这样页面的利用率最高，也就是索引更紧凑、更省空间。

重建索引 和 重建主键索引：

```sql
alter table T engine=InnoDB
```

### 覆盖索引

非聚簇索引 存储的值是 索引+行ID，查询的时候需要回表，而聚簇索引不需要这一步。

如果查询需要的列已经在索引中，则不需要回表。则索引“覆盖了”我们的查询需求，我们称为覆盖索引。

### 最左前缀原则

索引具有复用能力。因为可以支持最左前缀，所以当已经有了 (a,b) 这个联合索引后，一般就不需要单独在 a 上建立索引了。

如果通过调整顺序，可以少维护一个索引，那么这个顺序往往就是需要优先考虑采用的。

### 索引下推

MySQL 5.6 引入的索引下推优化（index condition pushdown)， 可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。

比如 张* 年龄，可以继续判断年龄，然后再继续判断名字。

## 锁

### 全局锁

Flush tables with read lock (FTWRL)

全局锁的典型使用场景是，做全库逻辑备份。也就是把整库每个表都 select 出来存成文本。

对于支持可重复读的引擎，mysqldump 使用参数–single-transaction，对于 MyISAM 这种不支持事务的引擎，如果备份过程中有更新，那么就破坏了备份的一致性。这时，我们就需要使用 FTWRL 命令。

single-transaction 方法只适用于所有的表使用事务引擎的库。如果有的表使用了不支持事务的引擎，那么备份就只能通过 FTWRL 方法。这往往是 DBA 要求业务开发人员使用 InnoDB 替代 MyISAM 的原因之一。

既然要全库只读，为什么不使用 set global readonly=true 的方式呢？

1. 在有些系统中，readonly 的值会被用来做其他逻辑，比如用来判断一个库是主库还是备库。因此，修改 global 变量的方式影响面更大，我不建议你使用。

2. 在异常处理机制上有差异。如果执行 FTWRL 命令之后由于客户端发生异常断开，那么 MySQL 会自动释放这个全局锁，整个库回到可以正常更新的状态。而将整个库设置为 readonly 之后，如果客户端发生异常，则数据库就会一直保持 readonly 状态，这样会导致整个库长时间处于不可写状态，风险较高。

业务的更新不只是增删改数据（DML)，还有可能是加字段等修改表结构的操作（DDL）。不论是哪种方法，一个库被全局锁上以后，你要对里面任何一个表做加字段操作，都是会被锁住的。


### 表级锁

表锁 + 元数据锁（meta data lock，MDL）

表锁的语法是 lock tables … read/write。与 FTWRL 类似，可以用 unlock tables 主动释放锁，也可以在客户端断开的时候自动释放。需要注意，lock tables 语法除了会限制别的线程的读写外，也限定了本线程接下来的操作对象。

对于 InnoDB 这种支持行锁的引擎，一般不使用 lock tables 命令来控制并发，毕竟锁住整个表的影响面还是太大。

元数据锁 MDL（metadata lock)。MDL 不需要显式使用，在访问一个表的时候会被自动加上。MDL 的作用是，保证读写的正确性。你可以想象一下，如果一个查询正在遍历一个表中的数据，而执行期间另一个线程对这个表结构做变更，删了一列，那么查询线程拿到的结果跟表结构对不上，肯定是不行的。

在 MySQL 5.5 版本中引入了 MDL，当对一个表做增删改查操作的时候，加 MDL 读锁；当要对表做结构变更操作的时候，加 MDL 写锁。

===========

给小表加字段改挂了怎么办？

事务中的 MDL 锁，在语句执行开始时申请，但是语句结束后并不会马上释放，而会等到整个事务提交后再释放。

如何安全地给小表加字段？

首先我们要解决长事务，事务不提交，就会一直占着 MDL 锁。

在 MySQL 的 information_schema 库的 innodb_trx 表中，你可以查到当前执行中的事务。如果你要做 DDL 变更的表刚好有长事务在执行，要考虑先暂停 DDL，或者 kill 掉这个长事务。

如果你要变更的表是一个热点表，虽然数据量不大，但是上面的请求很频繁，而你不得不加个字段，这时候 kill 可能未必管用，因为新的请求马上就来了。比较理想的机制是，在 alter table 语句里面设定等待时间，如果在这个指定的等待时间里面能够拿到 MDL 写锁最好，拿不到也不要阻塞后面的业务语句，先放弃。之后开发人员或者 DBA 再通过重试命令重复这个过程。

```sql
ALTER TABLE tbl_name NOWAIT add column ...
ALTER TABLE tbl_name WAIT N add column ... 
```

===========

从库备库的时候主库来了DDL怎么办？

工具备份流程 有个 show create table `t1`; -> SELECT * FROM `t1`; 必须先拿到表结构，再拿到表数据的过程。

如何在这个关键流程之前，那备份拿到的是新表。

如果在中间，就会报错，表结构已经发生变化。

如果之后，那就拿的是旧表。

===========

### 行锁

两阶段锁协议：在 InnoDB 事务中，行锁是在需要的时候才加上的，但并不是不需要了就立刻释放，而是要等到事务结束时才释放。

结论：如果你的事务中需要锁多个行，要把最可能造成锁冲突、最可能影响并发度的锁尽量往后放。

===========

死锁

现象：登上服务器一看，CPU 消耗接近 100%，但整个数据库每秒就执行不到 100 个事务。

死锁解决：

1. innodb_lock_wait_timeout 超时时间，默认 50s。

但这个时间不好确定，因为哪怕时间长一点，可能就是长事务。

2. innodb_deadlock_detect 死锁检测，默认打开。

但是死锁检测需要消耗大量CPU，1000个并发，检测就是100万这个量级，一个方法就是控制并发量。

客户端控制并发，用消息队列来。引擎层控制并发，修改源码。

还有个方法就是：把1000块钱，分成100个金额，去参与并发。

===========

删除表的10000行数据，怎么做最好？

在一个连接中循环执行 20 次 delete from T limit 500，因为这样避免锁占用，也避免长事务。


## 实战经验

### 普通索引、唯一索引

如果 LIMIT 1，普通索引和唯一索引差距微乎其微，因为局部性原理 和 数据按页读。

有一个场景：读少写多，写入的数据不会立刻去读，而且内存中没有数据页，这个时候 普通索引 更好。

因为 唯一索引 每次写入，比如要用 数据页 保证唯一性，如果内存里没有，就需要从磁盘读出来，而普通索引遇到这种情况，就不需要读，直接写入 写缓存（change buffer） 即可，等 下次读/定期，从磁盘取出来 merge 和 save。

如果这样的场景，写入马上去读，那最好关闭 写缓存，因为马上读的时候，会强制触发 merge 操作，从磁盘读出数据页，然后执行 merge，最后返回和写入，这样反而适得其反。

写缓存 通过参数 innodb_change_buffer_max_size 来动态设置。这个参数设置为 50 的时候，表示 change buffer 的大小最多只能占用 buffer pool 的 50%。

写缓存不要怕丢失，不会，因为还有 redo log。

有的时候，业务需要唯一性约束，那就该用就用，对于归档库，可以用普通索引，这个时候数据确保没有唯一性冲突。

===========

- merge 的过程不会把数据页写入磁盘，而是标记脏，写入redo log就结束了，后期有人会刷回磁盘。

- ** redo log 主要节省的是随机写磁盘的 IO 消耗（转成顺序写），而 change buffer 主要节省的则是随机读磁盘的 IO 消耗。**

===========

### 选错索引？

选错的本质在于优化器对表行数判断失误，因为 MVCC 机制，不可能维护一个行数属性，所以采样统计：随机选择 N 个数据页，统计不同的值，得到平均值，然后乘以 这个索引有多少数据页。

有的时候优化器还是会选择行数多的，因为还要考虑回表的成本。

重新统计索引信息命令：`analyze table t`。

当变更数据行数超过 1/M 时，重新做索引统计，调整 innodb_stats_persistent 参数：
- 设置为 on 的时候，表示统计信息会持久化存储。这时，默认的 N 是 20，M 是 10。
- 设置为 off 的时候，表示统计信息只存储在内存中。这时，默认的 N 是 8，M 是 16。

方案：
1. 使用 force index 指定索引。
2. 特殊处理，让优化器觉得这一步有很大的代价。比如优化器选择索引b是因为后面避免排序，那我可以故意按 b,a 排序，优化器就会放弃使用索引b。
3. 新建合适的索引，删除误导的索引。

===========

有一种情况，一个事务占着，另一个事务清空了数据，并且又新增了一批，这个时候索引行数判断时 旧 + 新 的数据。

首先，因为事务还在，旧数据不能删掉，索引上的数据是两份，其次，如果索引是主键索引，用的是 show table status 的值来统计的，不会出现统计两份数据的情况。

===========

### 字符串加索引

权衡下，使用前缀索引，定义好长度，既省空间，又不会增加太多查询成本。

使用 前缀索引 就用不上覆盖索引对查询性能的优化了，选择是否使用前缀索引时需要考虑的一个因素。

```sql
select 
  count(distinct left(email,4)）as L4,
  count(distinct left(email,5)）as L5,
  count(distinct left(email,6)）as L6,
  count(distinct left(email,7)）as L7,
from SUser; -- 找出区分度，把区分度维持在 95% 这个级别，就可以了。 Ln / 总行 > 95%。
```

对于身份证，特点就是前面几位区分度很小，可以采用：

1. 倒序存储。
2. 新字段，散列化，好处就是扫描行数为1，坏处就是需要维护新字段，占用空间；而且CPU占用比倒序多。

⚠️ 要考虑表的潜在规模，有的时候，十几万的小表，没必要搞这个。


### SQL语句变慢了

可能在刷脏页，导致原因：
1. redo log 写满了。这个时候更新被堵住，跌为0，这个不能接受。
2. 写入量比较大，内存不足。这是正常情况，mysql本来就充分使用内存，如果淘汰脏页太多，也不能接受。
3. 系统空闲的时候
4. 系统关闭流程

===========

InnoDB控制策略：

首先要告诉mysql机器的磁盘写入能力，innodb_io_capacity 参数，设置成 IOPS，用这个命令测：

`fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest `

InnoDB刷盘的速度参考两个因素：脏页比例 和redo log写盘速度。

innodb_max_dirty_pages_pct 脏页比例上限，默认 75%

意思就是，如何脏页比例很高，或者redo log快写满了，就会触发，然后乘以 innodb_io_capacity 得出最终刷盘速度。

平常多关注脏页比例：

```sql
select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
select @a/@b;
```

InnoDB刷盘有一个 “顺带机制”，就是刷页的时候，如果这个脏的数据页旁边的数据页也是脏的，那就一起带走。这个特性是 innodb_flush_neighbors 参数控制，为 1 表示开启，为 0 表示不开启顺带，8.0 中默认为0，如果是SSD，设置为 1 就可以了，因为瓶颈不在IO上，如果是机械硬盘，还是很有意义的。

如果 机器 规格很大，redo log 设置的文件很小，这个时候就会频繁刷脏页。


### 表删掉一变，大小不变

原因：数据页复用。

插入和删除 也会导致数据页空洞，所以始终建议添加而不是插入，但是一般数据页会预留一些空间，差不多 1/16空间。

【 插入改成添加，删除会导致空洞，有个做法就是，软删除，然后定期（找个不忙的时候）去清理删除标记的行，然后重建表。 】

```sql
alter table A engine=InnoDB -- 重建表
```

以前重建表，DDL期间，表不能有更新，所以有了 Online DDL，核心就是使用一个 日志 去记录新更新的值，然后重建结束后合并进去。

alter 语句启动需要获取MDL写锁，防止其他线程同时做DDL，然后拷贝就退化读锁，这样不会阻塞客户端更新操作。

如果是大表，这个很耗费IO和CPU资源，如果线上服务，控制好时间，然后使用 GitHub 开源的 gh-ost 来做比较安全。

===========

始终设置 innodb_file_per_table 为 ON，因为独立文件，当删除表的时候，直接删除文件，如果是OFF，用共享表空间，哪怕数据删除，文件大小都不会变，因为考虑复用。5.6.6 开始默认为 ON。

对于 Server层 来说，InnoDB 的 Online 就是 inplace，因为临时表不是 Server层 创建的，但 1TB 的数据，1.2TB 的磁盘不能做重建，因为内部需要创建临时表。

```sql
optimize table t; -- = `alter table t engine = InnoDB` + `analyze table t`
```

重建表 可能导致数据库文件变大了，因为数据页会预留 1/16大小的空间。

===========

### count(*)

count(字段)<count(主键 id)<count(1)≈count(*)

怎么最快？：索引的大小越小，窄索引，载入内存就越快，然后扫描索引就可以了。

之所以 InnoDB 不用一个属性维护行数，是因为MVCC要根据每个事务的情况，去获取它可重复读下应该有多少行数据。

show table status 里面的行数是基于索引采样统计出来的，也不准，误差 40% 左右。

用 Redis 保存行数 和 数据库保存 区别：数据库保存有 MVCC，它只能看到一致性读的行数。

### order by 原理

Using filesort：需要排序，MySQL会分配一块内存，sort_buffer。

参数 sort_buffer_size：如果需要的小于，那就在内存排序，如果大于，需要创建临时文件，用磁盘辅助排序。

===========

确定一个排序语句是否使用了临时文件:

```sql
/* 打开 optimizer_trace，只对本线程有效 */
SET optimizer_trace='enabled=on'; 
 
/* @a 保存 Innodb_rows_read 的初始值 */
select VARIABLE_VALUE into @a from  performance_schema.session_status where variable_name = 'Innodb_rows_read';
 
/* 执行语句 */
select city, name,age from t where city='杭州' order by name limit 1000; 
 
/* 查看 OPTIMIZER_TRACE 输出 */
SELECT * FROM `information_schema`.`OPTIMIZER_TRACE`\G
 
/* @b 保存 Innodb_rows_read 的当前值 */
select VARIABLE_VALUE into @b from performance_schema.session_status where variable_name = 'Innodb_rows_read';
 
/* 计算 Innodb_rows_read 差值 */
select @b-@a;
```

从 number_of_tmp_files 中看到是否使用了临时文件。

外部排序一般使用归并排序算法。

MySQL 将需要排序的数据分成 12 份，每一份单独排序后存在这些临时文件中。然后把这 12 个有序文件再合并成一个有序的大文件。

===========

如果行数据太大，sort_buffer太小，就会从 全字段排序 换成 rowid 排序，每次仅排序一个列，然后回表拿下一个。

MySQL 的一个设计思想：如果内存够，就要多利用内存，尽量减少磁盘访问。

max_length_for_sort_data 参数：MySQL 中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，MySQL 就认为单行太大，要换一个算法。

```sql
SET max_length_for_sort_data = 16;
````

排序的成本大，而如果就建立联合索引，这样根本就不需要排序。如果覆盖索引，连回表都不需要。

具体要权衡，测一下，比较。

===========

select * from t where city in (“杭州”," 苏州 ") order by name limit 100; 这个 SQL 语句是否需要排序？有什么方案可以避免排序？有 (city,name) 联合索引。

只能保证单个 city 内部，name 是递增的，可以 city 分开读，然后归并排序。

===========

### 随机消息

`select word from words order by rand() limit 3;`

order by rand() 使用了内存临时表，内存临时表排序的时候使用了 rowid 排序方法。

order by rand() 使用了磁盘临时表，但排序算法使用的是 优先队列排序算法，因为只要3个，没必要全部排序。

不管怎么养，order by rand() 都耗费巨大，详细参见：《06MySQL实战45讲：17 | 如何正确地显示随机消息？》

算法：

```sql
select count(*) into @C from t;
set @Y1 = floor(@C * rand());
set @Y2 = floor(@C * rand());
set @Y3 = floor(@C * rand());
select * from t limit @Y1，1； // 在应用代码里面取 Y1、Y2、Y3 值，拼出 SQL 后执行
select * from t limit @Y2，1；
select * from t limit @Y3，1；
````

### SQL语句逻辑相同，性能却差异巨大？

对索引字段做函数操作，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能。

`select * from tradelog where id + 1 = 10000` 也不行，必须 `where id = 10000 -1`。

数据类型转换也会导致索引失效，需要函数转换。

字符集不通也会导致索引失效，需要函数转换。


### 查一行就很慢

等待 MDL 锁

对于阻塞，performance_schema=ON，会有 10% 的性能损失，然后 `select blocking_pid from sys.schema_table_lock_waits ` 就能找到阻塞的 pid，然后 kill 掉。

等 flush，flush tables 命令 正常很快，除非等待其他语句，用 show processlist 查看。

等 行锁。找到阻塞的（`select * from t sys.innodb_lock_waits where locked_table=`'test'.'t'`\G
`），KILL QUERY 4 （停止当前正在执行的语句，如果行锁的语言已经完成了，那这个语句不能释放锁） 或 KILL 4 （这个直接断开连接，事务也会回滚）。

查询慢：没有索引。

一致性读的时候，当前事务的版本距离最新有几百万个。


### 幻读的问题

一个可重复读的事务，使用 WHERE 语句对几个行加锁，现在其他事务插入新行，这个行满足之前的 WHERE 条件，但是这些数据插入后没有锁，导致这个事务的锁有Bug，这就是幻读。

解决：MVCC + 行锁 + 间隙锁，问题就是锁住更大范围，更容易死锁。

如果改成 读提交（间隙锁是在可重复读隔离级别下才会生效） + binlog_format=row ，但是 mysqldump 备份的时候用的是可重复读。

加锁的规则：
原则 1：加锁的基本单位是 next-key lock。next-key lock 是前开后闭区间。
原则 2：查找过程中访问到的对象才会加锁。
优化 1：索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。
优化 2：索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁。
一个 bug：唯一索引上的范围查询会访问到不满足条件的第一个值为止。

详细理解参见案例：《06MySQL实战45讲：21 | 为什么我只改一行的语句，锁这么多？》


### 临时救场

1. 连接数据库跳过权限验证：使用–skip-grant-tables 参数启动

2. 如果 连接数 用完，优先 Kill 掉占着不用的连接。

详细理解参见案例：《06MySQL实战45讲：22 | MySQL有哪些“饮鸩止渴”提高性能的方法？》

### 保证数据持久性

#### binlog

binlog 的写入逻辑：事务执行过程中，先把日志写到 binlog cache，事务提交的时候，再把 binlog cache 写到 binlog 文件中。

一个事务的 binlog 是不能被拆开的，因此不论这个事务多大，也要确保一次性写入。这就涉及到了 binlog cache 的保存问题。

系统给 binlog cache 分配了一片内存，每个线程一个，参数 binlog_cache_size 用于控制单个线程内 binlog cache 所占内存的大小。如果超过了这个参数规定的大小，就要暂存到磁盘。

事务提交的时候，执行器把 binlog cache 里的完整事务写入到 binlog 中，并清空 binlog cache。

每个线程有自己 binlog cache，但是共用同一份 binlog 文件。

===========

参数 sync_binlog：
- sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync；
- sync_binlog=1 的时候，表示每次提交事务都会执行 fsync；
- sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。

write：指把日志写入到文件系统的 page cache，并没有把数据持久化到磁盘，所以速度比较快。
fsync：将数据持久化到磁盘的操作。一般情况下，我们认为 fsync 才占磁盘的 IOPS。

出现 IO 瓶颈的场景里，将 sync_binlog 设置成一个比较大的值，可以提升性能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成 0，比较常见的是将其设置为 100~1000 中的某个数值。

但是，将 sync_binlog 设置为 N，对应的风险是：如果主机发生异常重启，会丢失最近 N 个事务的 binlog 日志。


#### redo log

生成的 redo log 是要先写到 redo log buffer 的，也不是每次生成之后都要持久化到磁盘上。

如果事务执行期间 MySQL 发生异常重启，那这部分日志就丢了。由于事务并没有提交，所以这时日志丢了也不会有损失。

但是！事务还没提交的时候，redo log buffer 中的部分日志有可能被持久化到磁盘。

日志写到 redo log buffer 是很快的，wirte 到 page cache 也差不多，但是持久化到磁盘的速度就慢多了。控制 redo log 的写入策略，InnoDB 提供了 innodb_flush_log_at_trx_commit 参数：
- 设置为 0 的时候，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中 ;
- 设置为 1 的时候，表示每次事务提交时都将 redo log 直接持久化到磁盘；
- 设置为 2 的时候，表示每次事务提交时都只是把 redo log 写到 page cache。

InnoDB 有一个后台线程，每隔 1 秒，就会把 redo log buffer 中的日志，调用 write 写到文件系统的 page cache，然后调用 fsync 持久化到磁盘。

注意，事务执行中间过程的 redo log 也是直接写在 redo log buffer 中的，这些 redo log 也会被后台线程一起持久化到磁盘。也就是说，一个没有提交的事务的 redo log，也是可能已经持久化到磁盘的。

还有两种场景也会导致一个没有提交的事务写入磁盘：
1. redo log buffer 占用的空间即将达到 innodb_log_buffer_size 一半的时候，后台线程会主动写盘（提交到 page cache）。
2. 并行的事务提交的时候，顺带将这个事务的 redo log buffer 持久化到磁盘。

MySQL 的“双 1”配置，指的就是 sync_binlog 和 innodb_flush_log_at_trx_commit 都设置成 1：一个事务完整提交前，需要等待两次刷盘，一次是 redo log（prepare 阶段），一次是 binlog。这意味着我从 MySQL 看到的 TPS 是每秒两万的话，每秒就会写四万次磁盘。但是，我用工具测试出来，磁盘能力也就两万左右，怎么能实现两万的 TPS？因为 组提交（group commit）机制，相当于批量写入。

详细参见：《06MySQL实战45讲：23 | MySQL是怎么保证数据不丢的？》

日志机制得益于：
1. redo log 和 binlog 都是顺序写，磁盘的顺序写比随机写速度要快；
2. 组提交机制，可以大幅度降低磁盘的 IOPS 消耗。

如果你的 MySQL 现在出现了性能瓶颈，而且瓶颈在 IO 上，可以通过哪些方法来提升性能呢？
1. 设置 binlog_group_commit_sync_delay 和 binlog_group_commit_sync_no_delay_count 参数，减少 binlog 的写盘次数。这个方法是基于“额外的故意等待”来实现的，因此可能会增加语句的响应时间，但没有丢失数据的风险。
2. 将 sync_binlog 设置为大于 1 的值（比较常见是 100~1000）。这样做的风险是，主机掉电时会丢 binlog 日志。
3. 将 innodb_flush_log_at_trx_commit 设置为 2。这样做的风险是，主机掉电的时候会丢数据。

我不建议你把 innodb_flush_log_at_trx_commit 设置成 0。因为把这个参数设置成 0，表示 redo log 只保存在内存中，这样的话 MySQL 本身异常重启也会丢数据，风险太大。而 redo log 写到文件系统的 page cache 的速度也是很快的，所以将这个参数设置成 2 跟设置成 0 其实性能差不多，但这样做 MySQL 异常重启时就不会丢数据了，相比之下风险会更小。

===========

数据库的 crash-safe 保证的是：

1. 如果客户端收到事务成功的消息，事务就一定持久化了；

2. 如果客户端收到事务失败（比如主键冲突、回滚等）的消息，事务就一定失败了；

3. 如果客户端收到“执行异常”的消息，应用需要重连后通过查询当前状态来继续后续的逻辑。此时数据库只需要保证内部（数据和日志之间，主库和备库之间）一致就可以了。

### 保证主备一致

binlog 设置的是 statement 格式，如果语句中有 limit，那这个命令可能是 unsafe 的。

当 binlog_format 使用 row 格式的时候，binlog 里面记录了真实删除行的主键 id，这样 binlog 传到备库去的时候，就肯定会删除 id=4 的行，不会有主备删除不同行的问题。

mixed 格式可以利用 statment 格式的优点，同时又避免了数据不一致的风险：
- 因为有些 statement 格式的 binlog 可能会导致主备不一致，所以要使用 row 格式。
- 但 row 格式的缺点是，很占空间。比如你用一个 delete 语句删掉 10 万行数据，用 statement 的话就是一个 SQL 语句被记录到 binlog 中，占用几十个字节的空间。但如果用 row 格式的 binlog，就要把这 10 万条记录都写到 binlog 中。这样做，不仅会占用更大的空间，同时写 binlog 也要耗费 IO 资源，影响执行速度。
- 所以，MySQL 就取了个折中方案，也就是有了 mixed 格式的 binlog。mixed 格式的意思是，MySQL 自己会判断这条 SQL 语句是否可能引起主备不一致，如果有可能，就用 row 格式，否则就用 statement 格式。

在时间戳的语句，通过 SET TIMESTAMP 命令，MySQL 就确保了主备数据的一致性。

重放 binlog 数据的时候，是有风险的。因为有些语句的执行结果是依赖于上下文命令的，直接执行的结果很可能是错误的。最佳做法是，用 binlog 来恢复数据的标准做法是，用 mysqlbinlog 工具解析出来，然后把解析结果整个发给 MySQL 执行。
```shell
mysqlbinlog master.000001  --start-position=2738 --stop-position=2973 | mysql -h127.0.0.1 -P13000 -u$user -p$pwd;
```

解决循环复制的问题：

1. 规定两个库的 server id 必须不同，如果相同，则它们之间不能设定为主备关系；

2. 一个备库接到 binlog 并在重放的过程中，生成与原 binlog 的 server id 相同的新的 binlog；

3. 每个库在收到从自己的主库发过来的日志后，先判断 server id，如果跟自己的相同，表示这个日志是自己生成的，就直接丢弃这个日志。


### 保证高可用

主备同步的延迟时间 = 传输的时间 + 重放的时间。

> show slave status 命令 有个 seconds_behind_master，表示当前备库延迟了多少秒。

主备延迟的原因：

1. 备库所在机器的性能要比主库所在的机器性能差。

2. 备库的压力大，忽略了运营维护统计的业务。

一主多从。除了备库外，可以多接几个从库，让这些从库来分担读的压力。或者 通过 binlog 输出到外部系统，比如 Hadoop 这类系统，让外部系统提供统计类查询的能力。

3. 大事务

如果一个主库上的语句执行 10 分钟，那这个事务很可能就会导致从库延迟 10 分钟。比如 一次性地用 delete 语句删除太多数据。

4. 大表DDL

处理方案就是，计划内的 DDL，建议使用 gh-ost 方案。

5. 备库的并行复制能力

MariaDB 的并行复制策略 是模拟主库并发，因为主库是按组提交commit，传到备库里也是按组来分发worker，问题就在，每组事务之间延迟很低，而备库这一块延迟就高了，吞吐量不行。

MySQL之后的版本思路都是：同处于prepare状态的事务，备库是可以并行的，处于prepare的事务跟处于commit状态的事务，在备库也是可以并行的。

详细参见：《06MySQL实战45讲：26 | 备库为什么会延迟好几个小时？》


因为主备之间有延迟，所以切换的时候，有一段时间不可用，这个时候用两种策略：

可靠性优先策略

1. 保证网络延迟5秒以内。
2. 主库设为只读，业务暂停。
3. 判断备库的同步延迟一直到0。
4. 把备库改成读写状态，切换到备库

可用性优先策略

直接调整到备库执行，不等同步完成，在一些允许数据丢失的情况可以的。


===============

备库的主备延迟会表现为一个 45 度的线段：

1. 大事务（大表DDL，一个事务操作很多行）。
2. 备库上有个长事务。

===============


### 主库出问题，从库怎么办

一主多从的主备切换：有一库专门负责与主库进行切换，这个时候主库遇到问题就需要主备切换。

===============

基于位点的主备切换 

执行一条 change master 命令：
```sql
CHANGE MASTER TO 
MASTER_HOST=$host_name 
MASTER_PORT=$port 
MASTER_USER=$user_name 
MASTER_PASSWORD=$password 
MASTER_LOG_FILE=$master_log_name 
MASTER_LOG_POS=$master_log_pos  
```

问题就在 MASTER_LOG_POS 同步位点上，但这个位点很难精确取到，只能取一个大概位置。

一般取位点的方法：
1. 等待新主库 A’把中转日志（relay log）全部同步完成；
2. 在 A’上执行 show master status 命令，得到当前 A’上最新的 File 和 Position；
3. 取原主库 A 故障的时刻 T；
4. 用 mysqlbinlog 工具解析 A’的 File，得到 T 时刻的位点。
`mysqlbinlog File --stop-datetime=T --start-datetime=T`

但这个值不准确，会导致从库出现主键冲突，因为上面算出来的值是最后一个记录的开始位置。有两种方法：

1. 主动跳过一个事务。

```sql
set global sql_slave_skip_counter=1;
start slave;
```

2. 通过设置 slave_skip_errors 参数，直接设置跳过指定的错误。

在执行主备切换时，有这么两类错误，是经常会遇到的：
1062 错误是插入数据时唯一键冲突；
1032 错误是删除数据时找不到行。

我们可以把 slave_skip_errors 设置为 “1032,1062”，这样中间碰到这两个错误时就直接跳过。

这个方法的前提是这样跳过错误数据依然是无损的。

===============

GTID

GTID 的全称是 Global Transaction Identifier，也就是全局事务 ID，是一个事务在提交的时候生成的，是这个事务的唯一标识。按事务为单位，自动找位点。

```sql
CHANGE MASTER TO 
MASTER_HOST=$host_name 
MASTER_PORT=$port 
MASTER_USER=$user_name 
MASTER_PASSWORD=$password 
master_auto_position=1 
```

master_auto_position=1 就表示这个主备关系使用的是 GTID 协议。

详细参见：《06MySQL实战45讲：27 | 主库出问题了，从库怎么办？》
