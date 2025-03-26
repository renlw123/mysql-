# 字符集的相关操作

## 修改MySQL5.7字符集

在MySQL8版本之前，默认的字符集为latin1，utf8字符集指向的是utf8mb3。开发人员在数据库设计的时候往往会将编码修改为utf8字符集。如果遗忘修改默认的编码，就会出现乱码问题。从MySQL8.0开始数据库的默认编码改为了utf8mb4，从而避免上述乱码的问题。

### MySQL5.7查看字符集编码

```sql
mysql> show variables like '%character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.04 sec)

mysql> 

```

### MySQL8.0查看字符集编码

```sql
mysql> show variables like '%character%';
+-------------------------------------------------+--------------------------------+
| Variable_name                                   | Value                          |
+-------------------------------------------------+--------------------------------+
| character_set_client                            | utf8mb4                        |
| character_set_connection                        | utf8mb4                        |
| character_set_database                          | utf8mb4                        |
| character_set_filesystem                        | binary                         |
| character_set_results                           | utf8mb4                        |
| character_set_server                            | utf8mb4                        |
| character_set_system                            | utf8mb3                        |
| character_sets_dir                              | /usr/share/mysql-8.0/charsets/ |
| validate_password.changed_characters_percentage | 0                              |
+-------------------------------------------------+--------------------------------+
9 rows in set (0.02 sec)

mysql> 

```

### MySQL5.7操作

```sql

mysql> create database dbtest1;
Query OK, 1 row affected (0.01 sec)

mysql> use dbtest1
Database changed
mysql> create table emp1(id int, name varchar(16));
Query OK, 0 rows affected (0.04 sec)
mysql> select * from emp1;
Empty set (0.02 sec)

mysql> insert into emp1 (id, name) values (1, 'tom');
Query OK, 1 row affected (0.01 sec)

mysql> insert into emp1 (id, name) values (1, '任立伟');
ERROR 1366 (HY000): Incorrect string value: '\xE4\xBB\xBB\xE7\xAB\x8B...' for column 'name' at row 1

mysql> show create table emp1;
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| emp1  | CREATE TABLE `emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

#### MySQL5.7中修改字符集编码

```shell
vi /etc/my.cnf
```



```shell
[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

bind-address = 0.0.0.0

character_set_server=utf8
```

#### 再次查看MySQL5.7编码

```sql
mysql> show variables like '%character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

#### 但是这里不会对已经存在的数据库字符集进行重置

```sql
mysql> show create table emp1;
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| emp1  | CREATE TABLE `emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

#### 并且在之前的Latin1的库下创建新表字符集也会一直都是latin1

```sql
mysql> show create table emp1;
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| emp1  | CREATE TABLE `emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> use dbtest1
Database changed
mysql> create table database emp2(id int, name varchar(16));
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'database emp2(id int, name varchar(16))' at line 1
mysql> create table  emp2(id int, name varchar(16));
Query OK, 0 rows affected (0.02 sec)

mysql> show create table emp2;
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| emp2  | CREATE TABLE `emp2` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> 
```

#### 在修改了字符集后创建新的数据库表

```sql
mysql> create database dbtest2;
Query OK, 1 row affected (0.00 sec)

mysql> use dbtest2;
Database changed
mysql> create table emp1(id int, name varchar(16));
Query OK, 0 rows affected (0.02 sec)

mysql> insert into emp1(id, name) value (1, '任立伟');
Query OK, 1 row affected (0.00 sec)

mysql> select * from emp1;
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | 任立伟    |
+------+-----------+
1 row in set (0.00 sec)

mysql> show create emp1;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'emp1' at line 1
mysql> show create table emp1;
+-------+---------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                              |
+-------+---------------------------------------------------------------------------------------------------------------------------+
| emp1  | CREATE TABLE `emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+---------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> 
```

#### 修改已有的数据库字符集latin1为utf8

```sql
mysql> use dbtest1;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from emp1;
+------+------+
| id   | name |
+------+------+
|    1 | tom  |
+------+------+
1 row in set (0.00 sec)

mysql> alter database dbtest1 character set 'utf8';

mysql> show create database dbtest1;
+----------+------------------------------------------------------------------+
| Database | Create Database                                                  |
+----------+------------------------------------------------------------------+
| dbtest1  | CREATE DATABASE `dbtest1` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> show create table emp1;
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
| emp1  | CREATE TABLE `emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+-----------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)


mysql> alter table emp1 convert to character set 'utf8';
Query OK, 1 row affected (0.02 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> show create table emp1;
+-------+---------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                              |
+-------+---------------------------------------------------------------------------------------------------------------------------+
| emp1  | CREATE TABLE `emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+---------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```



### MySQL8.0操作

```sql
mysql> create database dbtest1;
Query OK, 1 row affected (0.02 sec)
mysql> use dbtest1
Database changed
mysql> use dbtest1;
Database changed
mysql> create table emp1(id int, name varchar(16));
Query OK, 0 rows affected (0.02 sec)
mysql> select * from emp1;
Empty set (0.02 sec)

mysql> insert into emp1 (id, name) values (1, 'tom');
Query OK, 1 row affected (0.01 sec)
mysql> select * from emp1;
+------+------+
| id   | name |
+------+------+
|    1 | tom  |
+------+------+
1 row in set (0.00 sec)

mysql> insert into emp1 (id, name) values (1, '任立伟');
Query OK, 1 row affected (0.01 sec)
mysql> select * from emp1;
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | tom       |
|    1 | 任立伟    |
+------+-----------+
2 rows in set (0.00 sec)

mysql> show create table emp1;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                        |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------+
| emp1  | CREATE TABLE `emp1` (
  `id` int DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> 
```

## 层级关系（字符集继承关系-如果某个层级未指定字符集，则会继承上一级默认值）

| 层级                    | 说明                                      | 设置方式                                                     | 继承关系                   |
| ----------------------- | ----------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| **服务器级** (Server)   | 服务器的默认字符集和排序规则              | `character_set_server` 和 `collation_server` 变量            | 全局默认值，影响新建数据库 |
| **数据库级** (Database) | 数据库的默认字符集和排序规则              | `CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;` | 继承服务器级默认值         |
| **表级** (Table)        | 表的默认字符集和排序规则                  | `CREATE TABLE mytable (...) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;` | 继承数据库级默认值         |
| **字段级** (Column)     | 字段的字符集和排序规则                    | `name VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;` | 继承表级默认值             |
| **连接级** (Connection) | 客户端与 MySQL 服务器之间的数据传输字符集 | `SET NAMES utf8mb4;` 或 `SET character_set_client = utf8mb4;` | 独立于数据库和表           |
| **语句级** (Statement)  | SQL 语句内部指定的字符集                  | `SELECT '你好' COLLATE utf8mb4_general_ci;`                  | 独立于表和字段             |

