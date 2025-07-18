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

是约束的内容有： `Primary Key`, `Foreign Key`, `Unique Key`, `Check`,`Null/Not Null`。
另外 `Default` 有的资料称之为约束，有的不是。因为它不强制，只是数据库自己填了默认值。

#### 约束的添加方式
***注意： Key类和Check 型约束，需要用 `ADD CONSTRAINT <constraint name>` 添加。`NULL`和`DEFAULT` 可以用 `MODIFY` 添加***

##### 删除约束
Key类和Check型约束都使用`DROP CONSTRAINT <constriant name>`来删除。如果忘记了或者一开始没有设置，可以在database schema中查看`index`。 


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

### MySQL的赋值运算符与判断运算符
`=` 可以用作赋值运算符，比如在`Update` 语句中 `SET` 任何类型值都是` 字段 = 值 `包括 `NULL`；
而在 `JOIN` 或者 `WHERE` 子句中 `=`可以用作常规值**不能用在`NULL`上**。 而 `IS`和`IS NOT`是专门用在`NULL`上的。

## Structred Query Language 语法
### SQL语句语法顺序
`SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT ...`


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

### Select Data

查询语句永远都使用 `SELECT < column name / or * > FROM <table name>`开始。如果没有子句，这一行就可以作为一个命令获取所输入的字段名的所有数据。
***使用 `*` 代表获取全部column的数据***

***重要的事说3遍： SELECT 后面接的是你要看的字段值 然后 FROM 从哪个 <TABLE表> 里看***
***重要的事说3遍： SELECT 后面接的是你要看的字段值 然后 FROM 从哪个 <TABLE表> 里看***
***重要的事说3遍： SELECT 后面接的是你要看的字段值 然后 FROM 从哪个 <TABLE表> 里看***

```sql
    select first_name from customers; -- 查询 `first_name` 1个列的全部内容

    select first_name,last_name,phone_number from customers; -- 查询 `first_name`, `last_name`, `phone_number` 3个列的全部内容
  
    select * from customers; -- 查询整个表所有列的内容
```
#### 结合WHERE子句
只需要在`SELECT`起始语句后加上`WHERE <column condition>`子句即可。如果有多个条件约束则可以根据需求添加使用`AND <extra column condition>` 或者 `OR <extra column condition>`即可。
```sql
   select * from products  -- 查询获取 全部字段 的数据 从products 表中
    where price>3;          -- 选择price价格字段 值大于3的数据
    
	select * from products
    where price>3                   --选择price价格大于3
    Or coffee_origin='Colombia';  -- 或者 coffee_origin字段等于 colombia的数据
    
    select name from products         -- 查询 name字段 的数据
    where coffee_origin = 'Colombia' -- 选择 coffee_origin字段等于 colombia
    and price=3;                        -- 并且 price价格 等于 3 的数据
```

***PS：在SQL中，数学逻辑符合与C#编程语言一致。 额外多出的是， 不等于符号有两种书写方式，可以自行选择`!=` 或者 `<>` ***

#### Mysql的条件判断运算符
##### 数学判断运算符
`>`, `<`, `>=`, `<=`, `!= or <>`
##### Null判断运算符
`IS NULL` , `IS NOT NULL`

##### 匹配集合判断运算符 IN 和 NOT IN 
一次性在一个字段中查询多个条件
```sql
select * from customers
where id in (1,3,5,7,9); -- 值查询id 匹配 1 3 5 7 9的用户数据

select * from customers
where id not in(1,2,3,4,5,6,7,8,9,10); -- 查询 id 不在1 至 10 的用户 ，也就是从id为11开始获取所有数据

select * from customers
where id not in(1,2,3,4,5,6,7,8,9,10)
and gender is not null;  -- 结合 is not null
```

##### 范围匹配运算符 BETWEEN...AND...
`BETWEEN <start value> AND <end value>`用于查询在 `start value` 和 `end value` 之间的数据。
***它实际等价于***
`WHERE column >= value1 AND column <= value2`
***要注意的是：如果value1 大于 value2，那么就会返回空***
***另外在处理字符串时，value2只会匹配到与value2完全相等的数据行，如果长于value2并不会被包含***
例如:
```sql
-- 数据有 Abc，BCD，CEF，D，Da，FFF
BETWEEN 'A' AND 'D'  -- 只会返回 Abc,BCD,CEF,D。 后面的Da和FFF不会被返回
```

```sql
select * from orders
where order_time between '2023-01-01' and '2023-03-31 23-59-59'; -- 选取 order_time字段值从2023年1月1日凌晨开始到2023年3月31日晚23点59分59秒之间所有的order数据

-- 它的对等操作
where order_time>= '2023-02-01' and order_time<'2023-03-01'; -- 忽略日期，这里仅作展示

-- 还可以利用MySQL内置函数
where year(order_time)=2023 and month(order_time)=2; 


```
##### 模糊匹配运算符 LIKE
使用 `LIKE` 关键字加 ` 带有通配符的字符串 `匹配条件即可模糊查询与`条件相匹配的内容`
***通配符 `%` 代表任意数量，任意字符的内容 ***
***通配符 `_` 代表 `一个` 任意字符***
***另外，模糊查询的条件可以是数值也可以是字符或字符串，但是都需要有用 `''` 单引号包裹起来。***

```sql
select * from customers
where last_name LIKE '%o%'; -- 匹配字段 last_name中 任意包含字母 o 的数据行

select phone_number from customers -- 获取phone_number字段的数据行
where last_name LIKE 'J%';  	-- 匹配条件是 last_name 字段 所有以字母 J 开始的数据

select * from customers
where first_name LIKE '_o__'; -- 匹配条件是 first_name字段 拥有4个字符 第二个字符是 o 的数据

select * from customers
where first_name LIKE '__a'; -- 匹配条件是 first_name字段 拥有3个字符 最后一个字符是 a 的数据

select * from products
where price LIKE '3%';  -- 匹配条件是 price 价格字段 以数值3开头的产品数据
-- 注意这里虽然 price 的类型是 decimal数值类型，LIKE后面的匹配条件依旧用 '' 包裹
```

#### Order Data 对查询结果进行排序
***如果没有 LIMIT 子句 ， ORDER BY 子句永远出现在查询语句的最后。如果有LIMIT子句，LIMIT子句在最后***
默认情况下排序是 `ASC` - ascending 升序，可以使用关键字 `DESC` - descending 设置为降序。
***ORDER BY 子句可以跟随多个 字段条件，它们用 `,` 分隔。***有多个条件的子句排序顺序是按照字段顺序，对结果`逐层级`依次排序处理。

```sql
select * from customers
order by last_name desc; -- 获取所有数据，根据last_name 字段进行降序排列

select * from customers
order by last_name asc , first_name desc;  -- 现根据 last_name 升序排列，然后再前者条件结果的基础上对 分段结果 进行基于 first_name 的升序排列。 
-- 在这个例子中， asc 可以省略，因为默认就是ascending 排序
```

#### Distinct Data 获取唯一值 即 去重
在查询语句的列名前加入关键字`DISTINCT`就可以获取列名或列名组合的唯一值 - **也就是去重**
 `SELECT DISTINCT <column name>`或   `SELECT DISTINCT <column name1>,..,<column namen>,`

 ```sql
 select distinct customer_id from orders; -- 获取 customer_id 去重

 select distinct customer_id,product_id from orders -- 获取 customer_id和product_id的组合 去重
 order by customer_id;
 ```
#### Limit Data 限制每页呈现的数据数量
在查询语句的最后使用`LIMIT <display count>`来限制展示数据的数量。语法：
1. `LIMIT <display count>;`
2. `LIMIT <display count> OFFSET <每页偏移>;`
3. `LIMIT <每页偏移>, <display count>;`

```sql
select * from customers
Limit 5;

select * from customers
Limit 5 offset 10;

select * from customers
Limit 10,5; -- 效果同上

select * from customers
order by first_name     -- 结合 order by 子句
Limit 5 offset 8;
```

#### 利用 AS 关键字提高查询结果中 列名 的可读性
有的时候建表的开发者把column的名字起的比较晦涩，对数据库使用者来说的阅读性不是很好。这时数据库使用者可以利用 AS 关键字将查询结果中的列明进行`展示性修改` (即并不修改实际的列名)，从而方便自己阅读数据库。

`Select <column name> AS <desired name for read> FROM <table name>;`
***AS 关键字是可以省略的，但是同时影响SQL语句的阅读性，所以不建议省略***

```sql
select coffee_origin as country, name as coffee from products;
```

### JOIN 从多个表中查询数据
#### INNER JOIN 查询两个表中的关联值匹配的数据
`inner join` 只会获取 目标值 同时匹配两个表存在的值的数据行。
比如查询id = 3 ，如果A和B表中都有 id=3的数据行，那么这个数据行的值就可以被获取。如果任意一个表中不存在id=3的数据行，那么id=3的数据行就不会被查询。

***因为 `INNER JOIN` 是最常见的查询，所以 `INNER`关键字可以省略 ***

```sql

-- 建议为了可读性， 用主表.主键 = 从表.外键 的规则书写
-- 查询也从主表查询
select products.name , orders.order_time from products 
inner join orders on products.id = orders.product_id;

-- 利用 AS 关键字实现简便写法
select p.name , o.order_time from products as p
inner join orders as o on p.id = o.product_id;

-- 添加了 WHERE 和 ORDER BY 的写法
select products.name , orders.order_time from products
inner join orders on products.id = orders.product_id
where products.id =4
order by orders.order_time;
```
语法总结：
```sql
select <parent table>.<column> ,...,<child table>.<column>,... 
from  <parent table>
inner join <child table> on <parent table>.<primary key> = <child table>.<foreign key>
where clause -- if needed
order by clause; -- if needed
```

#### LEFT JOIN 
以`左表`为基础，查询结果保留`左表`所有内容的同时，显示`右表`所有匹配的内容。若没有匹配的内容 `右表`在对应的`左表`行处显示的结果全部为Null

语法结构：

```sql
Select 字段 
FROM 左表
LEFT JOIN 右表 ON 左表.字段 = 右表.字段

-- examples
select c.*, o.* from customers as c
left join orders as o on c.id = o.customer_id;

-- 结合WHERE子句
select c.*, o.* from customers as c
left join orders as o on c.id = o.customer_id
where o.customer_id is null;
```

#### RIGHT JOIN
以`右表`为基础，查询结果保留`右表`所有内容的同时，显示`左表`所有匹配的内容。若没有匹配的内容 `左表`在对应的`右表`行处显示的结果全部为Null

***它和LEFT JOIN是镜像关系。而且MySQL官方也不建议使用RIGHT JOIN 因为有的数据库不持支这个功能。想使用这个功能，就用LEFT JOIN 把左右表的位置调换即可。***

```sql
Select 字段 
FROM 左表
LEFT JOIN 右表 ON 左表.字段 = 右表.字段
```
#### 多表联合查询 - 使用多个JOIN
先看实例
```sql
select p.name ,p.price ,c.first_name,c.last_name , o.order_time from products as p
inner join orders as o On p.id = o.product_id       -- 两个表JOIN 查询获得结果 R1
inner join customers as c on o.customer_id = c.id   -- 在与第三个表JOIN查询 实际上相当于 R1 与第3个表查询 得到结果R2。如果还有后续的JOIN查询，相当于R2 与下一个表JOIN 查询.... RN与表N进行JOIN查询
where c.last_name = 'Jones'
order by first_name;
```

### 如何创建DataBaseModel diagram并查看
Top Tools bar -> Database -> Reverse Engineer ->选择`目标connection`->选择`目标DataSchema`->一路下一步，遇到勾选全部勾选->保存自己记得diagram

查看的时候， Top Tools bar -> File -> Open Mode -> 要打开查看的 .mwb model文件 

## Concepts of DataBase Design
### Normalization
一种设计思想，让数据库的每张表只保存必要的信息，减少重复和冗余的信息。节省物理空间，提高查询效率。

#### First Norm Form
1. 没有重复的数据行。这里是指一摸一样的数据行。
2. 每一列的每个实例数据，都只存储一个值而非多个值。比如一个表格是`个人信息`，有一列是`喜好`。没行数据的`喜好`只能是一个，篮球或者音乐或者游泳。而不能是既有篮球又有音乐或者游泳，或都有。
3. 必须有一个Primary Key

#### Second Norm Form
1. 符合1st Norm Form
2. 所有非primary Key的Column依赖于Primary Key列

#### Third Norm Form
1. 符合2nd Norm Form
2. 所有非Primary Key只依赖于Primary Key

### Relationships