```java
/**
 信息性注释类型，用于指示接口类型声明旨在成为Java语言规范定义的功能接口。
 从概念上讲，功能接口仅具有一种抽象方法。由于默认方法具有实现，因此它们不是抽象的。如果接口声明的抽象方法覆盖了java.lang.Object的公共方法之一，则该方法也不会计入接口的抽象方法计数，因为该接口的任何实现都将具有java.lang.Object或其他地方的实现。
 （这里表达的意思是如果接口中的方法定义和object中的方法一致，则不能算是函数式方法）
 
 	请注意，可以使用lambda表达式，方法引用或构造函数引用来创建功能接口的实例。
 	如果使用此注释类型对类型进行注释，则编译器需要生成错误消息，除非：
		1该类型是接口类型，而不是注释类型，枚举或类。
    2带注释的类型满足功能接口的要求。
  
但是，编译器会将满足功能接口定义的任何接口视为功能接口，而不管接口声明中是否存在FunctionalInterface批注。
 *
 * @jls 4.3.2. The Class Object
 * @jls 9.8 Functional Interfaces
 * @jls 9.4.3 Interface Method Body
 * @since 1.8
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```

