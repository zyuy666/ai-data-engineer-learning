# leetcode SQL 刷题笔记

## 第一天总结

- 刷题数量：20道
- 主要考点：查询，连接，聚合
- 易错点：null值处理，日期函数  
## 一、MySQL 和Oracle 语法差异速查

| 场景                                    | MySQL                                                        | Oracle                                                       | 我错过的点                                                   |
| --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 字符串拼接                              | concat('a','b')                                              | 'a'\|\|'b'                                                   | qwe                                                          |
| 空值处理                                | if(field is null,true value,false value)                     | nvl()                                                        | nvl()错用在了MySQL中                                         |
| 日期+1                                  | date_add(date, interval n/-n day)                            | date + 1                                                     | MySQL date类型直接-1会有特殊情况 2000/01/01 -1 = 2000/01/00  |
| 日期格式化                              | date_format(date,'%Y-%m-%d %H:%i:%s')                        | TO_CHAR(created_at, 'YYYY-MM-DD HH24:MI:SS')                 |                                                              |
| 聚合函数，如果符合某条件就count 或者sum | AVG(order_date = customer_pref_delivery_date)                | AVG(DECODE(order_date, customer_pref_delivery_date, 1, 0))   | sum(if(order_date = customer_pref_delivery_date,1,0)，这种比较慢 |
| 聚合函数里可以distinct                  | count(disitnct empno)                                        | 12C文档后记录了这种函数count(disitnct empno)                 |                                                              |
| length区别（UTF8）                      | CHAR_LENGTH('你好') = 2<br/>字符长度<br/>LENGTH('你好') = 6<br/>字节长度 | LENGTH('你好') = 2<br/>字符长度<br/>LENGTHB('你好') = 6<br/>字节长度 |                                                              |
| cross join (笛卡尔积)                   | 标准sql ：from a cross on b  后无需接On                      | 标准sql                                                      | 可以不写from a join b on 1=1                                 |
| 部分/总数                               | avg(if(c.action = 'confirmed',1,0)) 用avg做                  |                                                              | count(case when lower(b.action) = 'confirmed' then 1 else null end)/count(b.user_id)<br/>繁琐 |
| ifnull                                  | IFNULL(SUM(units * price) / SUM(units), 0)                   | nvl()                                                        | case when count(b.product_id) = 0 then 0 else round(sum(a.price*b.units)/sum(b.units),2) end <br/>繁琐 |
| with t as                               | 记得使用 mysql8.0以上                                        |                                                              |                                                              |
|                                         |                                                              |                                                              |                                                              |

## 错题本（重点）

### 题目：197.上升的温度

表： `Weather`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| recordDate    | date    |
| temperature   | int     |
+---------------+---------+
id 是该表具有唯一值的列。
没有具有相同 recordDate 的不同行。
该表包含特定日期的温度信息
```

 

编写解决方案，找出与之前（昨天的）日期相比温度更高的所有日期的 `id` 。

返回结果 **无顺序要求** 。

结果格式如下例子所示。

 

**示例 1：**

```
输入：
Weather 表：
+----+------------+-------------+
| id | recordDate | Temperature |
+----+------------+-------------+
| 1  | 2015-01-01 | 10          |
| 2  | 2015-01-02 | 25          |
| 3  | 2015-01-03 | 20          |
| 4  | 2015-01-04 | 30          |
+----+------------+-------------+
输出：
+----+
| id |
+----+
| 2  |
| 4  |
+----+
解释：
2015-01-02 的温度比前一天高（10 -> 25）
2015-01-04 的温度比前一天高（20 -> 30）
```

**我的错误写法**

```select t.id from weather t left join weather y on (t.id - 1) = y.id where t.temperature > y.temperature```

-- 默认了随着id增长，recorddate也增长，没有考虑到id增长，recorddate减小的情况。

**正确写法**

```select t.id from (select id,temperature,date_add(recorddate,interval -1 day) ls from weather) t left join weather y on t.ls = y.recorddate where t.temperature > y.temperature```

### 题目：1581.进店却未进行过交易的顾客

表：`Visits`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| visit_id    | int     |
| customer_id | int     |
+-------------+---------+
visit_id 是该表中具有唯一值的列。
该表包含有关光临过购物中心的顾客的信息。
```

 

表：`Transactions`

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| transaction_id | int     |
| visit_id       | int     |
| amount         | int     |
+----------------+---------+
transaction_id 是该表中具有唯一值的列。
此表包含 visit_id 期间进行的交易的信息。
```

 

有一些顾客可能光顾了购物中心但没有进行交易。请你编写一个解决方案，来查找这些顾客的 ID ，以及他们只光顾不交易的次数。

返回以 **任何顺序** 排序的结果表。

返回结果格式如下例所示。

 

**示例 1：**

```
输入:
Visits
+----------+-------------+
| visit_id | customer_id |
+----------+-------------+
| 1        | 23          |
| 2        | 9           |
| 4        | 30          |
| 5        | 54          |
| 6        | 96          |
| 7        | 54          |
| 8        | 54          |
+----------+-------------+
Transactions
+----------------+----------+--------+
| transaction_id | visit_id | amount |
+----------------+----------+--------+
| 2              | 5        | 310    |
| 3              | 5        | 300    |
| 9              | 5        | 200    |
| 12             | 1        | 910    |
| 13             | 2        | 970    |
+----------------+----------+--------+
输出:
+-------------+----------------+
| customer_id | count_no_trans |
+-------------+----------------+
| 54          | 2              |
| 30          | 1              |
| 96          | 1              |
+-------------+----------------+
解释:
ID = 23 的顾客曾经逛过一次购物中心，并在 ID = 12 的访问期间进行了一笔交易。
ID = 9 的顾客曾经逛过一次购物中心，并在 ID = 13 的访问期间进行了一笔交易。
ID = 30 的顾客曾经去过购物中心，并且没有进行任何交易。
ID = 54 的顾客三度造访了购物中心。在 2 次访问中，他们没有进行任何交易，在 1 次访问中，他们进行了 3 次交易。
ID = 96 的顾客曾经去过购物中心，并且没有进行任何交易。
如我们所见，ID 为 30 和 96 的顾客一次没有进行任何交易就去了购物中心。顾客 54 也两次访问了购物中心并且没有进行任何交易。
```

**我的错误写法**

```select v.customer_id,count(v.visit_id) count_no_trans  from visits v  left join transactions t   on v.visit_id = t.visit_id  where t.amount is null  group by v.customer_id```

-- 没有考虑到amount可能有空值的情况

**正确写法**

```select v.customer_id,count(v.visit_id) count_no_trans  from visits v  left join transactions t   on v.visit_id = t.visit_id  where t.visit_id is null  group by v.customer_id```

### 题目：1280.学生们参加各科测试的次数

学生表: `Students`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| student_name  | varchar |
+---------------+---------+
在 SQL 中，主键为 student_id（学生ID）。
该表内的每一行都记录有学校一名学生的信息。
```

 

科目表: `Subjects`

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| subject_name | varchar |
+--------------+---------+
在 SQL 中，主键为 subject_name（科目名称）。
每一行记录学校的一门科目名称。
```

 

考试表: `Examinations`

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| student_id   | int     |
| subject_name | varchar |
+--------------+---------+
这个表可能包含重复数据（换句话说，在 SQL 中，这个表没有主键）。
学生表里的一个学生修读科目表里的每一门科目。
这张考试表的每一行记录就表示学生表里的某个学生参加了一次科目表里某门科目的测试。
```

 

查询出每个学生参加每一门科目测试的次数，结果按 `student_id` 和 `subject_name` 排序。

查询结构格式如下所示。

 

**示例 1：**

```
输入：
Students table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |
+------------+--------------+
Subjects table:
+--------------+
| subject_name |
+--------------+
| Math         |
| Physics      |
| Programming  |
+--------------+
Examinations table:
+------------+--------------+
| student_id | subject_name |
+------------+--------------+
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |
+------------+--------------+
输出：
+------------+--------------+--------------+----------------+
| student_id | student_name | subject_name | attended_exams |
+------------+--------------+--------------+----------------+
| 1          | Alice        | Math         | 3              |
| 1          | Alice        | Physics      | 2              |
| 1          | Alice        | Programming  | 1              |
| 2          | Bob          | Math         | 1              |
| 2          | Bob          | Physics      | 0              |
| 2          | Bob          | Programming  | 1              |
| 6          | Alex         | Math         | 0              |
| 6          | Alex         | Physics      | 0              |
| 6          | Alex         | Programming  | 0              |
| 13         | John         | Math         | 1              |
| 13         | John         | Physics      | 1              |
| 13         | John         | Programming  | 1              |
+------------+--------------+--------------+----------------+
解释：
结果表需包含所有学生和所有科目（即便测试次数为0）：
Alice 参加了 3 次数学测试, 2 次物理测试，以及 1 次编程测试；
Bob 参加了 1 次数学测试, 1 次编程测试，没有参加物理测试；
Alex 啥测试都没参加；
John  参加了数学、物理、编程测试各 1 次。
```

**我的错误写法**

```select s.student_id,s.student_name,b.subject_name,count(1) attended_exams from students s join subjects b  on 1 =1 left join examinations e  on s.student_id = e.student_id and b.subject_name = e.subject_name group by s.student_id,b.subject_name```

-- 没注意到Examinations table中的数据是重复的，导致关联后结果发散

**正确写法**

```select s.student_id,s.student_name,b.subject_name,sum(ifnull(e.cnt,0)) attended_exams from students s join subjects b  on 1 =1 left join (select student_id,subject_name,count(1) cnt from examinations group by student_id,subject_name) e  on s.student_id = e.student_id and b.subject_name = e.subject_name group by s.student_id,b.subject_name order by s.student_id,b.subject_name```

### 题目：570.至少有5名直接下属的经理

表: `Employee`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| department  | varchar |
| managerId   | int     |
+-------------+---------+
id 是此表的主键（具有唯一值的列）。
该表的每一行表示雇员的名字、他们的部门和他们的经理的id。
如果managerId为空，则该员工没有经理。
没有员工会成为自己的管理者。
```

 

编写一个解决方案，找出至少有**五个直接下属**的经理。

以 **任意顺序** 返回结果表。

查询结果格式如下所示。

 

**示例 1:**

```
输入: 
Employee 表:
+-----+-------+------------+-----------+
| id  | name  | department | managerId |
+-----+-------+------------+-----------+
| 101 | John  | A          | Null      |
| 102 | Dan   | A          | 101       |
| 103 | James | A          | 101       |
| 104 | Amy   | A          | 101       |
| 105 | Anne  | A          | 101       |
| 106 | Ron   | B          | 101       |
+-----+-------+------------+-----------+
输出: 
+------+
| name |
+------+
| John |
+------+
```

**我的错误写法**

```select m.name from employee m  join employee e  on m.id = e.managerid and (m.id != m.managerid or m.managerid is null)  group by m.name having count(e.managerid) >= 5```

-- 没有考虑到Employee中有同名但id不同的情况

**正确写法**

```select m.name from employee m  join employee e  on m.id = e.managerid and (m.id != m.managerid or m.managerid is null)  group by m.id,m.name having count(e.managerid) >= 5```

### 题目：1193.每月交易Ⅰ

表：`Transactions`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| country       | varchar |
| state         | enum    |
| amount        | int     |
| trans_date    | date    |
+---------------+---------+
id 是这个表的主键。
该表包含有关传入事务的信息。
state 列类型为 ["approved", "declined"] 之一。
```

 

编写一个 sql 查询来查找每个月和每个国家/地区的事务数及其总金额、已批准的事务数及其总金额。

以 **任意顺序** 返回结果表。

查询结果格式如下所示。

 

**示例 1:**

```
输入：
Transactions table:
+------+---------+----------+--------+------------+
| id   | country | state    | amount | trans_date |
+------+---------+----------+--------+------------+
| 121  | US      | approved | 1000   | 2018-12-18 |
| 122  | US      | declined | 2000   | 2018-12-19 |
| 123  | US      | approved | 2000   | 2019-01-01 |
| 124  | DE      | approved | 2000   | 2019-01-07 |
+------+---------+----------+--------+------------+
输出：
+----------+---------+-------------+----------------+--------------------+-----------------------+
| month    | country | trans_count | approved_count | trans_total_amount | approved_total_amount |
+----------+---------+-------------+----------------+--------------------+-----------------------+
| 2018-12  | US      | 2           | 1              | 3000               | 1000                  |
| 2019-01  | US      | 1           | 1              | 2000               | 2000                  |
| 2019-01  | DE      | 1           | 1              | 2000               | 2000                  |
+----------+---------+-------------+----------------+--------------------+-----------------------+
```

**我的错误写法**

```
select date_format(trans_date,'%Y-%m') month,country,count(1) trans_count ,count(if(lower(state)='approved',1,null)) approved_count,sum(amount ) trans_total_amount ,sum(if(lower(state)='approved',amount,null)) approved_total_amount 
from transactions
group by country,date_format(trans_date,'%Y-%m')
```

-- 没有考虑到sum(if(lower(state)='approved',amount,null)) 这里不允许的数据转为null，如果一个允许的都没有，sum出来的将会是null，需要转为0

**正确写法**

```
select date_format(trans_date,'%Y-%m') month,country,count(1) trans_count ,count(if(lower(state)='approved',1,null)) approved_count,sum(amount ) trans_total_amount ,sum(if(lower(state)='approved',amount,0)) approved_total_amount 
from transactions
group by country,date_format(trans_date,'%Y-%m')
```

