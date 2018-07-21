---
title:MySql数据库查询
---

# 数据库查询

## 条件

- 使用where子句对表中的数据筛选，结果为true的行会出现在结果集中
- 语法如下：

```sql
select * from 表名 where 条件;
```

#### 比较运算符

- 等于=
- 大于>
- 大于等于>=
- 小于<
- 小于等于<=
- 不等于!=或<>
- 查询编号大于3的学生

```sql
select * from students where id>3;
```

- 查询编号不大于4的科目

```sql
select * from subjects where id<=4;
```

- 查询姓名不是“黄蓉”的学生

```sql
select * from students where sname!='黄蓉';
```

- 查询没被删除的学生

```sql
select * from students where isdelete=0;
```

#### 逻辑运算符

- and
- or
- not
- 查询编号大于3的女同学

```sql
select * from students where id>3 and gender=0;
```

- 查询编号小于4或没被删除的学生

```sql
select * from students where id<4 or isdelete=0;
```

#### 模糊查询

- like
- %表示任意多个任意字符
- _表示一个任意字符
- 查询姓黄的学生

```sql
select * from students where sname like '黄%';
```

- 查询姓黄并且名字是一个字的学生

```sql
select * from students where sname like '黄_';
```

- 查询姓黄或叫靖的学生

```sql
select * from students where sname like '黄%' or sname like '%靖%';
```

#### 范围查询

- in表示在一个非连续的范围内
- 查询编号是1或3或8的学生

```sql
select * from students where id in(1,3,8);
```

- between ... and ...表示在一个连续的范围内
- 查询学生是3至8的学生

```sql
select * from students where id between 3 and 8;
```

- 查询学生是3至8的男生

```sql
select * from students where id between 3 and 8 and gender=1;
```

#### 空判断

- 注意：null与''是不同的
- 判空is null
- 查询没有填写地址的学生

```sql
select * from students where hometown is null;
```

- 判非空is not null
- 查询填写了地址的学生

```sql
select * from students where hometown is not null;
```

- 查询填写了地址的女生

```sql
select * from students where hometown is not null and gender=0;
```

#### 优先级

- 小括号，not，比较运算符，逻辑运算符
- and比or先运算，如果同时出现并希望先算or，需要结合()使用

## 聚合

- 为了快速得到统计数据，提供了5个聚合函数
- count(*)表示计算总行数，括号中写星与列名，结果是相同的
- 查询学生总数

```sql
select count(*) from students;
```

- max(列)表示求此列的最大值
- 查询女生的编号最大值

```sql
select max(id) from students where gender=0;
```

- min(列)表示求此列的最小值
- 查询未删除的学生最小编号

```sql
select min(id) from students where isdelete=0;
```

- sum(列)表示求此列的和
- 查询男生的编号之后

```sql
select sum(id) from students where gender=1;
```

- avg(列)表示求此列的平均值
- 查询未删除女生的编号平均值

```sql
select avg(id) from students where isdelete=0 and gender=0;
```

## 分组

- 按照字段分组，表示此字段相同的数据会被放到一个组中
- 分组后，只能查询出相同的数据列，对于有差异的数据列无法出现在结果集中
- 可以对分组后的数据进行统计，做聚合运算
- 语法：

```sql
select 列1,列2,聚合... from 表名 group by 列1,列2,列3...
```

- 查询男女生总数

```sql
select gender as 性别,count(*)
from students
group by gender;
```

- 查询各城市人数

```sql
select hometown as 家乡,count(*)
from students
group by hometown;
```

#### 分组后的数据筛选

- 语法：

```sql
select 列1,列2,聚合... from 表名
group by 列1,列2,列3...
having 列1,...聚合...
```

- having后面的条件运算符与where的相同
- 查询男生总人数

```sql
方案一
select count(*)
from students
where gender=1;
-----------------------------------
方案二：
select gender as 性别,count(*)
from students
group by gender
having gender=1;
```

#### 对比where与having

- where是对from后面指定的表进行数据筛选，属于对原始数据的筛选
- having是对group by的结果进行筛选

## 排序

- 为了方便查看数据，可以对数据进行排序
- 语法：

```sql
select * from 表名
order by 列1 asc|desc,列2 asc|desc,...
```

- 将行数据按照列1进行排序，如果某些行列1的值相同时，则按照列2排序，以此类推
- 默认按照列值从小到大排列
- asc从小到大排列，即升序
- desc从大到小排序，即降序
- 查询未删除男生学生信息，按学号降序

```sql
select * from students
where gender=1 and isdelete=0
order by id desc;
```

- 查询未删除科目信息，按名称升序

```sql
select * from subject
where isdelete=0
order by stitle;
```

## 获取部分行

- 当数据量过大时，在一页中查看数据是一件非常麻烦的事情
- 语法

```sql
select * from 表名
limit start,count
```

- 从start开始，获取count条数据
- start索引从0开始

#### 示例：分页

- 已知：每页显示m条数据，当前显示第n页
- 求总页数：此段逻辑后面会在python中实现
  - 查询总条数p1
  - 使用p1除以m得到p2
  - 如果整除则p2为总数页
  - 如果不整除则p2+1为总页数
- 求第n页的数据

```sql
select * from students
where isdelete=0
limit (n-1)*m,m
```

## 总结

- 完整的select语句

```sql
select distinct *
from 表名
where ....
group by ... having ...
order by ...
limit star,count
```

- 执行顺序为：
  - from 表名
  - where ....
  - group by ...
  - select distinct *
  - having ...
  - order by ...
  - limit star,count
- 实际使用中，只是语句中某些部分的组合，而不是全部