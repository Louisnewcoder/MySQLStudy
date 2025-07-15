# MySQL Study Note

## 安装过程中遇到的注意事项
### 主机名设置 - LocalHost or  %
LocalHost代表的是用户只能从MySQL服务器所在的**同一台机器**上链接
% 代表的是 All Hosts 表示用户可以**从任何主机**连接MySQL服务器

### 关于数据库管理账号的管理
root账号是最高权限账号。创建好之后妥善保管账号密码；
日常对数据库进行管理不要使用root账号。创建一个管理员级别账号用于日常管理 - *DBAdmin -级别的账号。通过workbench为这个账号创建链接，每次使用这个链接登录数据库。

## MySQL 关键概念
### 约束
约束是数据库的强制规则，要求数据只在符合规则的前提下才能存入数据库。比如：
指定某列不能为Null；
指定某个列的值必须>=100才能存入；
......

是约束的内容有： `Primary Key`, `Foreign Key`, `Unique`, `Check/Not Null`。
另外 `Default` 有的资料称之为约束，有的不是。因为它不强制，只是数据库自己填了默认值。

#### Key的概念
`Key`是数据库中用于标识记录、建立约束、或加速查询的数据字段。
`key`不单指一种类型，而是一个统称，包括Primary Key、Foreign Key、Unique Key、Index Key等。

##### Primary Key
又称`主键`，用来**唯一标识每一行数据的字段**。 一张表只能由一个`Primary Key`，但是它可以由多个列组成。
由多个列组成的`Primary Key`称为*复合主键*。

***需要注意的是： 它的值不能重复； 不允许为 null；默认创建唯一索引***
***另外，通常建议使用一个简单的、不可变的字段作为主键，比如自增ID。 如果主键是复合键，需要注意后续的索引策略和表关联的复杂性***

##### Foreign Key
它的作用是**用于建立两个表之间的关系**。`Foreign key` 引用另一个表的主键或唯一键。
在设置`Foreign Key`的表是子表，这个key引用的表称为父表。
子表上Foreign Key的类型必须与附表的引用值类型一致。

***建议对外键加索引，提高Join性能***。但实际上自己看的教程提到，foreign key创建时，会添加一个index。
***另外使用外键之后因为产生了表之间的关联，所以在Insert或Delete的时候就要注意数据插入的顺序。 插入的时候父表要先插入，否则当父表对应键值不存在时，子表插入会失败； 对应的删除父表数据的时候，要先删除子表的外键关联的数据***

如果设置外键时定义了 `ON DELETE Cascade`。这个操作可以在删除父表内容时，自动删除对应子表的外键行。
```SQL
FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE

-- 删除操作：
DELETE FROM users WHERE user_id = 100;  -- 自动删除orders表中相关记录
```
##### Unique Key
限制指定列的值不能重复。比如个名称列，不能有两个或以上的人叫同一个名字。


### Index的概念
**索引不是约束**。索引是加速查询操作的数据结构，索引通常建立在表的一个或多个字段上，便于快速定位数据，而不必扫描整个表。
它对 `Where`, `Join`, `Order by`, `Gorup by`等查询有效率提升。**表越大越重要**。

##### 使用索引的缺点
会占用额外的空间，另外执行`Insert/Update/Delete`等操作时，索引也需要维护，可能会影响性能。

***需要注意的是： 可以在经常用来查询、排序、过滤的字段上建议添加索引； 避免在频繁更新的字段上建立太多索引； 不要对低选择度的字段添加索引(比如只有2-3个值的字段，男女，大小，真假等)***



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
`DESCRIBE <table name>;` - 展示对指定表的描述


### 基础使用
#### 针对DataBase操作
`USE <DataBaseName>；`  进入一个已经存在的Database
`DROP DATABASE <DataBaseName>;` 删除一个名为 *DataBaseName*的Database。有趣的是`删除`一个database不是Delete而是**`Drop`**
`CREATE DATABASE <DataBaseName>;`   创建一个指定名称为 *DataBaseName*的Database




##### 针对表的操作
###### 创建表
`CREATE TABLE <TableName> ( column声明1, column声明2，...... column声明n)；`

```sql
create table products(
	id int auto_increment primary key,  # <column Name> Type 自增 为主键
    name varchar(30), # <column Name> Type(长度为30)
    price decimal(3,2) # <column Name> Type(一共3位数，小数点后保留2位）
);
```
###### 删除表
`DROP TABLE <Tablename>;` - 这个命令会将整个表从database中移除

###### 清除表的内容但保留表 Truncate Table
`Truncate TABLE <TableName>`

###### 创建Foreign Key
在创建表的时候，一般在定义完表内的各个 `字段(列)` 之后，使用语法对 `指定用于Foreign Key的列进行绑定`。
***注意： 子表中用于指定作为foreign key的 column 和 父表的parent key column都是用 `()` 包裹的***
`FOREGIN KEY  ( <column name for foreign key> )  REFERENCES <parentTableName>(<parent Key Column>)`

```sql
create table orders(
    id int auto_increment primary key,
    product_id int,
    customer_id int,
    order_time datetime,
    foreign key (product_id) references products(id),
    foreign key (customer_id) references customers(id)
);
```

###### 修改表
修改表的命令以`ALTER TABLE <target Table name>` 开始，然后加上要进行的`操作的指令`

1. 插入列
`ALTER TABLE <table name> ADD COLUMN <column name> COLUMNTYPE`

```sql
alter table proudcts add column coffee_origin varchar(30); --对表 products 添加一列，字段为 coffee_origin， 类型为varchar其长度为30

-- 也可以一次添加多个列，只需要连续添加使用 add column子句即可
 alter table pets add column test1 int, add column test2 varchar(10);
```
2. 删除列
`ALTER TABLE <table name> DROP COLUMN <column name>`

```sql
alter table products drop column coffee_origin; -- 在products表上，删除列名为  coffee_origin 的列

-- 也可以一次删除多个列，只需要连续添加 drop column 子句即可
 alter table pets drop column test1, drop column test2;
```

3. 添加Primary key
`ALTER TABLE <table name> ADD PRIMARY KEY (<column name>);`
**注意：指定的column name是被 `()` 包裹的**

4. 删除Primary Key
先删除`Auto_Increment约束`,然后执行下面的命令。
`ALTER TABLE <table name> DROP PRIMARY KEY;`
***要注意的是： 要先删除 `AUTO_INCREMENT` 约束***


5. 修改 当前列是否可以接受 NULL 值
只需要在`ALTER TABLE` 语句中使用 `MODIFY <column name> <original type>`

```sql
Alter table addresses modify id int;
```
6. 设置Foreign Key
如果是单独设置 foreign key的话，在`ALTER TABLE`后 添加一个`constraint` 名称，然后使用foreign key的常规绑定语句。

***注意：很多时候在创建table的时候没有声明constraint 是因为MySQL自动为指定为ForeignKey的column创建了默认名称的constraint。实际上是非常建议开发者在这个时候手动指定constraint名称的***

***另外，创建foreign key的时候MySQL会自动创建一个同名（constraint名称）索引，所以删除的时候也应该这个索引删掉***

```sql
 alter table people 
 add constraint FK_PeopleAddress -- 单独声明constraint的名字，方便后面管理
 foreign key (address_id) references addresses(id);
```

7. 删除Foreign Key
```sql
    alter table people
    drop foreign key FK_NewFKAddress,  -- 这个指令与下面一行是同级，所以用`,`分隔。
    drop index FK_NewFKAddress; -- 把系统创建的同名索引也删掉

    -- 也可以分开写效果一样
    alter table people drop foreign key FK_NewFKAddress;

    alter table people drop index FK_NewFKAddress;
```

8. 添加与删除 Unique 约束
在`ALTER TABLE`语句后，使用 ` ADD CONSTRAINT <constraint name> UNIQUE (<target column>);` 语法添加Unique CONSTRAINT；
然后使用 `DROP INDEX <constraint name>;` 删除对应的约束。

```sql
describe pets;
Alter table pets
add constraint unique_species Unique (species);

alter table pets
drop index unique_species;
```

9. 修改列的名字
可以使用 `Change` 命令和 `Rename column`命令。

```sql

    -- 使用Change 命令 必须要输入 TYPE 如果不改就输入原来的TYPE类型
    alter table pets
    change species animal_type varchar(20); -- CHANGE <current name> <target name> TYPE

    -- 使用 Rename column命令只更改名字 必须使用 TO 关键字链接
    alter table pets 
    rename column animal_type to species; -- RENAME COLUMN <current name> TO <target name>;
```

10. 修改列的类型
除了前面提到的`Change`命令，还可以使用`Modify Column <column name> [TYPE]`。
***需要注意的是： 如果数据表中这一列还没有数据之前，可以随便转换。但是当有了数据之后，只能进行兼容的转换。比如将字段从varchar 转换成char，或者int都是不允许的。 从同类型，从小转到大就可以，比如 int 到bigint， decimal(5,2) 转到 decimal(10,3)***
一些特殊情况`DATE`类型到`varchar`类型是允许的。 

***另外，当想给一个字段添加约束是，比如 not null。 如果已经有了数据，必须要保证当前所有数据的值都不是null 才能修改成功。***

```sql
    alter table pets 
    modify column food int; -- 把名为 food的列从一个其他的类型转换为 int   

```
## Data Manipulation Language
### Insert Data
基础语法
数值型数据`不需要引号`， 字符型数值需要 `单引号` 包裹。虽然`双引号`也可能工作，但是可能会有潜在问题。所以要保持良好的习惯使用**单引号**

#### 单次插入一条
`INSERT INTO <table name> ( <column1>, <column2>，...<columnN>  )  VALUES ( column1_value, column2_value, columnN_value);`
#### 单次插入多条
```sql
INSERT INTO <table name> ( <column1>, <column2>，...<columnN>  )  VALUES
 ( column1_value, column2_value, columnN_value),    -- 第一行
 ( column1_value, column2_value, columnN_value),    -- 第二行
 ...
 ( column1_value, column2_value, columnN_value)     -- 第N行
 ; 
 ```

### Update Data
***默认情况 sql_safe_updates 的检查是开着的***
当`sql_safe_updates`开着的时候 `WHERE` 子句只能用`KEY`来匹配，例如用某个表的primary key `id=999`。

如果想使用其他匹配方式最简单的方式是把它关掉 `SET sql_safe_updates=0 / false`。虽然还有其他方法，现在新手阶段先用这个。

使用语法：

```sql
UPDATE <table Name>     
SET <column name> = <value>   -- set 子句设置改变的内容，可以是一个或若干个字段以及每个字段要修改的值
WHERE <match condition>; -- WHERE 子句用来匹配要修改哪行或哪几行的数据，具体修改了多少以匹配结果来定
```
#### 更新一个字段的值
```sql
update products
set price = 9.00   -- set 子句控制改变的内容，这里是price字段
where id=7;         -- 这里只修改 id=7 的那一行数据的 price字段的值
```

#### 更新若干个字段的值
```sql
update products
set price = 5.50, coffee_origin = 'China' -- 更新了price字段和coffee_origin字段的值
where id=7;
```

### Delete Data
使用语法：
```sql
DELETE FROM <table name>
WHERE <match condition>;

-- examples
Delete from people
where first_name = 'Babara';

Delete from people
where age <=30;
```

***警告！！！如果 `sql_safe_updates`关闭了， 那么 `DELETE FROM <table name>;` 这一行命令就可以删除整个表的所有数据。十分危险！！！***
