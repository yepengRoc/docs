匿名函数和闭包 的体现

- java中无法把一个函数作为参数进行传递，返回结果也无法是一个函数


- 参数和返回只能是基本变量和实例化的对象

函数式接口：
1如果一个接口只有一个抽象方法，则是
2如果某个接口上声明了FunctionalInterface注解，则是
3如果接口只有一个抽象方法，但是没有声明FunctionalInterface,依旧是.编译器


/**

此注解来声明一个函数式接口。

 函数式接口必须要有一个抽象方法，
 如果接种中定义的抽象方法和Object类中的方法名相同，则不能认为是一个
 函数式接口，因为对象或多或少都会继承Object。接口的抽象方法不会加1

 lambda表达式必须是方法引用或构造函数引用
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}

例如：
//函数时接口
interface 接口名{
    void test();
}
//非函数式接口
interface 接口名{
    void test();
    void toString();
}
//函数式接口
interface 接口名{
    void test();
    String toString();//因为这里重写来Object类中的方法
}  

高阶函数：如果一个函数接收一个函数作为参数，或者返回一个函数作为返回值，那么该函数就叫做高阶函数

接口中也可以定义方法，方法用default修饰。默认方法，保证和老版本代码兼容。实现类默认拥有
接口中的默认方法

java中lambda是一个对象，其它语言中是函数,称为函数式接口.

函数式接口必须有一个上下文。
MyInterface my = () -> System.out.println("123"); 必须这样（）->{} 才能存在，MyInterface就是上下文，用来提供类型推断。







lambda表达式的作用

- 为java添加了缺失带的函数式编程特性，使我们将函数当做一等公民看待
- 在将函数作为一等公民的语言中，Lambda表达式的类型是函数。在java中，Lambda表达式是对象，必须依附于一类特别的对象类型-函数式接口（function interface）



java Lambda表达式是一种匿名函数；它是没有声明的方式，即没有访问修饰符、返回值声明和名字



### java中lambda表达式的基本语法

- （argument） -> (body)

  例如：（arg1,arg2...）-> {body}

  (type1 arg1,arg2...) -> {body}

lambda示例：

- （int a,int b） -> {return a + b;}
- () -> {syso("h w")}
- (String s) -> {syso(s)}
- () -> 30
- () - > {retrun 30;}

### lambda结构

- 一个lambda表达式可以有零个或多个参数

- 参数类型既可以明确声明，也可以根据上下文推断、例如（int a）与（b）效果相同

- 所有参数必须包含在圆括号内，参数之间用逗号隔开 例如：（a,b）或（int a,int b）或（String a,int b,float c）

- 空圆括号代表参数集为空 例如（）-> 30

- 当只有一个参数，且其类型可以推导时，圆括号（）可省略。例如 a -> return a*a

- lambda表达式的主体可包含零条或多条语句

- 如果lambda表达式只有一条语句，花括号{}可以省略。匿名函数的返回类型与该主体表达式一致;多条语句必须用{}包裹的代码块，匿名函数的返回类型和代码块的返回类型一致，没有则返回空


### lambda表达式的作用

- 传递行为，而不仅仅是值
- 提升抽象层次
- api重用性更好
- 更加灵活

### 函数式接口

- 函数式接口是只包含了一个抽象方法声明的接口
- Runnable接口就是一个函数式接口，只包含了一个含有run的抽象方法
- 每个lambda都能隐式地赋值给函数式接口
- 