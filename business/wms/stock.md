# stock 服务详解

## updateStockInventoryAndErpStock 库存占用

​	这里写的是虚拟组套占用

```mysql
# 零售店
## 零售店预售
update stock_erp_inventory 
	  set alloc_qty = alloc_qty + #{allocNum} 
	  where sku_id = #{skuId}
  	   and ware_id = #{wareId}
  	   and inventory_type = 1
update stock_inventory set order_book_num = order_book_num + #{allocNum} 
	where sku_id = #{skuId} 
	and ware_id = #{wareId}
## 1312 虚拟组套成品占用-正常销售
	-- 这里是减
	update stock_inventory set order_book_num = order_book_num + #{allocNum} 
	where sku_id = #{skuId} 
	and ware_id = #{wareId} 
	
## 虚拟组套子商品-1313- 预售 虚拟组套子商品
	update stock_erp_inventory 
	  set alloc_qty = alloc_qty + #{allocNum} 
	  where sku_id = #{skuId}
  	   and ware_id = #{wareId}
  	   and inventory_type = 1
update stock_inventory set order_book_num = order_book_num + #{allocNum} 
	where sku_id = #{skuId} 
	and ware_id = #{wareId}、
## 
	update stock_erp_inventory 
	  set alloc_qty = alloc_qty + #{allocNum} 
	  where sku_id = #{skuId}
  	   and ware_id = #{wareId}
  	   and inventory_type = 1
#非零售店 但是堂食

 
```

