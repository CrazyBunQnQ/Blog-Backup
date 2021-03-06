---
title: MySQL 常用 SQL
date: 2018-09-22 22:22:22
categories: 数据库
tags:
- MySQL
---

记录一些不经常用但是需要的时候很有用的查询语句，不定期更新

<!-- more -->

## 查询信息

### 查询库名

```mysql
select database();
```
## 常用操作

### 时间戳

MySQL 中没有产生微秒的函数，now() 只能精确到秒，也没有存储带有毫秒、微秒的日期时间类型。但是却有提取微妙的函数...

```mysql
# 显示当前时间戳(秒)
select unix_timestamp(now());
# 提取时间中的微秒 123456
select microsecond('12:00:00.123456');
```

### 类型转换及字符串拼接

- 类型转换：`CAST(value AS TYPE)`

    ```mysql
    # 数字转字符串
    cast(1 as char);
    ```
- 字符串拼接：`CONCAT(str1, str2, str3...)`

示例：两个数字拼接成分数

```mysql
SELECT CONCAT(CAST(1 AS CHAR), '/', CAST(2 AS CHAR)) AS RATIO FROM DUAL;
```

结果：![](http://wx2.sinaimg.cn/large/a6e9cb00ly1fvez03wzojj204u028mxc.jpg)

### 根据条件显示

```mysql
select if(condition, a, b);
```

- 如果 `condition` 为 `true` 则返回 `a`, 否则返回 `b`
- `if()` 返回一个数字或字符串值

### 判断空值

- 如果 `value1` 为空则返回 `value2`, 否则返回 `value1`

    ```mysql
    select ifnull(value1,value2);
    ```
- 如果 `value1` 为空返回 `value2`，否则返回 `value3`

    ```mysql
    select if(isnull(value1), value2, value3);

### 重复记录

- 删除重复记录: 根据 t 表中的 a, b 字段删除 t 表中多余的重复记录

    ```mysql
    delete from t where id not in (select id from
        (select min(id) as id from t group by a, b) as c);
    ```
    >必须要有 `select id from (...) as C`
- 查找重复记录: 根据 t 表中的 a, b 字段查找 t 表中多余的重复记录(不包含 rowid 最小的记录)
    - 方法一：子查询 (较快)


        ```mysql
        select * from 
          (select a, b, count(a) as numa, count(b) as numb from t group by a, b) as e
        where numa > 1 and numb > 1;
        ```
    - 方法二：`group by` 和 `having` (较慢)


        ```mysql
        select * from t group by a, b having count(a) > 1 and count(b) > 1;
        ```

### 同比上升的记录

从 `t` 表中查询所有与前 n 天相比，`a` 字段数据有所提高的数据。

```mysql
select t_cur.* from
    t t_cur, t t_old
  where
    to_days(t_cur.date) - to_days(t_old.date) = n
  and t_cur.a > t_old.a;
```

>`date` 是日期

[例题：给定一个 `Weather` 表，编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 Id。](https://leetcode-cn.com/problems/rising-temperature/)

### 交换相邻数据

- 交换 `id` 为连续整数的 `t` 表中两条相邻数据的 `id`
    - 方法一：自定义变量


        ```mysql
        select @num:=case
          when t.id%2=1 and t.id=tmp.id then t.id
          when t.id%2=1 then (t.id+1)
          when t.id%2=0 then (t.id-1)
          else 0 end id, t.other
        from t,
           (select max(id) id from t) tmp
        order by id;
        ```
    - 方法二：`union`

        ```mysql
        select ((id + 1) div 2 * 2 - mod(id + 1, 2)) id, other
        from t
        where ((id + 1) div 2) * 2 <= (select count(1) from t)
        union
        select *
        from t
        where ((id + 1) div 2) * 2 > (select count(1) from t)
        order by id;
        ```
- 交换指定字段的数据


    ```mysql
    ```

## 函数

```mysql
CREATE FUNCTION 函数名(参数 参数数据类型) RETURNS 返回数据类型
BEGIN
# 自定义变量
Declare 变量 数据类型;
# 赋值
Set 变量 = 参数 - 1;
  RETURN (
      # 查询语句，可以使用前面设置的变量或参数，但不能在此进行运算
      select DISTINCT Salary from Employee order by Salary DESC limit 变量, 1
  );
END
```

>在查询语句中不能直接写数学运算必须通过赋值来进行运算

### 数据类型

### 常用函数

#### 查询第 N 高的薪水

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
Declare M INT;
SET M = N - 1;
  RETURN (
      select distinct Salary from Employee order by Salary desc limit M, 1
  );
END
```

#### 分数排名

如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有 “间隔”。

```mysql
select Score,
       cast(case
         when @prevRank = Score then @curRank
         when @prevRank := Score then @curRank := @curRank + 1
         else @curRank := @curRank + 1
         end as signed) as Rank
from (select Score from Scores order by Score desc) p,
  (select @curRank :=0, @prevRank := null) r;
```

## 常见问题

### `limit m, n` 与 `limit m offset n` 区别

```mysql
select * from table limit m,n;                 
```
`limit m, n` 含义是跳过 m 条取出 n 条数据，limit 后面是从第 m 条开始读，读取 n 条信息，即从第 m + 1 条开始读取 n 条数据

```mysql
select * from table limit m offset n;
```
`limit m offset n` 含义是从第 n 条（不包括）数据开始取出 m 条数据，limit 后面跟的是 m 条数据，offset 后面是从第 n 条开始读取，即从 n+1 条开始读取 m 条数据