日期：2019-07-31

## 枚举对象

```java
/**
 * 商品类型 Enum
 * @author wangbaocai
 */
public enum SkuTypeEnum {
	
	SKU_TYPE_COMMON(1001, "一般商品"),
	SKU_TYPE_GIFT_CARD(1002, "礼品卡"),
	SKU_TYPE_GIFT(1004, "赠品"),
	SKU_TYPE_WRAPPER(1003, "包装材料"),
	SKU_TYPE_VIRTUAL_GROUP(1008,"虚拟组套");
	
	private Integer skuTypeId;
	private String skuTypeName;
	
	private SkuTypeEnum(Integer skuTypeId, String skuTypeName){
		this.skuTypeId = skuTypeId;
		this.skuTypeName = skuTypeName;
	}

	public Integer getSkuTypeId() {
		return skuTypeId;
	}

	public void setSkuTypeId(Integer skuTypeId) {
		this.skuTypeId = skuTypeId;
	}

	public String getSkuTypeName() {
		return skuTypeName;
	}

	public void setSkuTypeName(String skuTypeName) {
		this.skuTypeName = skuTypeName;
	}
	
}
```

编译为class文件

## 对class文件进行反编译

下载jad，把SkuTypeEnum.class放入到jad的文件夹下，通过执行命令jad -sjava SkuTypeEnum.class生成反编译的源码文件

```java
public final class SkuTypeEnum extends Enum
{

    public static SkuTypeEnum[] values()
    {
        return (SkuTypeEnum[])$VALUES.clone();
    }

    public static SkuTypeEnum valueOf(String name)
    {
        return (SkuTypeEnum)Enum.valueOf(com/chunbo/wms/util/enu/SkuTypeEnum, name);
    }

    private SkuTypeEnum(String s, int i, Integer skuTypeId, String skuTypeName)
    {
        super(s, i);
        this.skuTypeId = skuTypeId;
        this.skuTypeName = skuTypeName;
    }

    public Integer getSkuTypeId()
    {
        return skuTypeId;
    }

    public void setSkuTypeId(Integer skuTypeId)
    {
        this.skuTypeId = skuTypeId;
    }

    public String getSkuTypeName()
    {
        return skuTypeName;
    }

    public void setSkuTypeName(String skuTypeName)
    {
        this.skuTypeName = skuTypeName;
    }

    public static final SkuTypeEnum SKU_TYPE_COMMON;
    public static final SkuTypeEnum SKU_TYPE_GIFT_CARD;
    public static final SkuTypeEnum SKU_TYPE_GIFT;
    public static final SkuTypeEnum SKU_TYPE_WRAPPER;
    public static final SkuTypeEnum SKU_TYPE_VIRTUAL_GROUP;
    private Integer skuTypeId;
    private String skuTypeName;
    private static final SkuTypeEnum $VALUES[];

    static 
    {
        SKU_TYPE_COMMON = new SkuTypeEnum("SKU_TYPE_COMMON", 0, Integer.valueOf(1001), "\u4E00\u822C\u5546\u54C1");
        SKU_TYPE_GIFT_CARD = new SkuTypeEnum("SKU_TYPE_GIFT_CARD", 1, Integer.valueOf(1002), "\u793C\u54C1\u5361");
        SKU_TYPE_GIFT = new SkuTypeEnum("SKU_TYPE_GIFT", 2, Integer.valueOf(1004), "\u8D60\u54C1");
        SKU_TYPE_WRAPPER = new SkuTypeEnum("SKU_TYPE_WRAPPER", 3, Integer.valueOf(1003), "\u5305\u88C5\u6750\u6599");
        SKU_TYPE_VIRTUAL_GROUP = new SkuTypeEnum("SKU_TYPE_VIRTUAL_GROUP", 4, Integer.valueOf(1008), "\u865A\u62DF\u7EC4\u5957");
        $VALUES = (new SkuTypeEnum[] {
            SKU_TYPE_COMMON, SKU_TYPE_GIFT_CARD, SKU_TYPE_GIFT, SKU_TYPE_WRAPPER, SKU_TYPE_VIRTUAL_GROUP
        });
    }
}
```



final class SkuTypeEnum  final修饰，不能进行继承

private SkuTypeEnum 构造函数私有，不能进行实例创建

通过静态代码块static{}进行枚举内对象的实例化

所以枚举一旦创建就不可变。可以用来创建单例对象

