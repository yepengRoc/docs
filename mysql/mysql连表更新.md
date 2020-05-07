



把表order_collect_task_queue 中的字段shipment_spot_name的值更新为表order_header中字段shipment_spot_name的值

```mysql
UPDATE order_collect_task_queue a
 inner join order_header b on a.parent_order_id=b.order_id 
set a.shipment_spot_name= b.shipment_spot_name where a.carrier_Id=8000460 and a.order_status=2
and b.carrier_id=8000460 and LENGTH(IFNULL(a.shipment_spot_name,'')) < 1 
and LENGTH(IFNULL(b.shipment_spot_name,'')) > 1;

```

