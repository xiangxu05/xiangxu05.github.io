---
layout: post
title: Mysql｜数据库基本命令及相关知识
categories: [DATABASE]
description: 记录Mysql的基本命令以及一些相关的知识。
keywords: Mysql, DATABASE
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false 
---



这篇文章用来记录Mysql的基本命令，以及后续遇到的一些知识和问题。主要内容来源于：[MySQL入门技能树](https://edu.csdn.net/skill/mysql)，以及[MySQL 教程 | 菜鸟教程](https://www.runoob.com/mysql/mysql-tutorial.html)，感谢！

## 0x00 基本命令

### 数据库操作

- 登陆

```cmd
mysql -h localhost -P 3306 -u root -ppassword
```

localhost可以改成可以连接到的任意数据库，-P后面是端口，具体看数据库开放了哪个端口，默认是3306。

- 创建数据库

```mysql
create database database_name; #创建一个数据库
create database if not exists database_name; #创建一个数据库如果它不存在
```

- 显示数据库

```mysql
show database; #显示当前存在那些数据库
select database(); #查看当前命令行所在的数据库
```

- 切换数据库

```mysql
use database_name;
```

- 查看数据库创建信息

```mysql
show create database database_name;
show create database database_name \G
```

- 更改数据库名称

```mysql
rename database db_name to new_db_name #已经去掉了，由于此sql语句回造成数据丢失

#现行方法1
rename table test_old.table_test to test_new.table_test; #使用重命名的方式迁移表格
drop table if exists test_old; #删除数据库

#现行方法2
mysqldump -u root -p test_old > test_old_dump.sql; #导出数据库
use test_new;
source /path/to/sql/file/test_old_dump.sql; #导入到新的数据库
drop table if exists test_old;

#现行方法3
create table if not exists test_new.table_test like test_old.table_test;#通过复制表的方式
```

- 数据库编码

在MySQL中，会为创建的每个数据库指定一个字符编码。如果在创建数据库时没有为数据库指定字符编码，则MySQL会为数据库指定一个默认的字符编码，这个默认的字符编码在MySQL的配置文件my.cnf中进行配置。

```mysql
[client]
  default-character-set = utf8mb4
  [mysqld]
  character_set_server = utf8mb4
  [mysql]
  default-character-set=utf8mb4
```

或者在创建数据库时指定

```mysql
create database [if not exists] database_name default character set character_name collate collate_name [default encryption='N'];
#character 字符编码 eg.UTF-8
#collate 校验规则 eg.utf8_unicode_ci

show variables like '%character_set_database%'; #查看数据库字符编码
alter database database_name character set character_name collate collate_name;
#修改数据库字符编码
```

- 删除数据库

```
drop database [if exists] database_name;
```

### 数据表操作

- 创建数据表

```mysql
create table [if not exists] table_name(
	字段1,数据类型 [约束条件] [默认值],
	……
	[表约束条件]
	);
```

在创建数据表时，必须指定数据表的表名称，表名称在Windows操作系统上不区分大小写，在Linux操作系统上区分大小写。如果需要在Linux操作系统上不区分大小写，则需要在MySQL的配置文件my.cnf中添加一项配置。

------

```mysql
lower_case_table_names=1
```

==创建时指定主键==

```mysql
#单列主键
字段 数据类型 primary key [默认值] #在定义列时指定主键
[constraint 约束条件名] primary key [字段名] #在定义完所有列后指定主键

#多列联合主键
primary ky [字段1，字段2，字段3，……，字段n]
```

==也可以指定外键，从而来关联两张表格。==

指定的外键必须是另一张表的主键。一张表的外键可以不是本表的主键，但其对应着另一张表的主键。在一张表中定义了外键之后不允许删除另一张表中具有关联关系的行数据。

由外键引申出两个概念，分别是主表（父表）和从表（子表）。

主表（父表）：两个表具有关联关系时，关联字段中主键所在的表为主表（父表）。

从表（子表）：两个表具有关联关系时，关联字段中外键所在的表为从表（子表）。

```mysql
[constraint 外键名] foreign key 字段1 [,字段2，字段3，……]
	references 主表名 主键列1 [,主键列2，主键列3，……]
#外键名：定义外键时为数据表指定的外键名称。在同一张数据表中，外键的名称必须唯一。也就是说，在同一张数据表中，不能有相同名称的外键名称。
#FOREIGN KEY：指定外键包含哪些字段，可以是一个字段，也可以是多个字段的组合。
#REFERENCES：指定关联的主表名称。
#主表名：主键所在的表名称。
#主键列：主表中定义的主键字段，可以是一个字段，也可以是多个字段的组合。
```

==其他限定==

```mysql
字段名称 数据类型 not null #创建数据表时指定字段非空
字段名称 数据类型 default 默认值 #创建数据表时指定默认值
字段名称 数据类型 auto_increment #将整数类型的主键设置为默认递增类型

create table [if not exists] table_name()ENGINE=存储引擎名称; #创建数据表时指定存储引擎
default character set 编码 collate 校对规则 #创建数据表时指定编码
default charset=编码 collate=校对规则
```

- 查看数据表结构

```mysql
describe 表名称；
desc 表名称；

show create table 表名 \G #\G可以使输出结果信息更加美观，便于查看和阅读。
```

- 修改数据表

```mysql
alter table 原表名 rename [to] 新表明
```

- 增加字段

```mysql
alter table 表名 add column 新字段名 数据类型 [not null default 默认值]

#指定插入位置
alter table 表名 add column 新字段名 数据类型 [not null default 默认值] first;
alter table 表名 add column 新字段名 数据类型 [not null default 默认值] after 原有字段名;
```

- 修改字段名称

```mysql
alter table 表名 change 原有字段名 新字段名 新数据类型 #可以不改数据类型，但是数据类型不能为空
```

- 修改字段数据类型

```mysql
alter table 表名 modify 字段名 新数据类型 [default 默认值];
```

- 修改字段位置

```mysql
alter table 表名 modify 字段名 数据类型 first;
alter table 表名 modify 字段名 数据类型 after 字段名2;
```

- 删除字段

```mysql
alter table 表名 drop 字段名
```

- 修改表引擎

```mysql
alter table 表名 engine=存储引擎名称
```

- 取消数据表的外键约束

```mysql
alter table 表名 drop foreign key 外键名
```

- 删除数据表

```mysql
drop table [if exists] 数据表1 [,数据表2，……，数据表n]；#没有关联关系

#有关联关系
alter table 表名 drop foreign key 外键名
drop table [if exists] 数据表1 [,数据表2，……，数据表n]；
```

### 数据修改

- 数据插入

规则如下：

​	当需要向数据表中插入完整的行记录，并且没有指定需要插入数据的列或字段时，MySQL默认会向所有列或字段中插入数据，这就要求插入数据的值列表必须对应数据表中所有的列或字段。

​	当需要向数据表中插入完整的行记录，并且需要指定插入数据的列或字段时，必须指定所有的列或字段插入数据（整数类型并且标识为AUTO_INCREMENT的自增主键列或字段除外）。

​	当未指定插入数据的列或字段时，插入数据的值列表必须对应数据表中的所有列或字段，并且插入值列表的顺序必须和数据表中定义字段的顺序保持一致。

​	当指定了插入数据的列或字段时，插入数据的值列表必须对应列或字段列表，不需要按照表定义的列或字段的顺序插入，只需要保证插入的值列表的顺序与列或字段的列表顺序一致即可。

​	当指定了插入数据的列或字段时，可以向数据表中插入完整的行记录，也可以只向部分字段中插入数据。

```mysql
#插入完整的行记录
insert into table_name values (value1 [,value2,……,valuen])

insert into table_name #指定列或字段，指定需要插入数据的字段或列时，不需要按照表定义字段的顺序插入，
(column1 [, column2,……,columnn]) #只需要保证值列表的顺序与字段列表的顺序一致即可。
values
(value1 [,value2,……,valuen])

#指定字段插入数据
insert into table_name 
(column1 [, column2,……,columnn])
values
(value1 [,value2,……,valuen])

#一次插入多条数据
insert into table_name
values
(value1 [,value2,……,valuen]),
(value1 [,value2,……,valuen]),
……
(value1 [,value2,……,valuen]);

insert into table_name
(column1 [, column2,……,columnn])
values
(value1 [,value2,……,valuen]),
(value1 [,value2,……,valuen]),
……
(value1 [,value2,……,valuen]);

#将查询结果插入另一个表中
insert into target_table #插入数据的目标表
(tar_column1 [, tar_column2 , …… , tar_columnn]) #目标表字段列表
select
(src_column1 [, src_column2, …… , src_columnn]) #数据来源字段列表
from source_table #数据来源表
[where condition] #限制条件

create table taget_table like source_table; #使用CREATE…LIKE…语句创建
```

- 数据更新

```mysql
update table_name
set volumn1=value1, column2=value2, …… , columnn=valuen
[where condition] #更新所有数据只需要去掉条件这条语句就行了

#MySQL支持更新某个范围内的数据，可以通过BETWEEN…AND语句或者“>”“>=”“<”“<=”“<>”“!=”等运算符，或者LIKE、IN、NOT　IN等语句实现。

update t_goods set
t_upper_time = now()
where t_name regexp '裙$' #匹配正则表达式
```

- 数据删除

```mysql
delete from table_name [where condition]

#condition可以使用=，BETWEEN…AND语句或者“>”“>=”“<”“<=”“<>”“!=”等运算符，或者LIKE、IN、NOT　IN等语句实现，同样也可以用正则表达式regexp。

delete from table_name; #删除所有数据。
```

### 数据查询

```mysql
select column1, column2, …… #选择的列的名称，如果使用 * 表示选择所有列
from table_name #查询数据的表的名称 
[where condition] #指定过滤条件
[order by column_name [asc | desc]] #指定结果集的排序顺序，默认是升序（ASC）
[limit number]; #限制返回的行数

#condition可以使用=，BETWEEN…AND语句或者“>”“>=”“<”“<=”“<>”“!=”等运算符，或者LIKE、IN、NOT　IN等语句实现，同样也可以用正则表达式regexp。
```

## 0x01 相关知识

### Mysql体系结构

![体系结构图](/images/posts/2024-05-28-Mysql-commond.assets/3c7cec536a604495ae97cb54ffd7e6a2.png)

#### InnoDB 存储引擎（默认）

(1) 支持自动增长列AUTO_INCREMENT。自动增长列的值不能为空，且值必须唯一。MySQL中规定自增列必须为主键。

(2) 支持外键，保证数据的完整性和正确性。外键所在表为子表，外键所依赖的表为父表。父表中被子表外键关联的字段必须为主键。

(3) DML操作遵循ACID模型，支持事务。

(4) 行级锁 ，提高并发访问性能。

(5)表空间文件：xxx.ibd：xxx代表的是表名，innoDB引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm、sdi(8.0之后)）、数据和索引。

**优点**
提供良好的事务管理、崩溃修复能力和并发控制
**缺点**
读写效率稍差，占用的数据空间相对较大

#### MyISAM 存储引擎

(1) 不支持事务，不支持外键

(2) 支持表锁，不支持行锁

(3) 占用空间小，访问速度快

MyISAM存储引擎的文件类型：xxx.sdi：存储表结构信息 (8.0以后)；xxx.MYD: 存储数据；xxx.MYI: 存储索引

#### Memory 存储引擎

(1) 内存存放
(2) hash索引（默认）

xxx.sdi：存储表结构信息

![引擎比较](/images/posts/2024-05-28-Mysql-commond.assets/30d7e7b4350b44448627993e73505bb6.png)

### Mysql数据类型

#### 整型

MySQL中的整数类型包括TINYINT、SMALLINT、MEDIUMINT、INT(INTEGER)和BIGINT。不同的整数类型，其所需要的存储空间和数值范围不尽相同。

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_11,shadow_100,t_100,g_se,order_0,align_2,interval_4.jpeg)

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_20,shadow_100,t_100,g_se,order_0,align_2,interval_4.jpeg)

`CREATE TABLE t2(id INT(6));`指定的是显示宽度，只要不溢出都不会报错，`CREATE TABLE t3(id1 INT ZEROFILL, id2 INT(6) ZEROFILL);`可以用0填充高位数。

所有的整数类型都有一个可选的属性UNSIGNED（无符号属性），无符号整数类型的最小取值为0。所以，如果需要在MySQL数据库中保存非负整数值时，可以将整数类型设置为无符号类型。特别地，如果在MySQL中创建数据表时，指定数据字段为ZEROFILL，则MySQL会自动为当前列添加UNSIGNED属性。

#### 浮点数

浮点数类型主要有两种：单精度浮点数FLOAT和双精度浮点数DOUBLE。

![img](/images/posts/2024-05-28-Mysql-commond.assets/da577de74fd24084a2da99601fff16eb.jpg)

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_9,shadow_100,t_100,g_se,order_0,align_2,interval_4.jpeg)

对于浮点数来说，可以使用(M,D)的方式进行表示，(M,D)表示当前数值包含整数位和小数位一共会显示M位数字，其中，小数点后会显示D位数字，M又被称为精度，D又被称为标度。

```mysql
mysql> CREATE TABLE t7 (    
	-> f FLOAT(5,2),    
	-> d DOUBLE(5,2)    
	-> );
```

#### 定点数

MySQL中的定点数类型只有DECIMAL一种类型。DECIMAL类型也可以使用(M,D)进行表示，其中，M被称为精度，是数据的总位数；D被称为标度，表示数据的小数部分所占的位数。定点数在MySQL内部是以字符串的形式进行存储的，它的精度比浮点数更加精确，适合存储表示金额等需要高精度的数据。

DECIMAL(M,D)类型的数据的最大取值范围与DOUBLE类型一样，但是有效的数据范围是由M和D决定的。而DECIMAL的存储空间并不是固定的，由精度值M决定，总共占用的存储空间为M+2个字节。定点数类型中的DECIMAL类型不指定精度时，默认为DECIMAL(10,0)。

#### 日期和时间类型

MySQL提供了表示日期和时间的数据类型，主要有YEAR类型、TIME类型、DATE类型、DATETIME类型和TIMESTAMP类型。

·DATE类型通常用来表示年月日；

·DATETIME类型通常用来表示年、月、日、时、分、秒；

·TIME类型通常用来表示时、分、秒。

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_11,shadow_100,t_100,g_se,order_0,align_2,interval_4.jpeg)

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_11,shadow_100,t_100,g_se,order_0,align_2,interval_4-1716906865800-3.jpeg)

##### YEAR类型

在MySQL中，YEAR有以下几种存储格式：

·以4位字符串或数字格式表示YEAR类型，其格式为YYYY，==最小值为1901，最大值为2155==。

·以2位字符串格式表示YEAR类型，==最小值为00，最大值为99==。其中，当取值为00到69时，表示2000到2069；当取值为70到99时，表示1970到1999。如果插入的数据超出了取值范围，则MySQL会将值自动转换为2000。

·以2位数字格式表示YEAR类型，==最小值为1，最大值为99==。其中，当取值为1到69时，表示2001到2069，当取值为70到99时，表示1970到1999，输入00时，会存入0000。

##### TIME类型

（1）可以使用带有冒号的字符串，比如D HH:MM:SS、HH:MM:SS、HH:MM、D HH:MM、D HH或SS格式，都能被正确地插入TIME类型的字段中。其中D表示天，其最小值为0，最大值为34。如果使用带有D格式的字符串插入TIME类型的字段时，D会被转化为小时，计算格式为D*24+HH。

（2）可以使用不带有冒号的字符串或者数字，格式为"HHMMSS"或者HHMMSS。如果插入一个不合法的字符串或者数字，MySQL在存储数据时，会将其自动转化为00:00:00进行存储。

（3）使用CURRENT_TIME或者NOW()，会插入当前系统的时间。

注意：当使用带有冒号并且不带D的字符串表示时间时，表示当天的时间，比如12:10表示12:10:00，而不是00:12:10；当使用不带冒号的字符串或数字表示时间时，MySQL会将最右边的两位解析成秒，比如1210，表示00:12:10，而不是12:10:00。

##### DATE类型

DATE类型表示日期，没有时间部分，格式为YYYY-MM-DD，其中，YYYY表示年份，MM表示月份，DD表示日期。需要3个字节的存储空间。在向DATE类型的字段插入数据时，同样需要满足一定的格式条件。

·以YYYY-MM-DD格式或者YYYYMMDD格式表示的字符串日期，其最小取值为1000-01-01，最大取值为9999-12-03。

·以YY-MM-DD格式或者YYMMDD格式表示的字符串日期，此格式中，年份为两位数值或字符串满足YEAR类型的格式条件为：当年份取值为00到69时，会被转化为2000到2069；当年份取值为70到99时，会被转化为1970到1999。

·以YYYYMMDD格式表示的数字日期，能够被转化为YYYY-MM-DD格式。

·以YYMMDD格式表示的数字日期，同样满足年份为两位数值或字符串YEAR类型的格式条件为：当年份取值为00到69时，会被转化为2000到2069；当年份取值为70到99时，会被转化为1970到1999。

·使用CURRENT_DATE或者NOW()函数，会插入当前系统的日期。

##### DATETIME类型

DATETIME类型在所有的日期时间类型中占用的存储空间最大，总共需要8个字节的存储空间。在格式上为DATE类型和TIME类型的组合，可以表示为YYYY-MM-DD HH:MM:SS，其中YYYY表示年份，MM表示月份，DD表示日期，HH表示小时，MM表示分钟，SS表示秒。

在向DATETIME类型的字段插入数据时，同样需要满足一定的格式条件。

·以YYYY-MM-DD HH:MM:SS格式或者YYYYMMDDHHMMSS格式的字符串插入DATETIME类型的字段时，最小值为10000-01-01 00:00:00，最大值为9999-12-03 23:59:59。

·以YY-MM-DD HH:MM:SS格式或者YYMMDDHHMMSS格式的字符串插入DATETIME类型的字段时，两位数的年份规则符合YEAR类型的规则，00到69表示2000到2069；70到99表示1970到1999。

·以YYYYMMDDHHMMSS格式的数字插入DATETIME类型的字段时，会被转化为YYYY-MM-DD HH:MM:SS格式。

·以YYMMDDHHMMSS格式的数字插入DATETIME类型的字段时，两位数的年份规则符合YEAR类型的规则，00到69表示2000到2069；70到99表示1970到1999。

·使用函数==CURRENT_TIMESTAMP()和NOW()==，可以向DATETIME类型的字段插入系统的当前日期和时间。

##### TIMESTAMP类型

TIMESTAMP类型也可以表示日期时间，其显示格式与DATETIME类型相同，都是YYYY-MM-DD HH:MM:SS，需要4个字节的存储空间。但是TIMESTAMP存储的时间范围比DATETIME要小很多，只能存储“1970-01-01 00:00:01 UTC”到“2038-01-19 03:14:07 UTC”之间的时间。其中，UTC表示世界统一时间，也叫作世界标准时间。

如果向TIMESTAMP类型的字段插入的时间超出了TIMESTAMP类型的范围，则MySQL会抛出错误信息。

实际上，TIMESTAMP在存储数据的时候是以UTC（世界统一时间，也叫作世界标准时间）格式进行存储的，存储数据的时候需要对当前时间所在的时区进行转换，查询数据的时候再将时间转换回当前的时区。因此，使用TIMESTAMP存储的同一个时间值，在不同的时区查询时会显示不同的时间。

```mysql
show variables like 'time_zone'; #显示时区
set time_zone = '+6:00' #设置时区
```

#### 文本字符串类型

MySQL中，文本字符串总体上分为CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT、ENUM、SET和JSON等类型。

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_16,shadow_100,t_100,g_se,order_0,align_2,interval_4.jpeg)

##### CHAR与VARCHAR类型

CHAR类型的字段长度是固定的，为创建表时声明的字段长度，最小取值为0，最大取值为255。如果保存时，数据的实际长度比CHAR类型声明的长度小，则会在右侧填充空格以达到指定的长度。当MySQL检索CHAR类型的数据时，CHAR类型的字段会去除尾部的空格。对于CHAR类型的数据来说，定义CHAR类型字段时，声明的字段长度即为CHAR类型字段所占的存储空间的字节数。

VARCHAR类型修饰的字符串是一个可变长的字符串，长度的最小值为0，最大值为65535。检索VARCHAR类型的字段数据时，会保留数据尾部的空格。VARCHAR类型的字段所占用的存储空间为字符串实际长度加1个字节。

##### TEXT类型

在MySQL中，Text用来保存文本类型的字符串，总共包含4种类型，分别为TINYTEXT、TEXT、MEDIUMTEXT和LONGTEXT类型。在向TEXT类型的字段保存和查询数据时，不会删除数据尾部的空格，这一点和VARCHAR类型相同。其中，每种TEXT类型保存的数据长度和所占用的存储空间不同。

##### ENUM类型

ENUM类型也叫作枚举类型，ENUM类型的取值范围需要在定义字段时进行指定，其所需要的存储空间由定义ENUM类型时指定的成员个数决定。当ENUM类型包含1～255个成员时，需要1个字节的存储空间；当ENUM类型包含256～65535个成员时，需要2个字节的存储空间。ENUM类型的成员个数的上限为65535个。

当ENUM类型的字段没有声明为NOT NULL时，插入NULL也是有效的。

注意：在定义字段时，如果将ENUM类型的字段声明为NULL时，NULL为有效值，默认值为NULL；如果将ENUM类型的字段声明为NOT NULL时，NULL为无效的值，默认值为ENUM类型成员的第一个成员。另外，ENUM类型只允许从成员中选取单个值，不能一次选取多个值。

##### SET类型

SET表示一个字符串对象，可以包含0个或多个成员，但成员个数的上限为64。当SET类型包含的成员个数不同时，其所占用的存储空间也是不同的。

注意：SET类型在选取成员时，可以一次选择多个成员。

当向表中的SET类型的字段s插入重复的SET类型成员时，MySQL会自动删除重复的成员。

当向SET类型的字段插入SET成员中不存在的值时，MySQL会抛出错误。

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_11,shadow_100,t_100,g_se,order_0,align_2,interval_4-1716948510321-8.jpeg)

##### JSON类型

在MySQL 5.7中，就已经支持JSON数据类型。在MySQL 8.x版本中，JSON类型提供了可以进行自动验证的JSON文档和优化的存储结构，使得在MySQL中存储和读取JSON类型的数据更加方便和高效。

当需要检索JSON类型的字段中数据的某个具体值时，可以使用“->”和“->>”符号。

#### 二进制字符串类型

MySQL中的二进制字符串类型主要存储一些二进制数据，比如可以存储图片、音频和视频等二进制数据。MySQL中支持的二进制字符串类型主要包括BIT、BINARY、VARBINARY、TINYBLOB、BLOB、MEDIUMBLOB和LONGBLOB类型。

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_11,shadow_100,t_100,g_se,order_0,align_2,interval_4-1716948649879-11.jpeg)

![img](/images/posts/2024-05-28-Mysql-commond.assets/watermark,text_ZG1mZW5vd2JlaWppbmc,color_FFFFFF,size_7,shadow_100,t_100,g_se,order_0,align_2,interval_4.jpeg)

##### BIT类型

BIT类型中，每个值的位数最小值为1，最大值为64，默认的位数为1。BIT类型中存储的是二进制值。

##### BINARY与VARBINARY类型

BINARY类型为定长的二进制类型，当插入的数据未达到指定的长度时，将会在数据后面填充“\0”字符，以达到指定的长度。同时BINARY类型的字段的存储空间也为固定的值。

VARBINARY类型为变长的二进制类型，长度的最小值为0，最大值为定义VARBINARY类型的字段时指定的长度值，其存储空间为数据的实际长度值加1。

##### BLOB类型

MySQL中的BLOB类型包括TINYBLOB、BLOB、MEDIUMBLOB和LONGBLOB 4种类型，可以存储一个二进制的大对象，比如图片、音频和视频等。

需要注意的是，在实际工作中，往往不会在MySQL数据库中使用BLOB类型存储大对象数据，通常会将图片、音频和视频文件存储到服务器的磁盘上，并将图片、音频和视频的访问路径存储到MySQL中。
