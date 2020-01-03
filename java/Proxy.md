

​	Proxy提供用于创建动态代理类和实例的静态方法，它还是由这些方法创建的所有动态代理类的超类.

​	创建接口Foo的代理：

```java
 InvocationHandler handler = new MyInvocationHandler(...);
      Class<?> proxyClass = Proxy.getProxyClass(Foo.class.getClassLoader(), Foo.class);
     Foo f = (Foo) proxyClass.getConstructor(InvocationHandler.class).
                      newInstance(handler);
```

或者更简单的方式：

```java
Foo f = (Foo) Proxy.newProxyInstance(Foo.class.getClassLoader(),
                                           new Class<?>[] { Foo.class },
                                           handler);
```

​	动态代理类是在运行时创建的实现了接口列表的类，具有以下行为：

​	1代理接口，由代理类实现的接口

​	2 代理实例是一个代理类实例

​	每一个代理实例有一个关联的实现了InvocationHandler接口的invocation handler对象，

​	通过其代理接口一个代理实例的方法调用将被分派到{InvocationHandler#invoke}实例的调用处理程序的方法，并传递代理实例，一个{java.lang.reflect.Method}对象识别被调用的方法，方法的参数包含一个Object数组。调用适当的调用处理程序处理所述经编码的方法和 结果，它会返回作为代理实例的方法调用的结果被返回。

​	一个代理类必须有以下属性：

1 如果所有的代理接口是public，则代理类是public,final,非abstract

2 如果所有的代理接口是not-public，则代理类是not-public,final,非abstract

3代理类的不合格名称未指定。 然而，以字符串`"$Proxy"`开头的类名空间应该保留给代理类。

4 一个代理类继承自`java.lang.reflect.Proxy` 。

5 代理类完全按照相同的顺序实现其创建时指定的接口

6如果一个代理类实现一个非公共接口，那么它将被定义在与该接口相同的包中。 否则，代理类的包也是未指定的。 请注意，程序包密封不会阻止在运行时在特定程序包中成功定义代理类，并且类也不会由同一类加载器定义，并且与特定签名者具有相同的包。

7 由于代理类实现了在其创建时指定的所有接口， `getInterfaces`在其`类`对象上调用`getInterfaces`将返回一个包含相同列表接口的数组（按其创建时指定的顺序），在其`类`对象上调用`getMethods`将返回一个数组的`方法`对象，其中包括这些接口中的所有方法，并调用`getMethod`将在代理接口中找到可以预期的方法。

8 [`Proxy.isProxyClass`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/reflect/Proxy.html#isProxyClass-java.lang.Class-)方法将返回true，如果它通过代理类 - 由`Proxy.getProxyClass`返回的类或由`Proxy.newProxyInstance`返回的对象的类 - 否则为false。

9 所述`java.security.ProtectionDomain`代理类的是相同由引导类装载程序装载系统类，如`java.lang.Object` ，因为是由受信任的系统代码生成代理类的代码。 此保护域通常将被授予`java.security.AllPermission` 

10 每个代理类有一个公共构造一个参数，该接口的实现[`InvocationHandler`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/reflect/InvocationHandler.html) ，设置调用处理程序的代理实例。 而不必使用反射API来访问公共构造函数，也可以通过调用[`Proxy.newProxyInstance`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/reflect/Proxy.html#newProxyInstance-java.lang.ClassLoader-java.lang.Class:A-java.lang.reflect.InvocationHandler-)方法来创建代理实例，该方法将调用[`Proxy.getProxyClass`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/reflect/Proxy.html#getProxyClass-java.lang.ClassLoader-java.lang.Class...-)的操作与调用处理程序一起调用构造函数。

代理实例具有以下属性：

- 给定代理实例`proxy`和其代理类`Foo` ，以下表达式将返回true：

  ```
     proxy instanceof Foo 
  ```

  并且以下演员操作将会成功（而不是投掷一个`ClassCastException` ）：

  ```
     (Foo) proxy 
  ```

- 每个代理实例都有一个关联的调用处理程序，它被传递给它的构造函数。 静态[`Proxy.getInvocationHandler`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/reflect/Proxy.html#getInvocationHandler-java.lang.Object-)方法将返回与作为其参数传递的代理实例关联的调用处理程序。

- 代理实例上的接口方法调用将被编码并分派到调用处理程序的[`invoke`](http://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/reflect/InvocationHandler.html#invoke-java.lang.Object-java.lang.reflect.Method-java.lang.Object:A-)方法，如该方法的文档所述。

- `hashCode` ， `equals`在代理实例上的`toString`中声明的`java.lang.Object`或`toString`或`toString`或`toString`方法将被编码并分派到调用处理程序的`invoke`方法，方法与接口方法调用被编码和调度相同。 传递给`invoke`的`方法`对象的声明类将为`java.lang.Object` 。 从`java.lang.Object`的代理实例的其他公共方法不会被代理类覆盖，因此这些方法的调用与`java.lang.Object` 。

  多代理接口中复制的方法

  1 当代理类的两个或多个接口包含具有相同名称和参数签名的方法时，代理类接口的顺序变得重要。 当在代理实例上调用这种*重复方法*时，传递给调用处理程序的`方法`对象不一定是其声明类可以通过调用代理方法的接口的引用类型进行分配的对象。 存在此限制，因为生成的代理类中的相应方法实现无法确定其调用的接口。 因此，当在代理实例上调用重复的方法时，代理类的`方法`列表中包含方法（直接或通过超级接口继承）的最重要的接口中的方法的Method对象被传递给调用处理程序的`invoke`方法，而不管方法调用发生的引用类型。

  2 如果代理接口包含具有相同的名称和参数签名的方法`hashCode` ， `equals` ，或`toString`的方法`java.lang.Object` ，当这种方法在代理实例调用时， `方法`传递到调用处理程序的对象将作为`java.lang.Object 其声明类， 换句话说， `java.lang.Object`的公共非最终方法`java.lang.Object`上先于所有代理接口，以确定哪个`方法`对象传递给调用处理程序。

  3 还要注意，当将一个重复的方法分派到调用处理程序时， `invoke`方法可能只会将可分配给*所有*可以调用的*所有*代理接口中的方法的`throws`子句中的一种异常类型的检查异常类型抛出通过。 如果`invoke`方法抛出经过检查的异常是不能分配给任何通过它可以通过调用代理接口中的方法声明的异常类型，那么选中`UndeclaredThrowableException`将通过代理实例调用抛出。 此限制意味着，并非所有通过调用返回的异常类型`getExceptionTypes`上`方法`传递给对象`invoke`方法一定可以成功地抛出`invoke`方法。

```java
public class Proxy implements java.io.Serializable
```

