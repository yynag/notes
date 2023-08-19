# SQL

## 概念

OLTP 联机事务处理， 一般用于处理客户的事务和进行查询，需要随时对数据表中的记录进行增删改查，对实时性要求高。

OLAP  联机分析处理，一般用于市场的数据分析，通常数据量大，需要进行复杂的分析操作，可以对大量历史数据进行汇总和分析，对实时性要求不高。

那么对于 OLTP 来说，由于随时需要对数据记录进行增删改查，更适合采用行式存储，因为一行数据的写入会同时修改多个列。传统的 RDBMS 都属于行式存储，比如 Oracle、SQL Server 和 MySQL 等。

对于 OLAP 来说，由于需要对大量历史数据进行汇总和分析，则适合采用列式存储，这样的话汇总数据会非常快，但是对于插入（INSERT）和更新（UPDATE）会比较麻烦，相比于行式存储性能会差不少。


## SQL执行流程

1. 语法检查，关键词拼写正确。
2. 语义检查，列名、表名 拼写正确。
3. 权限检查，是否具备访问表的权限。
4. 优化器，如果共享池里有，用Hash拿到执行计划，如果没有，创建树解析，进入优化阶段，生成执行计划。
5. 执行。

绑定变量 可以复用执行计划，软解析：

```sql
select * from player where player_id = :player_id;
```

与 Oracle 不同， MySQL 存储引擎使用插件的形式：
1. InnoDB 默认，支持事务。
2. MyISAM 不支持事务，速度快，占用资源少，适合静态数据。
3. Memory 内存数据库。
4. NDB 集群。
5. Archive 归档，很好的压缩，适合仓库。

**数据库的设计在于表的设计，MySQL每个表都可以指定特定的存储引擎，这是它的强大之处。**


## 数据库操作

DDL，英文叫做 Data Definition Language，也就是数据定义语言，它用来定义我们的数据库对象，包括数据库、数据表和列。通过使用 DDL，我们可以创建，删除和修改数据库和表结构。

DML，英文叫做 Data Manipulation Language，数据操作语言，我们用它操作和数据库相关的记录，比如增加、删除、修改数据表中的记录。

DCL，英文叫做 Data Control Language，数据控制语言，我们用它来定义访问权限和安全级别。

DQL，英文叫做 Data Query Language，数据查询语言，我们用它查询想要的记录，它是 SQL 语言的重中之重。


## 表

```sql
CREATE DATABASE nba;  // 创建一个名为 nba 的数据库
DROP DATABASE nba;  // 删除一个名为 nba 的数据库

// 创建表
DROP TABLE IF EXISTS `player`;
CREATE TABLE `player`  (
  `player_id` int(11) NOT NULL AUTO_INCREMENT,
  `team_id` int(11) NOT NULL,
  `player_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `height` float(3, 2) NULL DEFAULT 0.00,
  PRIMARY KEY (`player_id`) USING BTREE,
  UNIQUE INDEX `player_name`(`player_name`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

— 排序规则是utf8_general_ci，代表对大小写不敏感，如果设置为utf8_bin，代表对大小写敏感

// 修改表结构

ALTER TABLE player ADD (age int(11)); — 添加字段

ALTER TABLE player RENAME COLUMN age to player_age; — 修改字段名

ALTER TABLE player MODIFY (player_age float(3,1)); — 修改字段的数据类型

ALTER TABLE player DROP COLUMN player_age; — 删除字段
```


### 表约束

主键起的作用是唯一标识一条记录，不能重复，不能为空，即 UNIQUE+NOT NULL。一个数据表的主键只能有一个。主键可以是一个字段，也可以由多个字段复合组成。

外键确保了表与表之间引用的完整性。一个表中的外键对应另一张表的主键。外键可以是重复的，也可以为空。

唯一性约束表明了字段在表中的数值是唯一的，即使我们已经有了主键，还可以对其他字段进行唯一性约束。

唯一性约束和普通索引（NORMAL INDEX）之间是有区别的。唯一性约束相当于创建了一个约束和普通索引，目的是保证字段的正确性，而普通索引只是提升数据检索的速度，并不对字段的唯一性进行约束。

NOT NULL 约束。对字段定义了 NOT NULL，即表明该字段不应为空，必须有取值。

DEFAULT，表明了字段的默认值。如果在插入数据的时候，这个字段没有取值，就设置为默认值。

CHECK 约束，用来检查特定字段取值范围的有效性，CHECK 约束的结果不能为 FALSE。


### 设计表原则

三少一多”原则：

1. 数据表的个数越少越好。

RDBMS 的核心在于对实体和联系的定义，也就是 E-R 图（Entity Relationship Diagram），数据表越少，证明实体和联系设计得越简洁，既方便理解又方便操作。

2. 数据表中的字段个数越少越好

字段个数越多，数据冗余的可能性越大。设置字段个数少的前提是各个字段相互独立，而不是某个字段的取值可以由其他字段计算出来。当然字段个数少是相对的，我们通常会在数据冗余和检索效率中进行平衡。

3. 数据表中联合主键的字段个数越少越好

设置主键是为了确定唯一性，当一个字段无法确定唯一性的时候，就需要采用联合主键的方式（也就是用多个字段来定义一个主键）。联合主键中的字段越多，占用的索引空间越大，不仅会加大理解难度，还会增加运行时间和索引空间，因此联合主键的字段个数越少越好。

4. 使用主键和外键越多越好

数据库的设计实际上就是定义各种表，以及各种字段之间的关系。这些关系越多，证明这些实体之间的冗余度越低，利用度越高。这样做的好处在于不仅保证了数据表之间的独立性，还能提升相互之间的关联使用率。

外键本身是为了实现强一致性，所以如果需要正确性>性能的话，还是建议使用外键，它可以让我们在数据库的层面保证数据的完整性和一致性。

在SQL学习之初，包括在系统最初设计的时候，还是建议采用规范的数据库设计，也就是采用外键来对数据表进行约束。因为这样可以建立一个强一致性，可靠性高的数据库结构，也不需要在业务层来实现过多的检查。

很多互联网的公司，尤其是超大型的数据应用场景，大量的插入，更新和删除在外键的约束下会降低性能，同时数据库在水平拆分和分库的情况下，数据库端也做不到执行外键约束。另外，在高并发的情况下，外键的存在也会造成额外的开销。因为每次更新数据，都需要检查另外一张表的数据，也容易造成死锁。

在项目后期，业务量增大的情况下，你需要更多考虑到数据库性能问题，可以取消外键的约束，转移到业务层来实现。而且在大型互联网项目中，考虑到分库分表的情况，也会降低外键的使用。

不过在SQL学习，以及项目早期，还是建议你使用外键。在项目后期，你可以分析有哪些外键造成了过多的性能消耗。一般遵循2/8原则，会有20%的外键造成80%的资源效率，你可以只把这20%的外键进行开放，采用业务层逻辑来进行实现，当然你需要保证业务层的实现没有错误。不同阶段，考虑的问题不同。当用户和业务量增大的时候，对于大型互联网应用，也会通过减少外键的使用，来减低死锁发生的概率，提高并发处理能力。

## SELECT

```sql

SELECT name FROM heros; — 简单的，查询列

SELECT name, hp_max, mp_max, attack_max, defense_max FROM heros — 查询多个列

SELECT * FROM heros — 所有列

SELECT name AS n, hp_max AS hm, mp_max AS mm, attack_max AS am, defense_max AS dm FROM heros — 在结果中，起别名


SELECT '王者荣耀' as platform, name FROM heros — 查询常数，虚构列，platform是列名，王者荣耀 是值

SELECT 123 as platform, name FROM heros — 这是数字


SELECT DISTINCT attack_range FROM heros — 去除重复行，DISTINCT 必须在前面。

SELECT DISTINCT attack_range, name FROM heros — 带上多个列，DISTINCT 是对所有列名的组合进行去重


SELECT name, hp_max FROM heros ORDER BY hp_max DESC; — 排序

— 1. ORDER BY 后面可以有一个或多个列名，如果是多个，先按第一个排序，第一个重复的，按第二个排序，以此类推。
— 2. ASC 代表递增排序，DESC 代表递减排序。如果没有注明排序规则，默认情况下是按照 ASC 递增排序。我们很容易理解 ORDER BY 对数值类型字段的排序规则，但如果排序字段类型为文本数据，就需要参考数据库的设置方式了，这样才能判断 A 是在 B 之前，还是在 B 之后。比如使用 MySQL 在创建字段的时候设置为 BINARY 属性，就代表区分大小写。
— 3. ORDER BY 可以使用非选择列进行排序，所以即使在 SELECT 后面没有这个列名，你同样可以放到 ORDER BY 后面进行排序。
— 4. ORDER BY 通常位于 SELECT 语句的最后一条子句，否则会报错。


SELECT name, hp_max FROM heros ORDER BY hp_max DESC LIMIT 5 — 返回约束结果

— ⚠️ 约束返回结果的数量，在不同的 DBMS 中使用的关键字可能不同，自行查阅。

```

{

FileSort 和 Index 排序：

Index 排序中，索引可以保证数据的有序性，不需要再进行排序，效率更高

FileSort 排序则一般在内存中进行排序，占用 CPU 较多。如果待排结果较大，会产生临时文件 I/O 到磁盘进行排序的情况，效率较低。

建议：

1. explain查看执行计划，看下优化器是什么排序。

2. SQL 中，可以在 WHERE 子句和 ORDER BY 子句中使用索引，目的是在 WHERE 子句中避免全表扫描，在 ORDER BY 子句避免使用 FileSort 排序。某些情况下全表扫描，或者 FileSort 排序不一定比索引慢。

3. 尽量使用 Index 完成 ORDER BY 排序。如果 WHERE 和 ORDER BY 后面是相同的列就使用单索引列；如果不同就使用联合索引。

4. 无法使用 Index 时，需要对 FileSort 方式进行调优。

}


关键词顺序不能错：
SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...


SELECT 语句的执行顺序：
FROM > WHERE > GROUP BY > HAVING > SELECT 的字段 > DISTINCT > ORDER BY > LIMIT

```sql
SELECT DISTINCT player_id, player_name, count(*) as num # 顺序 5
FROM player JOIN team ON player.team_id = team.team_id # 顺序 1
WHERE height > 1.80 # 顺序 2
GROUP BY player.team_id # 顺序 3
HAVING num > 2 # 顺序 4
ORDER BY num DESC # 顺序 6
LIMIT 2 # 顺序 7
```

**在 SELECT 语句执行这些步骤的时候，每个步骤都会产生一个虚拟表，然后将这个虚拟表传入下一个步骤中作为输入。**

执行顺序：
1. 多张表联查。先两张表求笛卡尔积得到 虚拟表1，在表1的基础上进行ON筛选得到表2，在表2的基础处理外部行，得到表3，以此类推，直到所有表处理完，得到原始数据表n。
2. 表n进过WHERE阶段，得到表n2。
3. 表n2进行GROUP 和 HAVING 阶段，分组和分组过滤，得到表n3和表n4。
4. 经过 SELECT 阶段提取列 和 DISTINCT 阶段过滤重复行，得到表n5和表n6。
5. ORDER BY 阶段，得到虚拟表n7。
6. LIMIT 阶段，得到最终的结果，对应的是虚拟表n8。


## 数据过滤

比较运算符：自行查阅

```sql

SELECT name, hp_max FROM heros WHERE hp_max > 6000;

SELECT name, hp_max FROM heros WHERE hp_max BETWEEN 5399 AND 6811;

SELECT name, hp_max FROM heros WHERE hp_max IS NULL;
```

逻辑运算符：自行查阅

```sql
SELECT name, hp_max, mp_max FROM heros WHERE hp_max > 6000 AND mp_max > 1700 ORDER BY (hp_max+mp_max) DESC;

SELECT name, hp_max, mp_max FROM heros WHERE (hp_max+mp_max) > 8000 OR hp_max > 6000 AND mp_max > 1700 ORDER BY (hp_max+mp_max) DESC

SELECT name, hp_max, mp_max FROM heros WHERE ((hp_max+mp_max) > 8000 OR hp_max > 6000) AND mp_max > 1700 ORDER BY (hp_max+mp_max) DESC

SELECT name, role_main, role_assist, hp_max, mp_max, birthdate
FROM heros 
WHERE (role_main IN ('法师', '射手') OR role_assist IN ('法师', '射手')) 
AND DATE(birthdate) NOT BETWEEN '2016-01-01' AND '2017-01-01'
ORDER BY (hp_max + mp_max) DESC
```

通配符，⚠️不同 DBMS 对通配符的定义不同

```sql
SELECT name FROM heros WHERE name LIKE '% 太 %’;
```

📝 如果是 % 太 % 不会使用索引，因为最前原则，太 % 可以。


## 函数

⚠️ 函数的问题：大部分 DBMS 会有自己特定的函数，这就意味着采用 SQL 函数的代码可移植性是很差的，因此在使用函数的时候需要特别注意。

### 1. 算术函数：自行查阅

```sql
SELECT ABS(-2)，— 运行结果为 2。
SELECT MOD(101,3)，— 运行结果 2。
SELECT ROUND(37.25,1)，— 运行结果 37.3。
```

### 2. 字符串函数：自行查阅

```sql
SELECT CONCAT('abc', 123)，运行结果为 abc123。
SELECT LENGTH('你好')，运行结果为 6。
SELECT CHAR_LENGTH('你好')，运行结果为 2。
SELECT LOWER('ABC')，运行结果为 abc。
SELECT UPPER('abc')，运行结果 ABC。
SELECT REPLACE('fabcd', 'abc', 123)，运行结果为 f123d。
SELECT SUBSTRING('fabcd', 1,3)，运行结果为 fab。
```

### 3. 日期函数：自行查阅

```sql
SELECT CURRENT_DATE()，运行结果为 2019-04-03。
SELECT CURRENT_TIME()，运行结果为 21:26:34。
SELECT CURRENT_TIMESTAMP()，运行结果为 2019-04-03 21:26:34。
SELECT EXTRACT(YEAR FROM '2019-04-03')，运行结果为 2019。
SELECT DATE('2019-04-01 12:00:05')，运行结果为 2019-04-01。
```

⚠️ DATE 日期格式必须是 yyyy-mm-dd 的形式。如果要进行日期比较，就要使用 DATE 函数，不要直接使用日期与字符串进行比较。

### 4. 转换函数：自行查阅

```sql
SELECT CAST(123.123 AS INT)，运行结果会报错。
SELECT CAST(123.123 AS DECIMAL(8,2))，运行结果为 123.12。
SELECT COALESCE(null,1,2)，运行结果为 1。
```

⚠️ CAST 函数在转换数据类型的时候，不会四舍五入，如果原数值有小数，那么转换为整数类型的时候就会报错。不过你可以指定转化的小数类型，在 MySQL 和 SQL Server 中，你可以用DECIMAL(a,b)来指定，其中 a 代表整数部分和小数部分加起来最大的位数，b 代表小数位数，比如DECIMAL(8,2)代表的是精度为 8 位（整数加小数位数最多为 8 位），小数位数为 2 位的数据类型。所以SELECT CAST(123.123 AS DECIMAL(8,2))的转换结果为123.12。

### 5. 聚集函数

聚集函数，它是对一组数据进行汇总的函数，输入的是一组数据的集合，输出的是单个值。比如 最大值，最小值，平均值。

```sql
SELECT COUNT(*) FROM heros WHERE hp_max > 6000

SELECT COUNT(role_assist) FROM heros WHERE hp_max > 6000

SELECT MAX(hp_max) FROM heros WHERE role_main = '射手' or role_assist = '射手'


— 可以在一条 SELECT 语句中进行多项聚集函数的查询👇

SELECT COUNT(*), AVG(hp_max), MAX(mp_max), MIN(attack_max), SUM(defense_max) FROM heros WHERE role_main = '射手' or role_assist = '射手’;  


— AVG、MAX、MIN 等聚集函数会自动忽略值为 NULL 的数据行
— MAX 和 MIN 函数也可以用于字符串类型数据的统计，如果是英文字母，则按照 A—Z 的顺序排列，越往后，数值越大。如果是汉字则按照全拼拼音进行排列。
— 把 name 字段统一转化为 gbk 类型，使用CONVERT(name USING gbk)，然后再使用 MIN 和 MAX 取最小值和最大值。👇

SELECT MIN(CONVERT(name USING gbk)), MAX(CONVERT(name USING gbk)) FROM heros;


SELECT COUNT(DISTINCT hp_max) FROM heros; — 对不同的取值进行聚集。

SELECT ROUND(AVG(DISTINCT hp_max), 2) FROM heros

SELECT COUNT(*), role_main FROM heros GROUP BY role_main — 对数据进行分组，并进行聚集统计

SELECT COUNT(*), role_assist FROM heros GROUP BY role_assist

SELECT COUNT(*) as num, role_main, role_assist FROM heros GROUP BY role_main, role_assist ORDER BY num DESC — 多个字段分组


— 使用 HAVING，对分组进行过滤 👇

SELECT COUNT(*) as num, role_main, role_assist FROM heros GROUP BY role_main, role_assist HAVING num > 5 ORDER BY num DESC

SELECT COUNT(*) as num, role_main, role_assist FROM heros WHERE hp_max > 6000 GROUP BY role_main, role_assist HAVING num > 5 ORDER BY num DESC

— 使用 GROUP BY 进行分组，如果想让输出的结果有序，可以在 GROUP BY 后使用 ORDER BY。因为 GROUP BY 只起到了分组的作用，排序还是需要通过 ORDER BY 来完成。

```

COUNT(*)和COUNT(1)
1. MyISAM 专门维护了一个值。
2. InnoDB 会找一个Size小的索引去全表扫描。

### 6. Examples

```sql
SELECT name, ROUND(attack_growth,1) FROM heros;

SELECT MAX(hp_max) FROM heros;

SELECT name, hp_max FROM heros WHERE hp_max = (SELECT MAX(hp_max) FROM heros);

SELECT CHAR_LENGTH(name), name FROM heros;

SELECT name, EXTRACT(YEAR FROM birthdate) AS birthdate FROM heros WHERE birthdate is NOT NULL

SELECT name, YEAR(birthdate) AS birthdate FROM heros WHERE birthdate is NOT NULL

SELECT * FROM heros WHERE DATE(birthdate)>'2016-10-01'

SELECT * FROM heros WHERE birthdate>'2016-10-01'  — ⚠️ 不安全，你无法确认 birthdate 的数据类型是字符串，还是 datetime 类型，如果你想对日期部分进行比较，那么使用DATE(birthdate)来进行比较是更安全的。

SELECT AVG(hp_max), AVG(mp_max), MAX(attack_max) FROM heros WHERE DATE(birthdate)>'2016-10-01'
```

## 子查询

### 非关联子查询

字查询的结果仅仅作为查询的条件执行。 👇

```sql
SELECT player_name, height FROM player WHERE height = (SELECT max(height) FROM player);
```

### 关联子查询

子查询中的表用到了外部的表，并进行了条件关联，因此每执行一次外部查询，子查询都要重新计算一次。 👇

```sql
SELECT player_name, height, team_id FROM player AS a WHERE height > (SELECT avg(height) FROM player AS b WHERE a.team_id = b.team_id) — 统计球队的平均身高，然后筛选身高大于的。
```

### EXISTS 子查询 

```sql
SELECT player_id, team_id, player_name FROM player WHERE EXISTS (SELECT player_id FROM player_score WHERE player.player_id = player_score.player_id)


SELECT player_id, team_id, player_name FROM player WHERE NOT EXISTS (SELECT player_id FROM player_score WHERE player.player_id = player_score.player_id)

```

### 集合比较子查询

```sql
— 在对 cc 建立索引的条件下，
— 表A大于表B，IN 效率高，非关联查询，因为表B对cc索引，表B的集合少，表A判断快。
— 表A小于表B，EXIST 效率高，关联查询，表A是外部表，肯定是越小越好。
SELECT * FROM A WHERE cc IN (SELECT cc FROM B)
SELECT * FROM A WHERE EXIST (SELECT cc FROM B WHERE B.cc=A.cc)

📝 可以这么理解哦，IN的时候，B是外循环，A是内循环，EXIST 的时候，B是内循环，A是外循环，外循环越小越好。


SELECT player_id, player_name, height FROM player WHERE height > ANY (SELECT height FROM player WHERE team_id = 1002)

SELECT player_id, player_name, height FROM player WHERE height > ALL (SELECT height FROM player WHERE team_id = 1002)
```

##### 将子查询作为计算字段

```sql
SELECT team_name, (SELECT count(*) FROM player WHERE player.team_id = team.team_id) AS player_num FROM team
```

## 表连接 SQL92 SQL99

表的组成是基于关系模型的，所以一个表就是一个关系。一个数据库中可以包括多个表，也就是存在多种数据之间的关系。而我们之所以能使用 SQL 语言对各个数据表进行复杂查询，核心就在于连接，它可以用一条 SELECT 语句在多张表之间进行查询。你也可以理解为，关系型数据库的核心之一就是连接。

SQL92 和 SQL99 是经典的 SQL 标准，也分别叫做 SQL-2 和 SQL-3 标准。

最重要的 SQL 标准就是 SQL92 和 SQL99。一般来说 SQL92 的形式更简单，但是写的 SQL 语句会比较长，可读性较差。而 SQL99 相比于 SQL92 来说，语法更加复杂，但可读性更强。

建议多表连接使用 SQL99 标准，因为层次性更强，可读性更强：
```sql
SELECT ...
FROM table1
    JOIN table2 ON table1 和 table2 的连接条件
        JOIN table3 ON table2 和 table3 的连接条件
```

### 不同 DBMS 中使用连接需要注意的地方

1. 不是所有的 DBMS 都支持全外连接。虽然 SQL99 标准提供了全外连接，但不是所有的 DBMS 都支持。不仅 MySQL 不支持，Access、SQLite、MariaDB 等数据库软件也不支持。不过在 Oracle、DB2、SQL Server 中是支持的。

2. Oracle 没有表别名 AS。为了让 SQL 查询语句更简洁，我们经常会使用表别名 AS，不过在 Oracle 中是不存在 AS 的，使用表别名的时候，直接在表名后面写上表别名即可，比如 player p，而不是 player AS p。

3. SQLite 的外连接只有左连接

### 性能注意

1. 控制连接表的数量。多表连接就相当于嵌套 for 循环一样，非常消耗资源，会让 SQL 查询性能下降得很严重，因此不要连接不必要的表。在许多 DBMS 中，也都会有最大连接表的限制。

2. 在连接时不要忘记 WHERE 语句。多表连接的目的不是为了做笛卡尔积，而是筛选符合条件的数据行，因此在多表连接的时候不要忘记了 WHERE 语句，这样可以过滤掉不必要的数据行返回。

3. 使用自连接而不是子查询。在许多 DBMS 的处理过程中，对于自连接的处理速度要比子查询快得多。你可以这样理解：子查询实际上是通过未知表进行查询后的条件判断，而自连接是通过已知的自身数据表进行条件判断，因此在大部分 DBMS 中都对自连接处理进行了优化。

#### 笛卡尔连接SQL92/交叉连接SQL99

笛卡尔乘积，也称为交叉连接，英文是 CROSS JOIN，是一个数学运算。假设我有两个集合 X 和 Y，那么 X 和 Y 的笛卡尔积就是 X 和 Y 的所有可能组合。👇

```sql
SELECT * FROM player, team — SQL92

SELECT * FROM player CROSS JOIN team — SQL99

SELECT * FROM t1 CROSS JOIN t2 CROSS JOIN t3 — 多表
```

### 等值连接SQ92/自然连接SQL99

**一般来说在 SQL99 中，我们需要连接的表会采用 JOIN 进行连接，ON 指定了连接条件，后面可以是等值连接，也可以采用非等值连接。**

```sql
SELECT player_id, player.team_id, player_name, height, team_name FROM player, team WHERE player.team_id = team.team_id — SQL92

SELECT player_id, a.team_id, player_name, height, team_name FROM player AS a, team AS b WHERE a.team_id = b.team_id — SQL92，使用别名

SELECT player_id, player.team_id, player_name, height, team_name FROM player AS a, team AS b WHERE a.team_id = b.team_id — ❌报错，如果我们使用了表的别名，在查询字段中就只能使用别名进行代替，不能使用原有的表名。

SELECT player_id, team_id, player_name, height, team_name FROM player NATURAL JOIN team — SQL99
— SQL99 中用 NATURAL JOIN 替代了 WHERE player.team_id = team.team_id。

SELECT player_id, player.team_id, player_name, height, team_name FROM player JOIN team ON player.team_id = team.team_id — SQL99，等值连接

SELECT player_id, team_id, player_name, height, team_name FROM player JOIN team USING(team_id) — SQL99，USING 等值连接，简化
```

### 非等值连接SQL92/99

```sql
SELECT p.player_name, p.height, h.height_level
FROM player AS p, height_grades AS h
WHERE p.height BETWEEN h.height_lowest AND h.height_highest — SQL92，如果连接多个表的条件是等号时，就是等值连接，其他的运算符连接就是非等值查询。

SELECT p.player_name, p.height, h.height_level
FROM player as p JOIN height_grades as h
ON height BETWEEN h.height_lowest AND h.height_highest — SQL99
```

### 外连接

除了查询满足条件的记录以外，外连接还可以查询某一方不满足条件的记录。两张表的外连接，会有一张是主表，另一张是从表。如果是多张表的外连接，那么第一张表是主表，即显示全部的行，而第剩下的表则显示对应连接的信息。

在 SQL92 中采用（+）代表从表所在的位置，而且在 SQL92 中，只有左外连接和右外连接，没有全外连接。

在 SQL99 中：
1. 左外连接：LEFT JOIN 或 LEFT OUTER JOIN
2. 右外连接：RIGHT JOIN 或 RIGHT OUTER JOIN
3. 全外连接：FULL JOIN 或 FULL OUTER JOIN

```sql
— 左外连接，就是指左边的表是主表，需要显示左边表的全部行，而右侧的表是从表，（+）表示哪个是从表。
SELECT * FROM player, team where player.team_id = team.team_id(+)  — SQL92
SELECT * FROM player LEFT JOIN team ON player.team_id = team.team_id — SQL99

— 右外连接，指的就是右边的表是主表，需要显示右边表的全部行，而左侧的表是从表
SELECT * FROM player, team where player.team_id(+) = team.team_id — SQL92
SELECT * FROM player RIGHT JOIN team ON player.team_id = team.team_id — SQL99


— 全外连接
— ⚠️ MySQL 不支持全外连接，否则的话全外连接会返回左表和右表中的所有行。当表之间有匹配的行，会显示内连接的结果。当某行在另一个表中没有匹配时，那么会把另一个表中选择的列显示为空值。
— 全外连接的结果 = 左右表匹配的数据 + 左表没有匹配到的数据 + 右表没有匹配到的数据。
SELECT * FROM player FULL JOIN team ON player.team_id = team.team_id
```

### 自连接

对同一个表进行操作。也就是说查询条件使用了当前表的字段。

```sql
SELECT b.player_name, b.height FROM player as a , player as b WHERE a.player_name = '布雷克 - 格里芬' and a.height < b.height — 查看比布雷克·格里芬高的球员都有谁，以及他们的对应身高

— 不使用自连接👇
SELECT height FROM player WHERE player_name = '布雷克 - 格里芬'
SELECT player_name, height FROM player WHERE height > 2.08


SELECT b.player_name, b.height FROM player as a JOIN player as b ON a.player_name = '布雷克 - 格里芬' and a.height < b.height — SQL99
```

## 视图

视图是一张虚拟表，目的是为不同的人员提供各自权限能够阅读的信息，还有简化封装查询的作用。

⚠️ 视图是虚拟表，它只是封装了底层的数据表查询接口，因此有些 RDBMS 不支持对视图创建索引（有些 RDBMS 则支持，比如新版本的 SQL Server）。

⚠️ 视图是虚拟表，临时表是实体表，临时表只在当前连接存在，关闭连接后就会自动释放。

```sql
CREATE VIEW player_above_avg_height AS
SELECT player_id, height
FROM player
WHERE height > (SELECT AVG(height) from player) — 创建视图

SELECT * FROM player_above_avg_height — 查看，正常的表

CREATE VIEW player_above_above_avg_height AS
SELECT player_id, height
FROM player
WHERE height > (SELECT AVG(height) from player_above_avg_height) — 嵌套视图

ALTER VIEW player_above_avg_height AS
SELECT player_id, player_name, height
FROM player
WHERE height > (SELECT AVG(height) from player) — 更新视图，增加一个 player_name 字段

⚠️ SQLite 不支持视图的修改，仅支持只读视图，也就是说你只能使用 CREATE VIEW 和 DROP VIEW，如果想要修改视图，就需要先 DROP 然后再 CREATE。

DROP VIEW player_above_avg_height — 删除视图
```

### 视图的作用

1. 利用视图完成复杂的连接。将复杂的查询语句拆开。

```sql
CREATE VIEW player_height_grades AS
SELECT p.player_name, p.height, h.height_level
FROM player as p JOIN height_grades as h
ON height BETWEEN h.height_lowest AND h.height_highest

SELECT * FROM player_height_grades WHERE height >= 1.90 AND height <= 2.08
```

2. 利用视图对数据进行格式化，便于展示

```sql
CREATE VIEW player_team AS 
SELECT CONCAT(player_name, '(' , team.team_name , ')') AS player_team FROM player JOIN team WHERE player.team_id = team.team_id — 使用 CONCAT 函数，将 player_name 字段和 team_name 字段进行拼接。
```

3. 使用视图与计算字段

```sql
CREATE VIEW game_player_score AS
SELECT game_id, player_id, (shoot_hits-shoot_3_hits)*2 AS shoot_2_points, shoot_3_hits*3 AS shoot_3_points, shoot_p_hits AS shoot_p_points, score  FROM player_score

SELECT * FROM game_player_score
```

## 存储过程

优点：
1. 一次编译，多次使用。
2. 减少工作量。
3. 安全性强。设置用户权限的。
4. 减少网络传输。代码封装。

⚠️ 争议：
1. 它的可移植性差，存储过程不能跨数据库移植。
2. 调试困难，只有少数 DBMS 支持存储过程的调试。对于复杂的存储过程来说，开发和维护都不容易。
3. 存储过程的版本管理也很困难，比如数据表索引发生变化了，可能会导致存储过程失效。我们在开发软件的时候往往需要进行版本管理，但是存储过程本身没有版本控制，版本迭代更新的时候很麻烦。
4. 它不适合高并发的场景，高并发的场景需要减少数据库的压力，有时数据库会采用分库分表的方式，而且对可扩展性要求很高，在这种情况下，存储过程会变得难以维护，增加数据库的压力，显然就不适用了。

```sql

— 定义 👇
CREATE PROCEDURE 存储过程名称 ([参数列表])
BEGIN
    需要执行的语句
END 

— 实例 👇
DELIMITER // — 默认的；是结束符
CREATE PROCEDURE `add_num`(IN n INT) — IN传入参数，不返回，OUT是返回结果吃，INOUT两者。
BEGIN
       DECLARE i INT;  — 声明
       DECLARE sum INT;
       
       SET i = 1; — 赋值
       SET sum = 0;
       WHILE i <= n DO — 分支自己查
              SET sum = sum + i;
              SET i = i +1;
       END WHILE;
       SELECT sum;
END //
DELIMITER ;

— OUT实例 👇

CREATE PROCEDURE `get_hero_scores`(
       OUT max_max_hp FLOAT,
       OUT min_max_mp FLOAT,
       OUT avg_max_attack FLOAT,  
       s VARCHAR(255)
       )
BEGIN
       SELECT MAX(hp_max), MIN(mp_max), AVG(attack_max) FROM heros WHERE role_main = s INTO max_max_hp, min_max_mp, avg_max_attack;
— 把从数据表中查询的结果存放到变量中，也就是为变量赋值。
END

CALL get_hero_scores(@max_max_hp, @min_max_mp, @avg_max_attack, '战士');
SELECT @max_max_hp, @min_max_mp, @avg_max_attack;

```

## 事务

Oracle 是支持事务的，而在 MySQL 中，则需要选择适合的存储引擎才可以支持事务。如果你使用的是 MySQL，可以通过 SHOW ENGINES 命令来查看当前 MySQL 支持的存储引擎都有哪些，以及这些存储引擎是否支持事务。

事务的常用控制语句：
1. START TRANSACTION 或者 BEGIN，作用是显式开启一个事务。
2. COMMIT：提交事务。当提交事务后，对数据库的修改是永久性的。
3. ROLLBACK 或者 ROLLBACK TO [SAVEPOINT]，意为回滚事务。意思是撤销正在进行的所有没有提交的修改，或者将事务回滚到某个保存点。
4. SAVEPOINT：在事务中创建保存点，方便后续针对保存点进行回滚。一个事务中可以存在多个保存点。
5. RELEASE SAVEPOINT：删除某个保存点。
6. SET TRANSACTION，设置事务的隔离级别。

设置事务不要自动提交：`set autocommit =0;`

`SET @@completion_type = 1;` 👇

1. completion=0，这是默认情况。也就是说当我们执行 COMMIT 的时候会提交事务，在执行下一个事务时，还需要我们使用 START TRANSACTION 或者 BEGIN 来开启。
2. completion=1，这种情况下，当我们提交事务后，相当于执行了 COMMIT AND CHAIN，也就是开启一个链式事务，即当我们提交事务之后会开启一个相同隔离级别的事务（隔离级别会在下一节中进行介绍）。
3. completion=2，这种情况下 COMMIT=COMMIT AND RELEASE，也就是当我们提交后，会自动与服务器断开连接。

## 隔离级别

### 异常

脏读：读到了其他事务还没有提交的数据。

不可重复读：对某数据进行读取，发现两次读取的结果不同，也就是说没有读到相同的内容。这是因为有其他事务对这个数据同时进行了修改或删除。

幻读：事务 A 根据条件查询得到了 N 条数据，但此时事务 B 更改或者增加了 M 条符合事务 A 查询条件的数据，这样当事务 A 再次进行查询的时候发现会有 N+M 条数据，产生了幻读。

【 幻读 是对读到的数据加锁，但是没有加间隙锁，导致新增的满足条件的数据没有锁。 】

### 隔离级别

读未提交（READ UNCOMMITTED ）

读已提交（READ COMMITTED），Oracle 和 SQL Server 默认级别，如果避免RR和SE，需要加锁

可重复读（REPEATABLE READ），MySQL默认

可串行化（SERIALIZABLE），牺牲并发性

## 游标

游标 提供了一种灵活的操作方式，可以让我们从数据结果集中每次提取一条数据记录进行操作。在 SQL 中，游标是一种临时的数据库对象，可以指向存储在数据库表中的数据行指针。

好处：可以解决复杂数据处理问题，面向过程编程，对数据行逐条扫描处理。

坏处：加锁，影响并发效率，容易造成内存不足。

建议：有代替方案使用代替方案。


1. 定义

```sql
DECLARE cursor_name CURSOR FOR select_statement — MySQL，SQL Server，DB2 和 MariaDB

DECLARE cursor_name CURSOR IS select_statement — Oracle 或者 PostgreSQL

DECLARE cur_hero CURSOR FOR 
	SELECT hp_max FROM heros; — MySQL例子
```

2. 打开

```sql
OPEN cursor_name
```

3. 取得数据

```sql
FETCH cursor_name INTO var_name ...
```

4. 关闭

```sql
CLOSE cursor_name
```

5. 释放

我们一定要养成释放游标的习惯，否则游标会一直存在于内存中，直到进程结束后才会自动释放。当你不需要使用游标的时候，释放游标可以减少资源浪费。

```sql
DEALLOCATE PREPARE
```

### 示例

```sql
CREATE PROCEDURE `calc_hp_max`()
BEGIN
       -- 创建接收游标的变量
       DECLARE hp INT;  
 
       -- 创建总数变量 
       DECLARE hp_sum INT DEFAULT 0;
       -- 创建结束标志变量  
     DECLARE done INT DEFAULT false;
       -- 定义游标     
       DECLARE cur_hero CURSOR FOR SELECT hp_max FROM heros;
       -- 指定游标循环结束时的返回值  
     DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true;  
       
       OPEN cur_hero;
       read_loop:LOOP 
       FETCH cur_hero INTO hp;
       -- 判断游标的循环是否结束  
       IF done THEN  
                     LEAVE read_loop;
       END IF; 
              
       SET hp_sum = hp_sum + hp;
       END LOOP;
       CLOSE cur_hero;
       SELECT hp_sum;
       DEALLOCATE PREPARE cur_hero;
END
```

### 复杂示例

假设我们想要对英雄的物攻成长（对应 attack_growth）进行升级，在新版本中大范围提升英雄的物攻成长数值，但是针对不同的英雄情况，提升的幅度也不同，具体提升的方式如下。
如果这个英雄原有的物攻成长小于 5，那么将在原有基础上提升 7%-10%。如果物攻成长的提升空间（即最高物攻 attack_max- 初始物攻 attack_start）大于 200，那么在原有的基础上提升 10%；如果物攻成长的提升空间在 150 到 200 之间，则提升 8%；如果物攻成长的提升空间不足 150，则提升 7%。
如果原有英雄的物攻成长在 5—10 之间，那么将在原有基础上提升 5%。
如果原有英雄的物攻成长大于 10，则保持不变。
以上所有的更新后的物攻成长数值，都需要保留小数点后 3 位。👇👇👇

```sql
CREATE PROCEDURE `alter_attack_growth`()
BEGIN
       -- 创建接收游标的变量
       DECLARE temp_id INT;  
       DECLARE temp_growth, temp_max, temp_start, temp_diff FLOAT;  
 
       -- 创建结束标志变量  
       DECLARE done INT DEFAULT false;
       -- 定义游标     
       DECLARE cur_hero CURSOR FOR SELECT id, attack_growth, attack_max, attack_start FROM heros;
       -- 指定游标循环结束时的返回值  
       DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true;  
       
       OPEN cur_hero;  
       FETCH cur_hero INTO temp_id, temp_growth, temp_max, temp_start;
       REPEAT
                     IF NOT done THEN
                            SET temp_diff = temp_max - temp_start;
                            IF temp_growth < 5 THEN
                                   IF temp_diff > 200 THEN
                                          SET temp_growth = temp_growth * 1.1;
                                   ELSEIF temp_diff >= 150 AND temp_diff <=200 THEN
                                          SET temp_growth = temp_growth * 1.08;
                                   ELSEIF temp_diff < 150 THEN
                                          SET temp_growth = temp_growth * 1.07;
                                   END IF;                       
                            ELSEIF temp_growth >=5 AND temp_growth <=10 THEN
                                   SET temp_growth = temp_growth * 1.05;
                            END IF;
                            UPDATE heros SET attack_growth = ROUND(temp_growth,3) WHERE id = temp_id;
                     END IF;
       FETCH cur_hero INTO temp_id, temp_growth, temp_max, temp_start;
       UNTIL done = true END REPEAT;
       
       CLOSE cur_hero;
       DEALLOCATE PREPARE cur_hero;
END
```

## ORM

ORM 解决的问题：提供了一种持久化模式，可以高效地对数据库进行访问。ORM 的英文是 Object Relation Mapping，中文叫对象关系映射。它是 RDBMS 和业务实体对象之间的一个映射，从图中你也能看到，它可以把底层的 RDBMS 封装成业务实体对象，提供给业务逻辑层使用。程序员往往关注业务逻辑层面，而不是底层数据库该如何访问，以及如何编写 SQL 语句获取数据等等。采用 ORM，就可以从数据库的设计层面转化成面向对象的思维。

随着项目规模的增大，在代码层编写 SQL 语句访问数据库会降低开发效率，也会提升维护成本，因此越来越多的开发人员会采用基于 ORM 的方式来操作数据库。这样做的好处就是一旦定义好了对象模型，就可以让它们简单可复用，从而不必关注底层的数据库访问细节，我们只要将注意力集中到业务逻辑层面就可以了。由此还可以带来另一点好处，那就是即便数据库本身进行了更换，在业务逻辑代码上也不会有大的调整。这是因为 ORM 抽象了数据的存取，同时也兼容多种 DBMS，我们不用关心底层采用的到底是哪种 DBMS，是 MySQL，SQL Server，PostgreSQL 还是 SQLite。

采用 ORM 当然也会付出一些代价，比如性能上的一些损失。面对一些复杂的数据查询，ORM 会显得力不从心。虽然可以实现功能，但相比于直接编写 SQL 查询语句来说，ORM 需要编写的代码量和花费的时间会比较多，这种情况下，直接编写 SQL 反而会更简单有效。

没有一种方式是一劳永逸的，在实际工作中，我们需要根据需求选择适合的方式。


---


## 调优

调优的目标就是让数据库运行更快，响应更快，吞吐更大，满足要求。

调优三个考虑点：

1. 选择大于努力，选择好 DBMS，数据表设计。
2. SQL查询优化：逻辑查询、物理查询优化。
3. 外援：Redis/Memchched，主从，分库分表。

### 要求哪里来

1. 用户的反馈，慢了。
2. 日志分析，数据库日志发现很多异常情况。
3. 服务器资源监控，CPU、内存、IO 使用情况。
4. 数据库内部状态监控，活动会话是否出于非常繁忙，SQL堆积，事务和锁等待。

### 调优的维护

1. 选择合适的DBMS

2. 表设计优化，第三范式、反范式优化、表字段数据类型

第三范式，减少冗余字段，事务更新插入的时候，不需要同时维护多个表。如果涉及到多表联查，可以反范式，用空间换时间。

【 我认为，可以修改主表，然后其他表的冗余字段可以通过异步去处理 + 后来一次确认。 】

字段的数据类型选择，能用数值不要用字符，字符串能短就短，能固定就固定，不能固定就用 VARCHAR。

3. 逻辑查询优化：语句优化

SQL 的查询重写包括了子查询优化、等价谓词重写、视图重写、条件简化、连接消除和嵌套连接消除等。

比如：EXISTS 子查询和 IN 子查询、使用 LIKE 代替 SUBSTRING 函数。

4. 物理查询优化：索引

重复度超过 10%，不可创建索引。其他。

5. 加缓存

6. 库级优化：主从、分库分表

### 优化步骤

观察服务器状态。如果存在周期性波动，加缓存/修改缓存失效策略。

如果还有不规则延迟和卡顿，开启慢查询（long_query_time 设置阈值），然后 EXPLAIN 或 SHOW PROFILING。

如果是 SQL执行时间长，那就 索引设计优化？JOIN表太多？数据表需要优化？

如果是 SQL等待时间长，调优服务器参数（增加数据库缓冲池）。

如果还没有解决，SQL查询是不是到瓶颈了？这个时候 读写分离、分库分表。


InnoDB 缓冲池包括了数据页、索引页、插入缓冲、锁信息、自适应 Hash 和数据字典信息等。

缓冲池的作用就是提升 I/O 效率，而我们进行读取数据的时候存在一个“局部性原理”，也就是说我们使用了一些数据，大概率还会使用它周围的一些数据，因此采用“预读”的机制提前加载，可以减少未来可能的磁盘 I/O 操作。

想了解 缓冲池，参阅《44-SQL必知必会：28 从磁盘I/O的角度理解SQL查询的成本》


## 范式

越高的范式，冗余越低。

1NF 数据表任何属性都是原子的，不可再分，不可能出现某个熟悉 A-1，A-2。

2NF 在 1NF 的基础表示 一张表是一个独立的对象，比如 球员信息表就是球员信息表，比赛表就是比赛信息表，一张表仅表达一个意思。

3NF 在 2NF 的基础上表示 一张表的属性不能传递依赖，比如球员信息表有球队名称，又有球队教练，这个球队名称 和 教练 是有依赖关系的不行。

BCNF（巴斯 - 科德范式），在 3NF 的基础上消除了主属性对候选键的部分依赖或者传递依赖关系。比如商品仓库库存管理员表，在满足3NF的前提下，所有属性不传递依赖，但是如果没有商品了，表也会被随之删除，所以进一步分拆 仓库表、库存表。

反范式在数据量大的场景下查询优化效果非常好，本质上时间换空间。而且数据仓库一般也是反范式设计，因为仓库基本上仅读，不需要修改。

### 属性

1. 超键：能唯一标识元组的属性集叫做超键。

对于球员表来说，超键就是包括球员编号或者身份证号的任意组合，比如（球员编号）（球员编号，姓名）（身份证号，年龄）等。【 能唯一标识 】

2. 候选键：如果超键不包括多余的属性，那么这个超键就是候选键。

候选键就是最小的超键，对于球员表来说，候选键就是（球员编号）或者（身份证号）。【 想想为什么叫候选 】

3. 主键：用户可以从候选键中选择一个作为主键。

主键是我们自己选定，也就是从候选键中选择一个，比如（球员编号）。

4. 外键：如果数据表 R1 中的某属性集不是 R1 的主键，而是另一个数据表 R2 的主键，那么这个属性集就是数据表 R1 的外键。

外键就是球员表中的球队编号，对应到球队表。

5. 主属性：包含在任一候选键中的属性称为主属性。
6. 非主属性：与主属性相对，指的是不包含在任何一个候选键中的属性。

在 player 表中，主属性是（球员编号）（身份证号），其他的属性（姓名）（年龄）（球队编号）都是非主属性。

## 索引

数据量小，索引没有用，反而增加评估时间。

重合度高于10%，也没用。

聚集索引（主键）查询效率比非聚集索引（其他索引）查询效率高。

B+ 树 就是非叶子节点用来存储索引，而叶子节点用来存储数据，分离的设计 很符合磁盘的特性。

现在MySQL对于Hash索引仅仅是 自适应Hash索引。当某个索引值使用非常频繁的时候，它会在 B+ 树索引的基础上再创建一个 Hash 索引，这样让 B+ 树也具备了 Hash 索引的优点。

查看自适应Hash索引是否打开：

```sql
show variables like '%adaptive_hash_index';
```

索引失效：
1. 让索引参与表达式计算。
2. 使用函数。
3. WHERE语句有一个 OR 条件没有索引，其他索引也会失效。
4. LIKE语句 前面是 % 不行，不满足最左原则。
5. 与NULL和NOT NULL判断会失效。
6. 违反 最左原则。

设置 联合索引（宽索引）可以避免回表，最好索引列的过滤能力强（满足条件的行数除以总行树，越小越好）。

一般原则就是三星索引：
1. 在 WHERE 条件所有等值的列，纳入索引。（最小化索引片）
2. GROUP BY 和 ORDER BY 的列纳入索引。 （避免排序）
3. SELECT需要的列，加入索引。 （避免回表查询）

但是，三星索引会让索引变宽，索引变大，得不偿失，所以需要测试比较。


联合索引最左原则，如果遇到范围条件查询，就会停止使用索引。

【 MongoDB 索引的 ESR原则：精确匹配、排序、范围，这样就避免了内存里重新排序。 】


## 锁

```sql

LOCK TABLE product_comment READ; — 读表锁
LOCK TABLE product_comment WRITE; — 写表锁
UNLOCK TABLE;

SELECT comment_id, product_id, comment_text, user_id FROM product_comment WHERE user_id = 912178 LOCK IN SHARE MODE; — 共享锁，读锁

SELECT comment_id, product_id, comment_text, user_id FROM product_comment WHERE user_id = 912178 FOR UPDATE; — 排他锁，写锁

```

意向锁：给更大一级别的空间示意里面是否已经上过锁。如果获取表锁，是不是要检查有没有行锁，这个时候意向锁可以告诉它目前有行锁。

乐观锁（Optimistic Locking）认为对同一数据的并发操作不会总发生，属于小概率事件，不用每次都对数据上锁，也就是不采用数据库自身的锁机制，而是通过程序来实现。在程序上，我们可以采用版本号机制或者时间戳机制实现。

版本号机制：提交的时候看版本号是不是与数据库一致（更新条件），不一致就不会修改。

时间戳机制：与版本号思想一致。

悲观锁（Pessimistic Locking）也是一种思想，对数据被其他事务的修改持保守态度，会通过数据库自身的锁机制来实现，从而保证数据操作的排它性。

防止死锁：
1. 如果涉及多个表，操作复杂，提前锁定所有资源，而不是逐一获取。
2. 如果更新的数据体量大，那就锁升级，行锁升级到表锁。
3. 不同事务并发读写表，可以约定访问表的顺序，采用一定顺序就不会撞。

### MVCC

MVCC 通过乐观锁的方式解决了不可重复读和幻读，降低了死锁的概率。

快照读：读取的是快照数据，比如 不加锁的简单的 SELECT。

当前读：读取最新数据，而不是历史版本的数据。加锁的 SELECT，或者对数据进行增删改都会进行当前读，比如：

```sql
SELECT * FROM player LOCK IN SHARE MODE;

SELECT * FROM player FOR UPDATE;

INSERT INTO player values ... — 📝 所有的更新、插入、新增都是当前读。

DELETE FROM player WHERE ...

UPDATE player SET ...
```

InnoDB 中 MVCC 通过 Undo Log + Read View 来实现，Undo Log 是数据写入历史日志，Read View来根据 事务版本号 和隔离级别选择哪些可见。所以如果是读提交，每次语句都是用 Read View 去看一遍。

InnoDB 在可重复读的下，通过 Next-Key锁 + MVCC 解决幻读。而在 读已提交 下，即使采用了 MVCC 方式也会出现幻读，因为只采用 记录锁（Recrod Locking）。

记录锁：针对单个行记录添加锁。

间隙锁（Gap Locking）：可以帮我们锁住一个范围（索引之间的空隙），但不包括记录本身。采用间隙锁的方式可以防止幻读情况的产生。

Next-Key 锁：帮我们锁住一个范围，同时锁定记录本身，相当于间隙锁 + 记录锁，可以解决幻读的问题。

## 优化器

查询优化器 包含逻辑优化、物理优化，目标就是找到查询的最佳执行计划。

逻辑优化，查询重写，就是通过简化条件、数学关系代数优化语句。

物理优化，分为 基于规则优化（RBO，Rule-Based Optimizer） 和 基于代价优化（CBO，Cost-Based Optimizer），前者像出租车老司机，后者像地图导航。

## CBO

有两张表显示预估代价：mysql.server_cost 和 mysql.engine_cost

```sql
SELECT * FROM mysql.server_cost — 服务器层
```

1. disk_temptable_create_cost，表示临时表文件（MyISAM 或 InnoDB）的创建代价，默认值为 20。
2. disk_temptable_row_cost，表示临时表文件（MyISAM 或 InnoDB）的行代价，默认值 0.5。
3. key_compare_cost，表示键比较的代价。键比较的次数越多，这项的代价就越大，这是一个重要的指标，默认值 0.05。
4. memory_temptable_create_cost，表示内存中临时表的创建代价，默认值 1。
5. memory_temptable_row_cost，表示内存中临时表的行代价，默认值 0.1。
6. row_evaluate_cost，统计符合条件的行代价，如果符合条件的行数越多，那么这一项的代价就越大，因此这是个重要的指标，默认值 0.1。

【 你看，如果更换更快地内存，使用 SSD，代价就可以调小。 】

```sql
SELECT * FROM mysql.engine_cost — 引擎层
```

1. io_block_read_cost，从磁盘中读取一页数据的代价，默认是 1。
2. memory_block_read_cost，从内存中读取一页数据的代价，默认是 0.25。

【 同样，SSD 和 更好的内存。 】

修改代价规则：

```sql
UPDATE mysql.engine_cost
  SET cost_value = 2.0
  WHERE cost_name = 'io_block_read_cost';
FLUSH OPTIMIZER_COSTS;

INSERT INTO mysql.engine_cost(engine_name, device_type, cost_name, cost_value, last_update, comment)
  VALUES ('InnoDB', 0, 'io_block_read_cost', 2,
  CURRENT_TIMESTAMP, 'Using a slower disk for InnoDB');
FLUSH OPTIMIZER_COSTS; — 针对专门的存储引擎
```

代价计算：

** 总代价 = I/O 代价 + CPU 代价 + 内存代价 + 远程代价 **
** IO代价 = 数据页 + 索引页 **
** CPU代价 = W（ IO和CPU转换系数 ）* CPU健比较 和 行估算 **


## 性能分析工具

### 慢查询

```sql
show variables like '%slow_query_log'; — 慢查询是否已经开启

set global slow_query_log='ON'; — 开启，注意加 global，否则报错。这个时候再查看一次，能看到日志文件的名称。


show variables like '%long_query_time%'; — 查看时间

set global long_query_time = 3; — 设置阈值3秒
```

我们可以使用 MySQL 自带的 mysqldumpslow 工具统计慢查询日志（这个工具是个 Perl 脚本，你需要先安装好 Perl），参数：

-s：采用 order 排序的方式，排序方式可以有以下几种。分别是 c（访问次数）、t（查询时间）、l（锁定时间）、r（返回记录）、ac（平均查询次数）、al（平均锁定时间）、ar（平均返回记录数）和 at（平均查询时间）。其中 at 为默认排序方式。
-t：返回前 N 条数据 。
-g：后面可以是正则表达式，对大小写不敏感。

```shell
perl mysqldumpslow.pl -s t -t 2 "C:\ProgramData\MySQL\MySQL Server 8.0\Data\DESKTOP-4BK02RP-slow.log"
```

### EXPLAIN

EXPLAIN 可以帮助我们了解数据表的读取顺序、SELECT 子句的类型、数据表的访问类型、可使用的索引、实际使用的索引、使用的索引长度、上一个表的连接匹配条件、被优化器查询的行的数量以及额外的信息（比如是否使用了外部排序，是否使用了临时表等）等。

拿到慢查询语句后：

```sql
EXPLAIN SELECT comment_id, product_id, comment_text, product_comment.user_id, user_name FROM product_comment JOIN user on product_comment.user_id = user.user_id
```

返回的结果，你看，当牵涉到多个表的时候，执行的顺序是根据 id 从大到小执行的，也就是 id 越大越先执行，当 id 相同时，从上到下执行。

对于参数：

- type 里，all 是最坏的情况，因为采用了全表扫描的方式。index 和 all 差不多，只不过 index 对索引表进行全扫描，这样做的好处是不再需要对数据进行排序，但是开销依然很大。

- range 表示采用了索引范围扫描。

- extra 列中看到 Using index，说明采用了索引覆盖，也就是索引可以覆盖所需的 SELECT 字段，就不需要进行回表，这样就减少了数据查找的开销。

- const 类型和 eq_ref 都使用了主键或唯一索引，不过这两个类型有所区别，const 是与常量进行比较，查询效率会更快，而 eq_ref 通常用于多表联查中。

- ref 类型表示采用了非唯一索引，或者是唯一索引的非唯一性前缀。

- eq_ref 类型是使用主键或唯一索引时产生的访问方式，通常使用在多表联查中。

- system 类型一般用于 MyISAM 或 Memory 表，属于 const 类型的特例。


效率从低到高依次为 all < index < range < index_merge < ref < eq_ref < const/system。我们在查看执行计划的时候，通常希望执行计划至少可以使用到 range 级别以上的连接方式，如果只使用到了 all 或者 index 连接方式，我们可以从 SQL 语句和索引设计的角度上进行改进。


### SHOW PROFILE 

SHOW PROFILE 相比 EXPLAIN 能看到更进一步的执行解析，包括 SQL 都做了什么、所花费的时间等。

```sql

show variables like 'profiling'; — 查看是否开启

set profiling = 'ON'; — 开启


select @@profiling; — 还有一种，查看是否打开

set profiling=1; — 还有一种，打开


show profiles; — 当前哪些

show profile; — 上一个

show profile for query 2; — 查看ID为2的语句

```

不过 SHOW PROFILE 命令将被弃用，我们可以从 information_schema 中的 profiling 数据表进行查看。


## 主从

主从的目的：读多写少、数据备份、高可用。

---

主从数据复制技术：

异步复制，问题就是主从切换的时候，丢数据。

5.5版本之后，半同步复制，事务提交之后，至少一个从库同步完才返回给客户端，5.7版本之后 还可以设置 rpl_semi_sync_master_wait_for_slave_count 参数确认多少台从库。

组复制，简称 MGR（MySQL Group Replication），是 MySQL 在 5.7.17 版本中推出的一种新的数据复制技术，这种复制技术是基于 Paxos 协议的状态机复制。就是 事务提交需要经过 N/2 + 1 个节点同意，保证 数据强一致性。

---

主从采用中间件的形式优势比较大。

## 其他DBMS

### Excel

进一步了解，参阅《44-SQL必知必会：35如何在Excel中使用SQL语言？》

### WebSQL

进一步了解，参阅《44-SQL必知必会：36WebSQL：如何在H5中存储一个本地数据库？》

### SQLite

SQLite内置在移动端，足够的轻量。

进一步了解，参阅《44-SQL必知必会：37SQLite：为什么微信用SQLite存储聊天记录？》

## 数据清洗

数据清洗，就是保证数据质量，比如数据重复、数据中有缺失值，或者单位不统一等。

进一步了解，参阅《44-SQL必知必会：45数据清洗：如何使用SQL对数据进行清洗？》

## 数据集成

数据集成是数据分析之前非常重要的工作，它将不同来源、不同规范以及不同质量的数据进行统一收集和整理，为后续数据分析提供统一的数据源。

进一步了解，参阅《44-SQL必知必会：46数据集成：如何对各种数据库进行集成和转换？》

## 数据分析

进一步了解，参阅《44-SQL必知必会：47如何利用SQL对零售数据进行分析？》

## 其他

### 没有 Binlog，怎么恢复数据

本质上就是拿到损坏的表，尝试恢复。

进一步了解，参阅《44-SQL必知必会：36数据库没有备份，没有使用Binlog的情况下，如何恢复数据？》

### SQL 注入

注入原理就是没有对用户输入的数据脱敏，直接拼接到SQL语句里。

进一步了解，参阅《44-SQL必知必会：37 SQL注入：你的SQL是如何被注入的？》

---

想了解 数据页，参阅《44-SQL必知必会：27从数据页的角度理解B+树查询》

想了解 数据页加载方式，参阅《44-SQL必知必会：28 从磁盘I/O的角度理解SQL查询的成本》

想了解 SQL查询成本，参阅《44-SQL必知必会：28 从磁盘I/O的角度理解SQL查询的成本》
