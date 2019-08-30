### 什么情况下会发生明明创建了索引，但是执行的时候并没有通过索引呢？ 

一条SQL语句的查询，可以有不同的执行方案，至于最终选择哪种方案，需要通过优化器进行选择，选择执行成本最低的方案。
在一条单表查询语句真正执行之前，MySQL的查询优化器会找出执行该语句所有可能使用的方案，对比之后找出成本最低的方案。
这个成本最低的方案就是所谓的执行计划。优化过程大致如下：
1、根据搜索条件，找出所有可能使用的索引 
2、计算全表扫描的代价 
3、计算使用不同索引执行查询的代价 
4、对比各种执行方案的代价，找出成本最低的那一个

### MySQL 5.6中，对索引做了哪些优化吗？ 

Index Condition Pushdown（索引下推）
MySQL 5.6引入了索引下推优化，默认开启，使用SET optimizer_switch = 'index_condition_pushdown=off';可以将其关闭。官方文档中给的例子和解释如下：
people表中（zipcode，lastname，firstname）构成一个索引
SELECT * FROM people WHERE zipcode='95054' AND lastname LIKE '%etrunia%' AND address LIKE '%Main Street%';
如果没有使用索引下推技术，则MySQL会通过zipcode='95054'从存储引擎中查询对应的数据，返回到MySQL服务端，然后MySQL服务端基于lastname LIKE '%etrunia%'和address LIKE '%Main Street%'来判断数据是否符合条件。
如果使用了索引下推技术，则MYSQL首先会返回符合zipcode='95054'的索引，然后根据lastname LIKE '%etrunia%'和address LIKE '%Main Street%'来判断索引是否符合条件。如果符合条件，则根据该索引来定位对应的数据，如果不符合，则直接reject掉。有了索引下推优化，可以在有like条件查询的情况下，减少回表次数。

## sql执行顺序

```sql
(8)SELECT (9) DISTINCT<select_list>
(1) FROM<left_table>
(3)<join_type>JOIN<right_table>
(2)ON<join_codition>
(4)WHERE<where_condition>
(5)GROUP BY<group_by_list>
(6)WITH{CUBE|ROLLUP}
(7)HAVING<having_condition>
(10)ORDER BY<order_by_list>
(11)LIMIT<limit_number>
```

