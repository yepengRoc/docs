# 日期函数

## 计算两个日期相差天数

1、利用TO_DAYS函数

```mysql
select to_days(now()) - to_days('20120512')
```

2、利用DATEDIFF函数

```mysql
select datediff(now(),'20120512')
```

3、利用TIMESTAMPDIFF函数

计算两日期时间之间相差的天数，秒数，分钟数，周数，小时数，这里主要分享的是通过MySql内置的函数 TimeStampDiff() 实现。

函数 TimeStampDiff() 是MySQL本身提供的可以计算两个时间间隔的函数，语法为：

```mysql
TIMESTAMPDIFF(unit,datetime_expr1,datetime_expr2)
```

返回日期或日期时间表达式datetime_expr1 和datetime_expr2the 之间的整数差。其中unit单位有如下几种，分别是：FRAC_SECOND (microseconds), SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, QUARTER, YEAR 。该参数具体释义如下：

FRAC_SECOND   表示间隔是毫秒
SECOND   秒
MINUTE   分钟
HOUR   小时
DAY   天
WEEK   星期
MONTH   月
QUARTER   季度
YEAR   年

例如：

#计算两日期之间相差多少周

```mysql
select timestampdiff(week,'2011-09-30','2015-05-04');
```

#计算两日期之间相差多少天

select timestampdiff(day,'2011-09-30','2015-05-04');
另外计算两日期或时间之间相差多少天还可以使用 to_days 函数，但是该函数不用于阳历出现(1582)前的值，原因是当日历改变时，遗失的日期不会被考虑在内。因此对于1582 年之前的日期(或许在其它地区为下一年 ), 该函数的结果实不可靠的。具体用法如：

```mysql
to_days(end_time) - to_days(start_time);
```

#计算两日期/时间之间相差的秒数：

```mysql
select timestampdiff(SECOND,'2011-09-30','2015-05-04');
```


另外还可以使用 MySql 内置函数 UNIX_TIMESTAMP 实现，如下：

```msyql
SELECT　UNIX_TIMESTAMP(end_time) - UNIX_TIMESTAMP(start_time);　
```

#计算两日期/时间之间相差的时分数：　

```msyql
select timestampdiff(MINUTE,'2011-09-30','2015-05-04');
```


另外还可以如下实现：

```mysql
SELECT　SEC_TO_TIME(UNIX_TIMESTAMP(end_time) -　UNIX_TIMESTAMP(start_time));
```

[原文链接](https://blog.csdn.net/moshenglv/article/details/82527845)



## 日期加减天数

1MySQL 为日期增加一个时间间隔：date_add()

now()       //now函数为获取当前时间

```mysql
select date_add(now(), interval 1 day); - 加1天
select date_add(now(), interval 1 hour); -加1小时
select date_add(now(), interval 1 minute); - 加1分钟
select date_add(now(), interval 1 second); -加1秒
select date_add(now(), interval 1 microsecond);-加1毫秒
select date_add(now(), interval 1 week);-加1周
select date_add(now(), interval 1 month);-加1月
select date_add(now(), interval 1 quarter);-加1季
select date_add(now(), interval 1 year);-加1年
```

MySQL adddate(), addtime()函数，可以用date_add() 来替代。

2. MySQL 为日期减去一个时间间隔：date_sub()

MySQL date_sub() 日期时间函数 和date_add() 用法一致。

MySQL 中subdate(),subtime()函数，建议，用date_sub()来替代。
[原文链接](https://blog.csdn.net/asdkwq/article/details/77881850)