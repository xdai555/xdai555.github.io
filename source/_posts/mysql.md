## 介绍

1. 是一个**关系型数据库**管理系统，开源，体积小，速度快
2. 存储引擎： 
   - MyISAM：MySQL5.0之前的默认数据库引擎，最为常用。拥有较高的插入，查询速度，但不支持事务
   - **InnoDB**：事务型数据库的首选引擎，支持ACID事务，支持行级锁定，MySQL5.5起成为默认数据库引擎
   - MEMORY引擎：MEMORY存储引擎提供“内存”表，也不支持事务、外键。
   - ARCHIVE引擎：ARCHIVE存储引擎是被设计用来存储企业中的大量流水数据的存储引擎

## 服务管理

名称：mysqld （新版是 mariadb）

默认端口：3306

安装：`yum -y groupinstall "MySQL Database server" `

`yum install mariadb-server` 安装

安装完成后，使用`systemctl start mariadb `开启服务，`lsof -i:3306`查看端口

 控制台命令使用 `;`或者`\g` 结束，使用`\c`来清除当前行的语句；

显示所有的数据库`show databases;`，`use xxx;`选择数据库

元数据查询：

服务器版本信息``SELECT VERSION();``
当前数据库名``SELECT DATABASE();``
当前用户名``SELECT USER();``
服务器配置变量``SHOW VARIABLES;``
服务器状态``SHOW STATUS;``



配置文件位于 `/etc/my.cnf`

```shell
[root@kvm ~]# cat /etc/my.cnf
[mysqld]
# 数据库文件
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# 设置服务器编码格式，修改后需要重启服务
character-set-server=utf8
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
# 设置客户端编码格式，修改后不需要重启服务
[client]
default-character-set=utf8
```



## SQL 语句

SQL： structured query language 结构化查询语言

### 语法组成

#### DML 数据操作语言：增删改查

select、insert、update、delete

- 增加数据

```
insert into <表名> [列名] values(<值列表>)
# 插入单条数据
insert into student(name,sex,birthday) values('eric',1,'20000101');
# 插入多条数据
insert into student(name,sex,birthday) values('eric',1,'20000101'),('zhangsan',1,'19990101'),('lisi',1,'19980101');
# 查看数据
MariaDB [db01]> select * from student;
+----+----------+-----+------------+
| id | name     | sex | birthday   |
+----+----------+-----+------------+
|  1 | eric     |   1 | 2000-01-01 |
|  2 | eric     |   1 | 2000-01-01 |
|  3 | zhangsan |   1 | 1999-01-01 |
|  4 | lisi     |   1 | 1998-01-01 |
+----+----------+-----+------------+
4 rows in set (0.00 sec)

# 添加可以不写字段名字，不写的话是顺序赋值
MariaDB [db01]> insert into student values(5,'wangwu',1,'19980101');
MariaDB [db01]> select * from student where id=5;
+----+--------+-----+------------+
| id | name   | sex | birthday   |
+----+--------+-----+------------+
|  5 | wangwu |   1 | 1998-01-01 |
+----+--------+-----+------------+
1 row in set (0.00 sec)
```

- 查找数据

  ```
  select 
  [FROM <distinct(去重)> <as 别名> <tabble>]
    [WHERE] [GROUP BY] [HAVING] [ORDER BY] [LIMIT] [PROCEDURE]
  
  from子句：指定查询数据的表
  where子句：查询数据的过滤条件
  group by子句：对匹配where子句的查询结果进行分组
  having子句：对分组后的结果进行条件限制。
  order by子句：对查询结果结果进行排序，后面跟desc降序或asc升序（默认）。
  limit子句：对查询的显示结果限制数目
  procedure子句：查询存储过程返回的结果集数据
  
  集函数：count、min、max、sum、avg 等
  select count(*) from student;
  
  ```

  - where 子句
    - [NOT] ETWEEN   AND：是否在两数之间：`select * from student where id between 2 and 3;`
    - [NOT] IN <值表>  ：是否在特定的集合里`select * from student where id  in (1,3,5);`
    - LIKE ：是否匹配
    - IS [NOT] NULL  Regex：判断是否为空

  

- 更新数据

  update  <表名> set <列名 = 更新值> where <更新条件>  [limit=1] （多条数据限制修改）

  `update student set name='lisi' where name='zhangsan' ;  # 把张三改为李四`

- 删除数据

  delete from <表名> [where <条件>] [limit=1]
  `delete student where name='zhangsan'; # 把张三删除`

  

  **多表查询**：连接查询

  - 内连接 INNER JOIN

  - 外连接 OUTER JOIN

    左外连接 LEFT JOIN ：返回左表当中的所有行，如果左表行在右表中无匹配行，则结果中显示空

    右外连接 RIGHT JOIN

  - 查询结果合并：相同结构的多张表合并

  - 子查询



#### DCL 数据控制语言：控制存取许可、权限

grant、revoke

#### DDL 数据定义语言：建立数据库、数据库对象

create table、drop table、alter table

`create database <name>; `

`drop database <name>;`

#### 功能函数

日期、数学、字符、系统



查看表：`desc <table>`，`show create table user;`

查看列：`show columns from user;`

数据类型：数字、字符串、日期



创建数据表 student：

```
MariaDB [db01]> create table student(
    -> id int unsigned not null auto_increment primary key,
    -> name varchar(20) not null,
    -> sex tinyint not null,
    -> birthday date not null
    -> )
    -> ;
```



查看 `show create table student;`  `desc student;`

临时表：只在当前会话有效，退出后消失

创建：

```
MariaDB [db01]> create TEMPORARY table student2(
    -> id int unsigned not null auto_increment primary key,
    -> name varchar(20) not null,
    -> sex tinyint not null,
    -> birthday date not null
    -> )
    -> ;
```



## 日志查看

- 错误日志：记录MySQL服务器启动、关闭和运行时出错等信息

- 查询日志：记录MySQL服务器的启动和关闭信息、客户端的连接信息、更新数据库记录SQL语句和查询数据库记录SQL语句 `show variables like 'log_error%'`

- 慢查询日志：记录执行时间超过指定时间的查询语句，通过工具分析慢查询日志可以定位MySQL服务器性能瓶颈所在  `show variables like 'slow_query_log%'`，默认关闭 ，修改配置文件永久生效`slow_query_log = 1`

  `mysqldumpslow` 查看总数

- 二进制日志：以二进制形式记录数据库的各种操作，但不记录查询语句



  `mysql> show processlist;` 查看当前连接状态



## 数据备份与导入

`mysqldump -uroot -pPassword [database name]` 只备份所有的表，不备份数据库

恢复的时候要先创建对应的表，再进行恢复

`mysqldump db01 > /tmp/db01.sql`



第一种恢复方法：`mysql -uroot -ppassword [database name]< /tmp/db01.sql`

第二种：进入数据库后，`source /tmp/db01.sql`