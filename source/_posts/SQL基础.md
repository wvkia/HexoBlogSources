---
title: SQL基础
date: 2018-04-09 23:09:42
tags: [SQL]
categories: Sql基础
---
#### 第一章 数据库和Sql
##### （一）Sql语句及种类
1. DDL（Data Definition Language）数据定义语言
> 用来创建或删除存储数据用的数据库以及数据表等对象
> 1. Create 创建语句
>2. Drop 删除语句
>3. Alter 修改语句

2. DML（Data Manipulation Language）数据操作语言
> 用来查询或变更表格记录的语句
> 1. Select 查询数据
> 2. Insert 插入语句
> 3. Update 语句
> 4. Delete 删除语句

3. DCL（Data Control Language）数据控制语句
> 用来确认或者取消变更的语句，或者设定权限
> 1. Commit 确认数据变更
> 2. RollBack 回滚
> 3. Grant 赋权
> 4. Revoke 取消权限

##### （二）建表
> Create database <'database'>

> Create table <'tablename'>(
><'column'> 数据类型 约束,
>
>)

> Drop table <'tablename'>

表定义的更新
> 
>> alter table tablename add column <'列定义'>
>> alter tbale <'tablename'> drop column <'列名'>
>> 


----------


#### 第二章 数据查询
##### (一)Select语句

1. 列查询
> select <'columnname'> ,... from <'tablename'>
>
> select * from <'tablename'>
>
2. As 设置别名
> select col **as** name1 from <'tablename'>
>
3. Distinct 删除重复行
> select distinct <'column'>,... from <'tablename'>
> Distinct 只能用于第一个列名之前
4. Where 条件表达式
> select columnname from tablename where "条件子句"

##### （二）运算符
1. 算术
> \+  \-  *  / 
> 注意 NULL和任何算术运算都是NULL

2. 比较
> \>  <  >=  <=  <>  is null
>

3. 逻辑
> NOT  AND OR

--------

#### 第三章 聚合和排序
##### （一）聚合函数
> COUNT
> SUM
> AVG
> MAX
> MIN 
> 注意Count(*)会计算加上NULL的行数，但其他函数包括Count(列名)都会得到NULL之外的数据然后计算

> 使用聚合函数也可以使用 DISTINCT进行过滤重复， select count( distinct 列名 )

> 聚合函数使用的要素
>> 常数
>> 列名
>> 其他聚合函数

##### （二）分组Group
    Select 列名，... from 表名 Group by 列名，...
> 注意Group by子句的含义就是分割，分组，在group by 中指定的列称为 分组列，将数据按照Group by分组之后，在进行计算，NULL也会被作为特定的一组数据。
>
> 因为采用列进行分组，所以每一行数据都是一组

    select 列名，... from 表名 where 条件子句 group by 列名...
> 分组和where查询结合
> 注意执行的顺序    From --> Where --> Group by --> select

##### 需要注意的常见错误

1. 在Select子句中书写多余的列
> 通过某个聚合键将表分组之后，结果中的一行数据就代表一组
> 此时，Select子句中只能存在
>> 常数
>> 聚合函数
>> Group By指定的列名，也就是聚合键

2. 在Group By语句中使用别名
> Group By语句不能使用As别名，因为sql执行顺序是先是From从表，然后根据Where过滤，然后通过Group > By聚合键进行分组，最后通过Select语句进行列的获取，所有As列名在最后

3. Group By的结果是不排序的，完全随机的

4. 在Where中使用聚合函数
> 例如 Select id,count(\*) from table where count(*) >=2 group by id;
> 但是这样会发生错误，只有Select语句和Having子句以及Order by可以使用聚合函数，如果想对分组之后的数据进行聚合函数的过滤，可以通过Having子句

#### （三）Having子句
    对分组后的结果进行条件，Select 列名，... From table Group By 列名，... Having 分组结果对应的条件
> Having子句使用的要素
>> 常数
>> 聚合函数
>> Group By指定的列，即聚合键

Having子句和Where条件
Where子句 = 指定行所对应的条件
Having子句 = 指定组所对应的条件
尽可能的先在Where子句中过滤掉条件：
> 1. 在使用Count聚合函数等对表的数据进行聚合操作时，DMBMS会先对数据进行排序处理，而数据量大的话排序会增加服务器负担，通过Where子句过滤后，行数会减少，但Having子句是在数据拿到之后即排序之后才对数据进行分组，因此尽可能使用Where过滤
> 2. 可以对Where所在列进行加索引，可以很大加快速度

#### （四）Order By排序
默认升序ASC，DESC降序
可以指定多个排序键
NULL值会在开头或者结尾进行特殊处理
排序键可以使用别名As
执行顺序：
> From --> Where --> Group by --> Having --> Select --> Order by








