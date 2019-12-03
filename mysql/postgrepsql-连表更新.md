业务需要在日商品销量上加上销售总价，当前记录，并无总价字段，故新加total_sale_price作为总价字段，然后把这个字段的值通过sql更新上。统计出10月份需要更新的sql有20w条，在库里执行的时候，一秒大概只能执行2到3条，表中的数据有2000多w,当然库中其它表的数据也挺大的。按照这个速度一分钟大概150条，一个小时才能更新9000条，一天才能把所有的数据更新上，太慢了。后想到通过连表更新

```sql
-- 首先创建临时表，记录需要更新的数据
create table backend.tt1 as 
select a.ware_id,a.sku_id,to_date(b.create_date,'yyyy-mm-dd') as date_flag,sum(a.sku_num*a.sale_price) as total_sale_price
    from oms_db.oms_order_detail a
    inner join oms_db.oms_order_main b
      on a.order_id = b.order_id
     WHERE is_payed = 1
        and  b.order_type in (1,43)
        and b.is_parent = 0
           and b.create_date >= '2019-10-01' 
and b.create_date < '2019-11-01'
  group by a.ware_id,a.sku_id,date_flag
order by date_flag;
-- 通过连表更新把临时表中的数据更新到业务表上
update dps_sku_sales_predict set total_sale_price = tt1.total_sale_price from tt1
 where dps_sku_sales_predict.sku_id=	tt1.sku_id and dps_sku_sales_predict.ware_id=tt1.ware_id	 and dps_sku_sales_predict.date_flag=tt1.date_flag ;

```

只用了几十秒就搞定了。