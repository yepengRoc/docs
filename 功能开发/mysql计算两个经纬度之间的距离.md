数据库中经纬度是用逗号分隔的，经度在前面，纬度在后面



计算的时候需要使用同类型的经纬度，不能起点是百度类型经纬度，终点是高德类型经纬度



计算公式来源：

https://blog.csdn.net/qq_25175063/article/details/78435954 



计算方式：

https://www.cnblogs.com/endv/p/11986741.html 



个人汇总出来的sql:

-- 方正国际大厦 纬度39.99059  经度116.31970
SELECT order_id,ROUND(6378.138 * 2 * ASIN(SQRT(
                POW(
                    SIN(
                        (39.99059 * PI() / 180 - substring_index(prof_c,',', -1) * PI()/180) / 2
                    ),
                    2
                ) + COS(39.99059 * PI() / 180) * COS(substring_index(prof_c,',', -1) * PI() / 180) * POW(
                    SIN(
                        (116.319701* PI() / 180 - substring_index(prof_c,',', 1) * PI() / 180) / 2
                    ),
                    2
                )
            )
        ) * 1000
    ) AS distance
FROM oms_order_profiles where prof_type = 'LON_LAT' and prof_i = 2 limit 100;

-- 116.344236,39.998751 2.3 2281   116.322224,40.047221 6.3  6308
SELECT ROUND(6378.138 * 2 * ASIN(SQRT(
                POW(
                    SIN(
                        (39.99059 * PI() / 180 - 39.998751 * PI()/180) / 2
                    ),
                    2
                ) + COS(39.99059 * PI() / 180) * COS(39.998751 * PI() / 180) * POW(
                    SIN(
                        (116.319701* PI() / 180 - 116.344236 * PI() / 180) / 2
                    ),
                    2
                )
            )
        ) * 1000
    ) AS distance









```mysql


-- 方正国际大厦 纬度39.99059  经度116.31970  116.477675,39.995602
(SELECT b.root_parent_id,sum(b.total_price),ROUND(6378.138 * 2 * ASIN(SQRT(
                POW(SIN((39.995602 * PI() / 180 - substring_index(prof_c,',', -1) * PI()/180) / 2),2) 
								+ COS(39.995602 * PI() / 180) * COS(substring_index(prof_c,',', -1) * PI() / 180) * POW(
                    SIN((116.477675* PI() / 180 - substring_index(prof_c,',', 1) * PI() / 180) / 2),2))
        ) * 1000
    ) AS distance
FROM oms_order_profiles a 
inner join oms_order_main b on a.order_id = b.order_id 
 where a.prof_type = 'LON_LAT' and a.prof_i = 2   
and a.create_date > '2020-03-01 00:00:00' and a.create_date < '2020-04-01 00:00:00'
and b.province='北京'
GROUP BY b.root_parent_id HAVING distance <= 6000)
 UNION(
		SELECT b.root_parent_id,sum(b.total_price),ROUND(6378.138 * 2 * ASIN(SQRT(
                POW(
                    SIN((39.989325 * PI() / 180 - substring_index(prof_c,',', -1) * PI()/180) / 2),2)
									+ COS(39.989325 * PI() / 180) * COS(substring_index(prof_c,',', -1) * PI() / 180) * POW(
                    SIN((116.470698* PI() / 180 - substring_index(prof_c,',', 1) * PI() / 180) / 2),2))
        ) * 1000
    ) AS distance
FROM oms_order_profiles a 
inner join oms_order_main b on a.order_id = b.order_id 
where a.prof_type = 'LON_LAT' and a.prof_i in(1,3) 
and a.create_date > '2020-03-01 00:00:00' and a.create_date < '2020-04-01 00:00:00'
and b.province='北京'
GROUP BY b.root_parent_id HAVING distance <= 6000 
);

select * from oms_order_main where create_date > '2020-03-01 00:00:00' and create_date < '2020-04-01 00:00:00' and root_parent_id is null;


select root_parent_id from oms_order_main where create_date > '2020-01-01 00:00:00' GROUP BY root_parent_id HAVING count(order_id) > 1 ;
select * from oms_order_profiles limit 1;

-- 116.344236,39.998751 2.3 2281   116.322224,40.047221 6.3  6308
SELECT ROUND(6378.138 * 2 * ASIN(SQRT(
                POW(
                    SIN(
                        (39.99059 * PI() / 180 - 39.998751 * PI()/180) / 2
                    ),
                    2
                ) + COS(39.99059 * PI() / 180) * COS(39.998751 * PI() / 180) * POW(
                    SIN(
                        (116.319701* PI() / 180 - 116.344236 * PI() / 180) / 2
                    ),
                    2
                )
            )
        ) * 1000
    ) AS distance
FROM
    oms_order_ limit  1;
```



```mysql
-- 把3月份的数据 创建一个表
select max(COALESCE(d.prof_c,c.prof_c,cast(a.root_parent_id as TEXT))) as rootParentOrderId,a.root_parent_id,b.prof_c,b.prof_i,
a.order_source,sum(a.total_price) as total_price
into  backend.temp_table_three_month
from oms_order_main a 
inner join oms_order_profiles b on a.order_id = b.order_id and b.prof_type = 'LON_LAT' and b.create_date > '2020-03-01 00:00:00'
LEFT OUTER join oms_order_profiles c on a.order_id = c.order_id and c.prof_type='ORDER_ID' and c.prof_cat = '3RD'
 and c.create_date > '2020-03-01 00:00:00' and c.prof_c != '-1'
LEFT OUTER JOIN oms_order_profiles d on a.order_id = d.order_id and d.prof_type = 'THIRD_PARENT_ID' and d.prof_c != '-1'
and d.prof_cat = '3RD' and d.create_date > '2020-03-01 00:00:00'
where a.create_date > '2020-03-01 00:00:00' and a.create_date < '2020-04-01 00:00:00'
and a.root_parent_id is not null
-- and a.order_id = 164398871
GROUP BY a.root_parent_id,b.prof_c,b.prof_i,a.order_source;
```

