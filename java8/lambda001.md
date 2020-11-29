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







## lambda表达式的作用

- 为java添加了缺失带的函数式编程特性，使我们将函数当做一等公民看待
- 在将函数作为一等公民的语言中，Lambda表达式的类型是函数。在java中，Lambda表达式是对象，必须依附于一类特别的对象类型-函数式接口（function interface）

外部迭代

```java
List<Integer> numbers = Arrays.asList(1,2,3,4,5,6);
for(int number : numbers){
  syso(numbers);
}
```

内部迭代

```java
List<Integer> numbers = Arrays.asList(1,2,3,4,5,6);
numbers.forEach(new Comsumer<Integer>(){
  public void accept(Ingteger value){
    syso(value);
  }
});
```

迭代2

```java
List<Integer> numbers = Arrays.asList(1,2,3,4);
numbers.forEach(value -> syso(value));
```

迭代3

```java
List<Integer> numbers = Arrays.asList(1,2,3,4);
number.forEach(System.out::println);
```

## Lambda概要

java Lambda表达式是一种匿名函数；它是没有声明的方式，即没有访问修饰符、返回值声明和名字

## Lambda表达式作用

- 传递行为，而不仅仅是值
  - 提升抽象层次
  - API重用性更好
  - 更加灵活

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



方法引用： method reference，实际上是Lambda表达式的一种语法糖

可以把方法引用看做一个「函数指针」，function pointer

classname::staticmethod

classname.saticmethod

方法引用共分为4类：

1类名：：静态方法名

2引用名（对象名）：： 实例方法名

3类名：：实例方法名

4构造方法引用:     类名：：new



## 集合 流

List

MyList

Stream流



流由3部分构成：

1源

2零个或多个中间操作

3终止操作

流操作的分类

1惰性求值

2及早求值

stream.xxx().zzz().count()



- Collection 提供了新的stream()方法
- 流不存储值，通过管道的方式获取值
- 本质是函数式的，对流的操作会生成一个结果，不过并不会修改底层的数据源，集合可以作为流的底层数据源
- 延迟查找，很多流操作（过滤、映射、排序等）都可以延迟实现



Stream -> Source -> Transforming values -> Operations



描述性语言

select name from student where age > 20 and address ='beijing' order by age desc;

描述性语言：

student.stream().filter(student -> student.getAge() > 20 && student.getAddress().equal("beijing")).sorted((s1,s2) -> s2.getAge() - s1.getAge()).forEach(System.out::println);

中间操作都会返回一个Stream对象，比如返回Stream<Student>,Stream<String>,Stream<String>，终止操作则不会返回Stream类型，可能不返回值，也可能返回其它类型的单个值

内部迭代



外部迭代 -- 最古老的迭代方式

集合关注的是数据与数据存储本身；

流关注的是对数据的计算



流与迭代器类似的一点是：流是无法重复使用或消费的。



## Stream 操作类型

- Intrmediate:一个流可以后面跟随0个或多个intermediate操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用，这类操作都是延迟的（lazy）,就是说，仅仅调用到这类方法，并没有真正开始流的遍历
- Terminal:一个流只能有一个teriminal操作，当这个操作执行后，流就被用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal操作的执行，才会真正开始流的遍历，并且会生成一个结果