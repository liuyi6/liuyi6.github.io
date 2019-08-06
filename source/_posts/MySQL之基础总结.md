------

title: MySQL之基础总结
date: 2019-01-26 20:58:19
tags: [MySQL]
categories: 
- MySQL

------
## 一、MySQL说明

### 1、数据库概述

数据库就是**存储数据的仓库**，其本质是一个**文件系统**，数据按照特定的格式将数据存储起来，用户可以通过SQL对数据库中的数据进行增加、修改、删除及查询操作。

数据库中的记录是**有行有列的数据库**称为关系型数据库，与之相反的就是NoSQL（Not Only SQL）数据库了。

数据库管理系统（DataBase Management System，DBMS）：指一种[操作和管理数据库]的大型软件，用于建
立、使用和维护数据库，对数据库进行统一管理和控制，以保证数据库的安全性和完整性。用户通过数据库管理系统访问数据库中表内的数据(记录)。

常见的数据库管理系统有：MySQL、Oracle、SQLServer等

### 2、MySQL简介

MySQL 是当前最流行的关系型数据库管理系统，在WEB应用方面 MySQL1是最好的RDBMS应用软件之一。

MySQL发展历程：

* 1996年，MySQL 1.0发布
* 1996年10月，MySQL 3.11.1发布(MySQL没有2.x版本)，最开始只提供Solaris下的二进制版本。一个月后，Linux版本出现。

* 1999～2000年，MySQL AB公司在瑞典成立。期间开发出了Berkeley DB引擎, 由于BDB支持事务处理，MySQL开始支持事务处理。
* 2000年，MySQL不仅公布自己的源代码，并采用GPL(GNU General Public License)许可协议，正式进入
  开源世界。同年4月，MySQL对旧的存储引擎ISAM进行了整理，将其命名为MyISAM。
* 2001年，MySQL集成存储引擎InnoDB，这个引擎不仅能支持事务处理，并且支持行级锁。后来该引擎被证明是最为成功的MySQL事务存储引擎，MySQL与InnoDB的正式结合版本是4.0。
* 2008年1月，MySQL AB公司被**Sun公司收购**，MySQL数据库进入Sun时代。
* 2009年4月，Oracle公司收购Sun公司，自此MySQL数据库进入Oracle时代，而其存储引擎InnoDB在2005年就被Oracle公司收购。
* 2010年12月，**MySQL 5.5发布**，其主要新特性包括半同步的复制及对SIGNAL/RESIGNAL的异常处理功能的支持，最重要的是**InnoDB存储引擎成为MySQL的默认存储引擎**。

### 3、SQL简介

【SQL是Structured Query Language的缩写】，是一种**访问关系型数据库的标准语言**，它的前身是著名的关系数据库原型系统System R所采用的SEQUEL语言。一般认为SQL92是是SQL的国际标准。MySQL等数据库都是在SQL92或者SQL99这些国际SQL标准基础之上扩展出自己的SQL语句，如MySQL中的limit关键字。

SQL分为如下几类：

- **数据定义语言**，简称【DDL】(Data Definition Language)，用来定义数据库对象：数据库，表，列等。
  关键字：create，alter，drop等

- **数据操作语言**，简称【DML】(Data Manipulation Language)，用来对数据库中表的记录进行更新。关键
  字：insert，delete，update等
- **数据控制语言**，简称【DCL】(Data Control Language)，用来定义数据库的访问权限和安全级别，及创建
  用户；关键字：grant等
- **数据查询语言**，简称【DQL】(Data Query Language)，用来查询数据库中表的记录。关键字：select，
  from，where等

## 二、SQL语句

### 1、DDL语句

#### 1.1、数据库操作

**创建数据库**

```mysql
create database 数据库名;
create database 数据库名 character set 字符集;
```

**查看数据库**

查看数据库服务器中的所有的数据库:

```mysql
show databases;
```

查看某个数据库的定义的信息:

```mysql
show create database 数据库名;
```

**删除数据库**

```mysql
drop database 数据库名称;
```

**其他操作**

切换数据库：

```mysql
use 数据库名;
```

查看正在使用的数据库:

```mysql
select database();
```

#### 1.2、表操作

**字段类型**

在进行表操作过程中涉及到表中的字段，下表列出了MySQL中的字段类型：

|     分类     |   类型名称   | 说明                           |
| :----------: | :----------: | ------------------------------ |
|     整数     |   tinyInt    | 很小的整数                     |
|     整数     |   smallint   | 小的整数                       |
|     整数     |  mediumint   | 中等大小的整数                 |
|     整数     | int(integer) | 普通大小的整数                 |
|     小数     |    float     | 单精度浮点数                   |
|     小数     |    double    | 双精度浮点数                   |
|     小数     | decimal(m,d) | 压缩严格的定点数               |
|     日期     |     year     | 1901~2155                      |
|     日期     |     time     | -838：59：59~838：59：59       |
|     日期     |     date     | 1000-01-01~9999-12-03          |
|     日期     |   datetime   | YYYY-MM-DD HH:MM:SS            |
|     日期     |  timestamp   | eg：UTC-2038-01-19 03:14:07UTC |
| 文本、二进制 |   char(m)    | m为0~255之间整数               |
| 文本、二进制 |  varchar(m)  | m为0~65535之间整数             |
| 文本、二进制 |   tinyblob   | 长度0~255字节                  |
| 文本、二进制 |     blob     | 长度0~65535字节                |
| 文本、二进制 |  mediumblob  | 长度0~167772150字节            |
| 文本、二进制 |   longblob   | 长度0~4294967295字节           |
| 文本、二进制 |   tinytext   | 长度0~255字节                  |

**创建表**

```mysql
create table 表名(
	字段名 类型(长度) 约束,
	字段名 类型(长度) 约束
);
```

此处涉及到单表约束，包括

* 主键约束：primary key

- 唯一约束：unique
- 非空约束：not null

其中，主键约束 = 唯一约束 + 非空约束

**查看表**

查看数据库中的所有表：

```mysql
show tables;
```

查看表结构：

```mysql
desc 表名;
```

**删除表**

```mysql
drop table 表名;
```

**修改表**

```mysql
# 修改表添加列
alter table 表名 add 列名 类型(长度) 约束;
# 修改表修改列的类型长度及约束
alter table 表名 modify 列名 类型(长度) 约束;
# 修改表修改列名
alter table 表名 change 旧列名 新列名 类型(长度) 约束;
# 修改表删除列
alter table 表名 drop 列名;
# 修改表名
rename table 表名 to 新表名;
# 修改表的字符集
alter table 表名 character set 字符集;
```

### 2、DML语句

#### 2.1、插入记录

```mysql
# 向表中插入某些列
insert into 表 (列名1,列名2,列名3..) values (值1,值2,值3..);
# 向表中插入所有列
insert into 表 values (值1,值2,值3..);
# 向表中插入查出数据
insert into 表 (列名1,列名2,列名3..) values select (列名1,列名2,列名3..) from 表
insert into 表 values select * from 表
```

注意：

1. 列名数与values后面的值的个数相等
2. 列的顺序与插入的值得顺序一致
3. 列名的类型与插入的值要一致
4. 插入值得时候不能超过最大长度
5. 值如果是字符串或者日期需要加引号（一般是单引号）

#### 2.2、更新记录

```mysql
update 表名 set 字段名=值,字段名=值;
update 表名 set 字段名=值,字段名=值 where 条件;
```

注意：
1. 列名的类型与修改的值要一致
2. 修改值得时候不能超过最大长度
3. 值如果是字符串或者日期需要加引号

#### 2.3、删除记录

```mysql
delete from 表名 [where 条件];
```

区分【delete from 表名】与【truncate table 表名】:

- delete ：一条一条删除，不清空auto_increment记录数。
- truncate ：直接将表删除，重新建表，auto_increment将置为零，从新开始。

### 3、DQL语句

现有一数据表product，其创建语句如下：

```mysql
CREATE TABLE product (
	pid INT PRIMARY KEY AUTO_INCREMENT, #自增加AUTO_INCREMENT
	pname VARCHAR(20),#商品名称
	price DOUBLE, #商品价格
	pdate DATE, # 日期
	sid VARCHAR(20) #分类ID
);
```

#### 3.1、简单查询

简单查询关键字：`select`、`from`

查询所有的商品：

```mysql
select * from product;
```

查询商品名和商品价格：

```mysql
select pname,price from product;
```

别名查询，使用的as关键字，as可以省略的

表别名：

```mysql
select * 1 from product as p;
```

列别名：

```mysql
select pname as pn from product;
```

去掉重复值：

```mysql
select distinct price from product;
```

查询结果是表达式（运算查询）：

将所有商品的价格+10元进行显示

```mysql
select pname,price+10 from product;
```

#### 3.2、条件查询

条件查询关键字：`where`

where之后条件语句的写法如下句所示：

|   运算符   |        类型         | 说明                                                       |
| :--------: | :-----------------: | ---------------------------------------------------------- |
| 比较运算符 | <、>、<=、>=、=、<> | d大于、小于、大于(小于)等于、不等于                        |
| 比较运算符 | between ... and ... | 在某一区间，in(1,100)                                      |
| 比较运算符 |     like '_an%'     | 模糊查询，like语句中，%代表零或多个任一字符，_代表一个字符 |
| 比较运算符 |       is null       | 判断是否为空                                               |
| 逻辑运算符 |         and         | 多条件同时成立                                             |
| 逻辑运算符 |         or          | 多条件任一成立                                             |
| 逻辑运算符 |         not         | 不成立                                                     |

如查询价格大于60的产品：

```mysql
select * from product where price > 60;
```

查询id在区间中：

```mysql
select * from product where pid in (2,5,8);
```

#### 3.3、排序

排序关键字：`order by`，其中desc是降序，asc是升序。

查询所有的商品，按价格降序：

```mysql
select * from product 1 order by price desc;
```

#### 3.4、聚合函数

聚合函数又称组函数，通常只对单列操作

常用的聚合函数：

```mysql
sum()：求某一列的和
avg()：求某一列的平均值
max()：求某一列的最大值
min()：求某一列的最小值
count()：求某一列的元素个数
```

如查询所有商品的价格的总和：

```mysql
select sum(price) from product;
```

#### 3.5、分组

分组关键字：`GROUP BY`、`HAVING`

如根据cid分组，分组统计每组商品的平均价格，并且平均价格> 60

```mysql
select cid,avg(price) from product group by cid having avg(price)>60;
```

注意：

* select语句中的列（非聚合函数列），必须出现在group by子句中
* group by子句中的列，不一定要出现在select语句中
* 聚合函数只能出现select语句中或者having语句中，一定不能出现在where语句中。

#### 其他

`union`集合的并集（不包含重复记录），`unionall`集合的并集（包含重复记录）

## 三、多表关系

### 1、表与表间关系

表与表之间的关系，说的是**表与表之间数据的关系**。

- 一对一关系，如学生与学号
- 一对多关系，如用户和订单
- 多对多关系，多对多需借助中间表实现，如商品和订单

### 2、外键

表与表之间的关系是通过外键约束进行实现的，通过外键关联起来的两张表分为主表和从表，其中有外键的表为主表，通过外键关联的表为从表。

**主键外键**即将从表中的主键作为主表中的外键。

使用外键的目的：

- 保证数据完整性（数据保存在多张表中的时候）
- 在互联网项目中，一般情况下，不建议建立外键关系。

主表添加外键：

```mysql
alter table 表名 add [constraint][约束名称] foreign key (主表外键字段) references 从表(从
表主键)
```

主表删除外键：

```mysql
alter table 表名 drop foreign key 外键约束名称
```

### 3、一对一关系

一对一在开发中应用不多，它们可以创建在一张表中。

一对一关系有两种创建方式：

1. 外键唯一，将从表中的某一列加唯一约束，并将此列设为主表的外键

   - 给外键列wid指定唯一约束（不加唯一约束就是一对多关系）

     ```mysql
     ALTER TABLE husband ADD wid INT UNIQUE;
     ```

   - 添加外键约束

     ```mysql
     ALTER TABLE husband ADD FOREIGN KEY (wid) REFERENCES wife (id);
     ```

2. 主键作外键，将从表的主键设为主表的外键

   - 将已有的字段设置为自增主键

     ```mysql
     ALTER TABLE user MODIFY id INT PRIMARY KEY AUTO_INCREMENT;
     ```

   - 将主键设为主表外键

     ```mysql
     ALTER TABLE score ADD FOREIGN KEY (u_id) REFERENCES USER (id);
     ```

### 4、一对多关系

一般**有外键的为多的一方**。

一对多关系和一对一关系的创建很类似，唯一区别就是外键不唯一。

一对多关系创建包括两步：添加无唯一约束的外键列、添加外键约束

### 5、多对多关系

多对多关系其实就是两个一对多关系的组合，需要中间表去完成多对多关系的创建。

多对多关系的创建：

- 创建中间表，并在其中创建多对多关系中两张表的外键列
- 在中间表中添加外键约束
- 在中间表中添加联合主键约束

## 四、多表关联查询

MySQL 中可在 SELECT , UPDATE 和 DELETE 语句中使用 JOIN关键字在两个或多个表中查询数据。

JOIN 按照功能大致分为如下三类：

- CROSS JOIN （交叉连接）
- INNER JOIN （内连接或等值连接）
- OUTER JOIN （外连接）

### 1、交叉连接

交叉连接关键字：`CROSS JOIN`

**交叉连接也叫笛卡尔积连接**。笛卡尔积是指在数学中，两个集合X 和Y 的笛卡尓积（ Cartesian product ），
又称直积，表示为X*Y ，第一个对象是X 的成员而第二个对象是Y 的所有可能有序对的其中一个成员。

交叉连接表现为：**行数相乘、列数相加**，它分为：

- 隐式交叉连接

  ```mysql
  SELECT * FROM A, B;
  ```

- 显式交叉连接

  ```mysql
  SELECT * FROM A CROSS JOIN B;
  ```

### 2、内连接

内连接关键字：`INNER JOIN`

内连接也叫**等值连接**，内联接使用比较运算符根据每个表共有的列的值匹配两个表中的行。

- 隐式内连接

  ```mysql
  SELECT * FROM A,B WHERE A.id = B.id
  ```

- 显式内连接

  ```mysql
  SELECT * FROM A INNER JOIN B ON A.id = B.id
  ```

### 3、外连接

外联接分为左向外联接、右向外联接或完整外部联接。

外连接需要有主表或者保留表的概念。

在 FROM 子句中指定外联接时，可以由下列几组关键字中的一组指定：

- 左外连接：`LEFT JOIN 1 或 LEFT OUTER JOIN`

  ```mysql
  SELECT * FROM A LEFT JOIN B ON A.id = B.id
  ```

- 右外连接：`RIGHT JOIN或RIGHT OUTER JOIN`

  ```mysql
  SELECT * FROM A RIGHT JOIN B ON A.id = B.id
  ```

- 全外连接(MySQL不支持)

  ```mysql
  SELECT * FROM A FULL JOIN B ON A.id = B.id
  ```

  外连接总结：

- 通过业务需求，分析主从表
- 如果使用LEFT JOIN ，则主表在它左边
- 如果使用RIGHT JOIN ，则主表在它右边
- 查询结果以主表为主，从表记录匹配不到，则补null

### 4、分页查询

分页查询关键字：`lIMIT`

lIMIT 关键字不是SQL92 标准提出的关键字，它是**MySQL 独有的语法**。通过limit 关键字， MySQL 实现了物理分页。

分页分为逻辑分页和物理分页：

- 物理分页，通过LIMIT关键字，直接在数据库中进行分页，最终返回的数据，只是分页后的数据。
- 逻辑分页，将数据库中的数据查询到内存之后再进行分页。

SQL语句格式：

```mysql
SELECT * FROM table LIMIT 1 [offset,] rows
```

其中：offset 为编译量；rows 为每页多少行记录。

### 5、子查询

**子查询允许把一个查询嵌套在另一个查询当中**。子查询，又叫内部查询，相对于内部查询，包含内部查询的就称为外部查询。
对应的外部查询是`select、insert、update、delete`中之一时，子查询可以包含普通select可以包括的任何子句，如：`distinct、 group by、order by、limit、join、union`等；

子查询可以放在select中、from 后、where 中，但当其放在group by 和order by 中时无实用意义。

## 五、SQL解析顺序

现有一SQL语句，如下所示：

```mysql
SELECT DISTINCT
	< select_list >
FROM
	< left_table > < join_type >
JOIN < right_table > ON < join_condition >
WHERE
	< where_condition >
GROUP BY
	< group_by_list >
HAVING
	< having_condition >
ORDER BY
	< order_by_condition >
LIMIT < limit_number >
```

先给出结论，SQL语句的执行顺序如下所示：

```mysql
FROM <left_table>
ON <join_condition>
<join_type> JOIN <right_table> #第二步和第三步会循环执行
WHERE <where_condition>  	   #第四步会循环执行，多个条件的执行顺序是从左往右的。
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT 						   #分组之后才会执行SELECT
DISTINCT <select_list>
ORDER BY <order_by_condition>  #前9步都是SQL92标准语法。
LIMIT <limit_number>           #limit是MySQL的独有语法。
```

下面对此执行过程进行分析：

在进行分析之前，创建两张表并插入数据，进行过程分析：

```mysql
CREATE TABLE table1
(
uid VARCHAR(10) NOT NULL,
name VARCHAR(10) NOT NULL,
PRIMARY KEY(uid)
)ENGINE=INNODB DEFAULT CHARSET=UTF8;

CREATE TABLE table2
(
oid INT NOT NULL auto_increment,
uid VARCHAR(10),
PRIMARY KEY(oid)
)ENGINE=INNODB DEFAULT CHARSET=UTF8;

INSERT INTO table1(uid,name) VALUES('aaa','mike'),('bbb','jack'),('ccc','mike'),
('ddd','mike');
INSERT INTO table2(uid) VALUES('aaa'),('aaa'),('bbb'),('bbb'),('bbb'),('ccc'),(NULL);
```

进行查询的语句为：

```mysql
SELECT
	a.uid,
	count(b.oid) AS total
FROM
	table1 AS a
LEFT JOIN table2 AS b ON a.uid = b.uid
WHERE
	a. NAME = 'mike'
GROUP BY
	a.uid
HAVING
	count(b.oid) < 2
ORDER BY
	total DESC
LIMIT 1;
```

## 二、解析步骤分析

如上的SQL语句解析共分为十步：

### 1、FROM语句

对FROM的左边的表和右边的表计算笛卡尔积(CROSS JOIN)产生虚表VT1 。注：笛卡尔积为**行相乘、列相加**。

```mysql
select * from table1,table2;
#运行结果

```

### 2、ON过滤

对虚表VT1 进行ON筛选，只有那些符合的行才会被记录在虚表VT2中。

**注意**：这里因为语法限制，使用了`WHERE`代替，从中读者也可以感受到两者之间微妙的关系；

```mysql
SELECT
	*
FROM
	table1,
	table2
WHERE
	table1.uid = table2.uid;
	
#运行结果

```

### 3、OUTER JOIN

如果指定了OUTER JOIN（如left join、 right join） ，那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2 中，产生虚拟表VT3 。
如果FROM子句中包含两个以上的表的话，那么就会对上一个join连接产生的结果VT3和下一个表重复执行步骤1~3
这三个步骤，一直到处理完所有的表为止。

```mysql
SELECT
	*
FROM
	table1 AS a
LEFT OUTER JOIN table2 AS b ON a.uid = b.uid;
#运行结果

```

### 4、WHERE过滤

对虚拟表VT3 进行WHERE条件过滤，只有符合的记录才会被插入到虚拟表VT4 中。

注意：此时因为分组，不能使用聚合运算；也不能使用SELECT中创建的别名；

where与ON的区别：

- 如果有外部列，ON针对过滤的是关联表，主表（保留表）会返回所有的列；
- 如果没有添加外部列，两者的效果是一样的；

此外，MySQL 是从左往右去执行WHERE 条件的；Oracle 是从右往左去执行WHERE 条件的。因而写WHERE条件的时候，优先级高的部分要去编写过滤力度最大的条件语句。

在有WHERE与ON的语句中需要注意的是：

- 对主表的过滤应该放在WHERE；
- 对于关联表，先条件查询后连接则用ON，先连接后条件查询则用WHERE；

```mysql
SELECT
	*
FROM
	table1 AS a
LEFT OUTER JOIN table2 AS b ON a.uid = b.uid
WHERE
	a. NAME = 'mike';
#运行结果

```

### 5、GROUP BY

根据group by子句中的列，对VT4中的记录进行分组操作，产生虚拟表VT5 。

**注意**：

其后处理过程的语句（如SELECT、HAVING）所用到的列必须包含在GROUP BY中。未出现的，需用聚合函数；

这是由于GROUP BY改变了对表的引用，将其转换为新的引用方式，能够对其进行下一级逻辑操作的列会减少；
我的理解是：
根据分组字段，将具有相同分组字段的记录归并成一条记录，因为每一个分组只能返回一条记录，除非是被过滤掉
了，而不在分组字段里面的字段可能会有多个值，多个值是无法放进一条记录的，所以必须通过聚合函数将这些具
有多值的列转换成单值；

```mysql
SELECT
	*
FROM
	table1 AS a
LEFT OUTER JOIN table2 AS b ON a.uid = b.uid
WHERE
	a. NAME = 'mike'
GROUP BY
	a.uid;
#运行结果

```

### 6、HAVING

对虚拟表VT5 应用having过滤，只有符合的记录才会被 插入到虚拟表VT6 中。

```mysql
SELECT
	*
FROM
	table1 AS a
LEFT OUTER JOIN table2 AS b ON a.uid = b.uid
WHERE
	a. NAME = 'mike'
GROUP BY
	a.uid
HAVING
	count(b.oid) < 2;
#运行结果

```

### 7、SELECT

对SELECT子句中的元素进行处理，计算SELECT 子句中的表达式，生成VT6-J1表

### 8、DISTING

寻找VT6-J1中的重复列，并删掉，生成VT6-J2表。

如果在查询中指定了DISTINCT子句，则会创建一张内存临时表（如果内存放不下，就需要存放在硬盘了）。这张
临时表的表结构和上一步产生的虚拟表VT5是一样的，不同的是对进行DISTINCT操作的列增加了一个唯一索引，以此来除重复数据。

### 9、ORDER BY

从VT6-J2 中的表中，根据ORDER BY 子句的条件对结果进行排序，生成VT7表。
注意：
唯一可使用SELECT中别名的地方；

```mysql
SELECT
	*
FROM
	table1 AS a
LEFT OUTER JOIN table2 AS b ON a.uid = b.uid
WHERE
	a. NAME = 'mike'
GROUP BY
	a.uid
HAVING
	count(b.oid) < 2
ORDER BY
	total DESC;
#运行结果

```

### 10、LIMIT

LIMIT子句从上一步得到的VT6虚拟表中选出从指定位置开始的指定行数据。
注意：
offset 和rows 的正负带来的影响；
**当偏移量很大时效率是很低的**，可以采用如下做法：

- 采用子查询的方式优化，在子查询里先从索引获取到最大id，然后倒序排，再取N行结果集
- 采用INNER JOIN优化，JOIN子句里也优先从索引获取ID列表，然后直接关联查询获得最终结果

```mysql
SELECT
	*
FROM
	table1 AS a
LEFT OUTER JOIN table2 AS b ON a.uid = b.uid
WHERE
	a. NAME = 'mike'
GROUP BY
	a.uid
HAVING
	count(b.oid) < 2
ORDER BY
	total DESC
LIMIT 1;
#运行结果

```

## 三、解析顺序总结

如图所示：

![1551167760992](E:/new%20blog/source/_posts/images/2018-02/1551167760992.png)

说明：

```mysql
1. FROM（将最近的两张表，进行笛卡尔积）---VT1
2. ON（将VT1按照它的条件进行过滤）---VT2
3. LEFT JOIN（保留左表的记录）---VT3
4. WHERE（过滤VT3中的记录）--VT4…VTn
5. GROUP BY（对VT4的记录进行分组）---VT5
6. HAVING（对VT5中的记录进行过滤）---VT6
7. SELECT（对VT6中的记录，选取指定的列）--VT6-J1
8. ORDER BY（对VT7的记录进行排序）--VT6-J2
9. LIMIT（对排序之后的值进行分页）--MySQL特有的语法VT7
```

此处不同的表个数有不同的处理：

- **单表查询**：根据WHERE 条件过滤表中的记录，形成中间表（这个中间表对用户是不可见的）；然后根据SELECT 的选择列选择相应的列进行返回最终结果。
- **两表连接查询**：对两表求积（笛卡尔积）并用ON 条件和连接连接类型进行过滤形成中间表；然后根据WHERE条件过滤中间表的记录，并根据SELECT 指定的列返回查询结果。
- **多表连接查询**：先对第一个和第二个表按照两表连接做查询，然后用查询结果和第三个表做连接查询，以此类推，直到所有的表都连接上为止，最终形成一个中间的结果表，然后根据WHERE条件过滤中间表的记录，并根据SELECT指定的列返回查询结果。