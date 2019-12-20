​	类加载器是一个加载类的对象。是一个抽象类，通过给定的二进制名称，类加载器尝试定位或生成组成class定义的数据。常用的策略从文件系统中通过二进制名字转换为一个class文件。  

​	每一个Class对象都有一个getClassLoader()方法，用来获取定义Class的类加载器。  

​	对于数组类型的Class不是类加载器创建的，是在运行时动态创建的。数组的getClassLoader() 返回的类加载器和数组中对象的类加载器一样。如果数组中的元素是原生类型（char byte short int long double boolean）,则没有类加载器。因为是bootstrap加载的。getClassLoader() 返回nul l。  

​	应用程序实现类加载器的子类，以扩展虚拟机动态加载类的方式。  

​	类加载器使用委托模型查找类和资源。每一个类加载器都有一个相关的父类加载器，当查找一个类或资源时，首选让父类加载器进行查找。虚拟机内建的类加载器叫bootstrap 类加载器，没有父类加载器，是其它类加载的父类。  

​	类加载器支持并行加载类，如果当前虚拟器支持并行。并行需要在类初始化的时候调用ClassLoader.registerAsParallelCapable注册它们，包括类的子类。  

​	在不是严格分层的委托模型行，具备并行加载能力的类加载器，在并行加载的时候或导致死锁，在类加载期间加载锁被持有 。查看loadClass 进行查看。  

​	通常，虚拟机会以一种平台相关的方式加载类，例如在unix系统中，虚拟机从环境变量CLASSPATH的值的文件夹中加载类。  

​		然而，一些类不一定来源于一个文件，可以从其它资源获取，例如网络，或者通过一个应用。方法defineClass(String, byte[], int, int)转换一个字节数组为class实例。新定义的class实例可以通过Class.newInstance 创建。  

​	以下实例通过网络构建实例



```java
ClassLoader loader&nbsp;= new NetworkClassLoader(host,&nbsp;port);
Object main&nbsp;= loader.loadClass("Main", true).newInstance();
		
```

​	网络类加载器必须定义findClass和loadClassData方法。一旦下载组成class的字节，需要通过defineClass去创建一个类的实例。例如：

```java
 class NetworkClassLoader extends ClassLoader {
         String host;
         int port;

         public Class findClass(String name) {
             byte[] b = loadClassData(name);
             return defineClass(name, b, 0, b.length);
         }
         private byte[] loadClassData(String name) {
             // load the class data from the connection
             &nbsp;.&nbsp;.&nbsp;.
         }
     }
```

​		二进制名字

​		任何作为字符提供给类加载使用的参数，都必须是复合java规范的二进制名称。

```java
java.lang.String   //String类
javax.swing.JSpinner$DefaultEditor//JSpinner 内部类DefaultEditor
java.security.KeyStore$Builder$FileBuilder$1  //KeyStore$Builder$FileBuilder 中的匿名内部类1，因为匿名类没有名字，所以用数字表示
java.net.URLClassLoader$3$1 //URLClassLoader类中的匿名内部类3种匿名内部类1
  /**
  内部类通过$分割
  **/
```

​	

```java
public abstract class ClassLoader {

    private static native void registerNatives();
    static {
        registerNatives();
    }

    // The parent class loader for delegation
    // Note: VM hardcoded the offset of this field, thus all new fields
    // must be added *after* it.
    private final ClassLoader parent;
```

