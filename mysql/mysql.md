

### 数据类型

| 名称         | 类型           | 说明                                                         |
| :----------- | :------------- | :----------------------------------------------------------- |
| INT          | 整型           | 4字节整数类型，范围约+/-21亿                                 |
| BIGINT       | 长整型         | 8字节整数类型，范围约+/-922亿亿                              |
| FLOAT/REAL   | 浮点型         | 4字节浮点数，范围约+/-1038                                   |
| DOUBLE       | 浮点型         | 8字节浮点数，范围约+/-10308                                  |
| DECIMAL(M,N) | 高精度小数     | 由用户指定精度的小数，例如，DECIMAL(20,10)表示一共20位，其中小数10位，通常用于财务计算 |
| CHAR(N)      | 定长字符串     | 存储指定长度的字符串，例如，CHAR(100)总是存储100个字符的字符串 |
| VARCHAR(N)   | 变长字符串     | 存储可变长度的字符串，例如，VARCHAR(100)可以存储0~100个字符的字符串 |
| BOOLEAN      | 布尔类型       | 存储True或者False                                            |
| DATE         | 日期类型       | 存储日期，例如，2018-06-22                                   |
| TIME         | 时间类型       | 存储时间，例如，12:20:59                                     |
| DATETIME     | 日期和时间类型 | 存储日期+时间，例如，2018-06-22 12:20:59                     |



### 几种数据库操作

**DDL：Data Definition Language**

DDL允许用户定义数据，也就是创建表、删除表、修改表结构这些操作。通常，DDL由数据库管理员执行。

**DML：Data Manipulation Language**

DML为用户提供添加、删除、更新数据的能力，这些是应用程序对数据库的日常操作。

**DQL：Data Query Language**

DQL允许用户查询数据，这也是通常最频繁的数据库日常操作。



### 术语

表的每一行称为**记录（Record）**，记录是一个逻辑意义上的数据。

表的每一列称为**字段（Column）**，同一个表的每一行记录都拥有相同的若干字段。

通过某个字段唯一区分出不同的记录，这个字段被称为***主键***。

关系数据库实际上还允许通过多个字段唯一标识记录，即两个或更多的字段都设置为主键，这种主键被称为**联合主键。**对于联合主键，允许一列有重复，只要不是所有主键列都重复即可



在表中，通过某个字段，可以把数据与另一张表关联起来，这种列称为**外键**。

```
ALTER TABLE students
ADD CONSTRAINT fk_class_id
FOREIGN KEY (class_id)
REFERENCES classes (id);

ALTER TABLE students
DROP FOREIGN KEY fk_class_id;
```

其中，外键约束的名称`fk_class_id`可以任意，`FOREIGN KEY (class_id)`指定了`class_id`作为外键，`REFERENCES classes (id)`指定了这个外键将关联到`classes`表的`id`列（即`classes`表的主键）。通过定义外键约束，关系数据库可以保证无法插入无效的数据。即如果`classes`表不存在`id=99`的记录，`students`表就无法插入`class_id=99`的记录。

### 常用sql

**DDL：Data Definition Language**

``` sql
CREATE DATABASE ;
CREATE DATABASE IF NOT EXISTS 数据库名;
drop database <数据库名>;
USE <数据库名>;	//选择MySQL数据库


CREATE TABLE table_name (column_name column_type (参数),
                        column_name column_type,
                        column_name column_type);

//参数中可填：AUTO_INCREMENT定义列为自增的属性，一般用于主键，数值会自动加1。
//PRIMARY KEY关键字用于定义列为主键。 您可以使用多列来定义主键，列间以逗号分隔。
//NOT NULL

DROP TABLE table_name ;

ALTER TABLE table_name  DROP i/ ADD i INT/ MODIFY c CHAR(10)/ CHANGE i j BIGINT;
ALTER TABLE table_name RENAME TO newtable_name;//修改表名

CREATE INDEX indexName ON table_name (column_name);
DROP INDEX [indexName] ON mytable; 


```



**DML：Data Manipulation Language**

``` sql
INSERT INTO table_name ( field1, field2,...fieldN )
                       VALUES
                       ( value1, value2,...valueN );
                       
UPDATE table_name SET field1=new-value1, field2=new-value2
[WHERE Clause]

DELETE FROM table_name [WHERE Clause]
```



**DQL：Data Query Language**

``` sql
SELECT column_name,column_name
FROM table_name
[WHERE Clause]			//你可以使用 WHERE 语句来包含任何条件。
[LIMIT N][ OFFSET M]	
//你可以使用 LIMIT 属性来设定返回的记录数。
//你可以通过OFFSET指定SELECT语句开始查询的数据偏移量。默认情况下偏移量为0。
//ORDER BY column_name 加上DESC表示“倒序”
//GROUP BY column_name
```





### 内置函数

| 函数  | 说明                                   |
| :---- | :------------------------------------- |
| SUM   | 计算某一列的合计值，该列必须为数值类型 |
| AVG   | 计算某一列的平均值，该列必须为数值类型 |
| MAX   | 计算某一列的最大值                     |
| MIN   | 计算某一列的最小值                     |
| COUNT | 表示查询所有列的行数                   |





### 连接查询

- **INNER JOIN（内连接,或等值连接）**：获取两个表中字段匹配关系的记录。
- **LEFT JOIN（左连接）：**获取左表所有记录，即使右表没有对应匹配的记录。
- **RIGHT JOIN（右连接）：** 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。



**INNER JOIN**

``` sql
SELECT column_name1,column_name2,column_name3 FROM table_name1 a INNER JOIN table_name2 b ON a.column_name1 = b.column_name1;
```



我们把tableA看作左表，把tableB看成右表，那么INNER JOIN是选出两张表都存在的记录：

![inner-join](mysql.assets/l.png)

LEFT OUTER JOIN是选出左表存在的记录：

![left-outer-join](mysql.assets/l-20230208103603168.png)

RIGHT OUTER JOIN是选出右表存在的记录：

![right-outer-join](mysql.assets/l-20230208103603231.png)

FULL OUTER JOIN则是选出左右表都存在的记录：

![full-outer-join](mysql.assets/l-20230208103603214.png)





### 事务

把多条语句作为一个整体进行操作的功能，被称为数据库***事务***

数据库事务具有ACID这4个特性：

- A：Atomic，原子性，将所有SQL作为原子工作单元执行，要么全部执行，要么全部不执行；
- C：Consistent，一致性，事务完成后，所有数据的状态都是一致的，即A账户只要减去了100，B账户则必定加上了100；
- I：Isolation，隔离性，如果有多个事务并发执行，每个事务作出的修改必须与其他事务隔离；
- D：Duration，持久性，即事务完成后，对数据库数据的修改被持久化存储。



使用`BEGIN`开启一个事务，使用`COMMIT`提交一个事务，这种事务被称为*显式事务*

`COMMIT`是指提交事务，即试图把事务内的所有SQL所做的修改永久保存。如果`COMMIT`语句执行失败了，整个事务也会失败。

有些时候，我们希望主动让事务失败，这时，可以用`ROLLBACK`回滚事务，整个事务会失败：

``` sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;/ ROLLBACK;
```



### 隔离级别

对于两个并发执行的事务，如果涉及到操作同一条记录的时候，可能会发生问题。因为并发操作会带来数据的不一致性，包括脏读、不可重复读、幻读等。数据库系统提供了隔离级别来让我们有针对性地选择事务的隔离级别，避免数据不一致的问题。

SQL标准定义了4种隔离级别，分别对应可能出现的数据不一致的情况：

| Isolation Level  | 脏读（Dirty Read） | 不可重复读（Non Repeatable Read） | 幻读（Phantom Read） |
| :--------------- | :----------------- | :-------------------------------- | :------------------- |
| Read Uncommitted | Yes                | Yes                               | Yes                  |
| Read Committed   | -                  | Yes                               | Yes                  |
| Repeatable Read  | -                  | -                                 | Yes                  |
| Serializable     | -                  | -                                 | -                    |

```sql
SET TRANSACTION ISOLATION LEVEL IsolationLevel;
```

默认的隔离级别是Repeatable Read。



一个事务会读到另一个事务更新后但未提交的数据，如果另一个事务回滚，那么当前事务读到的数据就是脏数据，这就是脏读（Dirty Read）。

不可重复读是指，在一个事务内，多次读同一数据，在这个事务还没有结束时，如果另一个事务恰好修改了这个数据，那么，在第一个事务中，两次读取的数据就可能不一致。

幻读是指，在一个事务中，第一次查询某条记录，发现没有，但是，当试图更新这条不存在的记录时，竟然能成功，并且，再次读取同一条记录，它就神奇地出现了。