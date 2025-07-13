# MySQL Study Note

## 安装过程中遇到的注意事项
### 主机名设置 - LocalHost or  %
LocalHost代表的是用户只能从MySQL服务器所在的**同一台机器**上链接
% 代表的是 All Hosts 表示用户可以**从任何主机**连接MySQL服务器

### 关于数据库管理账号的管理
root账号是最高权限账号。创建好之后妥善保管账号密码；
日常对数据库进行管理不要使用root账号。创建一个管理员级别账号用于日常管理 - *DBAdmin -级别的账号。通过workbench为这个账号创建链接，每次使用这个链接登录数据库。

## MySQL 关键概念
### Key的概念
`Key`是数据库中用于标识记录、建立约束、或加速查询的数据字段。
`key`不单指一种类型，而是一个统称，包括Primary Key、Foreign Key、Unique Key、Index Key等。

#### Primary Key
又称`主键`，用来**唯一标识每一行数据的字段**。 一张表只能由一个`Primary Key`，但是它可以由多个列组成。
由多个列组成的`Primary Key`称为*复合主键*。

***需要注意的是： 它的值不能重复； 不允许为 null；默认创建唯一索引***
***另外，通常建议使用一个简单的、不可变的字段作为主键，比如自增ID。 如果主键是复合键，需要注意后续的索引策略和表关联的复杂性***

#### Foreign Key
它的作用是**用于建立两个表之间的关系**。`Foreign key` 引用另一个表的主键或唯一键。
在设置`Foreign Key`的表是子表，这个key引用的表称为父表。
子表上Foreign Key的类型必须与附表的引用值类型一致。

***建议对外键加索引，提高Join性能***
***另外使用外键之后因为产生了表之间的关联，所以在Insert或Delete的时候就要注意数据插入的顺序。 插入的时候父表要先插入，否则当父表对应键值不存在时，子表插入会失败； 对应的删除父表数据的时候，要先删除子表的外键关联的数据***

如果设置外键时定义了 `ON DELETE Cascade`。这个操作可以在删除父表内容时，自动删除对应子表的外键行。
```SQL
FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE

-- 删除操作：
DELETE FROM users WHERE user_id = 100;  -- 自动删除orders表中相关记录
```



## Structred Query Language 语法
### 每一条命令以 *;* 结束
在SQL中每一条命令都已semicolon *;* 结束

## SQL 命令
***SQL命令可以些大写也可以些小写。现在的workbench会自动把大写的命令输入转换为小写的关键字。***

### SQL命令的分类
#### DDL - Data Definition Language - 用来创建和定义数据的名命令。
`Create` - create a databse,and its tables, columns and indexes.

`Alter` - Alter the structure of the database objects - add aor remove columns, indexes, etc.

`Drop` - Delete tables, indexes, and even the entire database.

`Rename` - Rename a table.

`Truncate` - clear out the contents of a table. Effectively the same as deleting and re-creating the table.

#### DML - Data Manipulation Language - 用来操作数据的命令。
`Select` - Query the database to retrive rows of data.

`Insert` - Insert data into a table.

`Update` - Change the data in columns of a table(s).

`Delete` - Delete rows in a table(s).

### 基础查询
`SHOW DATABASES;` - 展示所有一创建的databases
`SHOW TABLES；` - 展示*当前*仓库所有的tables， 这个命令需要进入一个Database之后才有用

### 基础使用
#### 针对DataBase操作
`USE <DataBaseName>；`  进入一个已经存在的Database
`DROP DATABASE <DataBaseName>;` 删除一个名为 *DataBaseName*的Database。有趣的是`删除`一个database不是Delete而是**`Drop`**
`CREATE DATABASE <DataBaseName>;`   创建一个指定名称为 *DataBaseName*的Database