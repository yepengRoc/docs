## 视频29

driver是一个接口，但是没有指定实现。但是找到了对应的实现类。

肯定是有一种机制，

了解serviceloader类



 	一个简单的服务提供者加载的设施。

​	service 服务是接口和抽象类的集合，

Meta-info/services



## 视频30

serviceload由bootstrap加载，但是实现类由appclassloader加载

查看load方法，使用了线程上下文加载器。
懒加载对应的实现类。

因为设置为ext加载器后，迭代器不工作了。因为ext 不能加载classpath下的类。

添加TraceClassLoading进行查看。

--作业

## 视频31

//源码查看

Class.forName("com.mysql.jdbc.Driver");

DriverManager.getConnection("","","");



jdbc通过spi自动加载的，新的数据库连接不用classforname. 也可以类。



避免不同命名空间加载的相同的类

