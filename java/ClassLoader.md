	类加载器是一个加载类的对象。是一个抽象类，通过给定的二进制名称，类加载器尝试定位或生成组成class定义的数据。常用的策略从文件系统中通过二进制名字转换为一个class文件。  

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

## getSystemClassLoader方法讲解

​	返回用于委托的系统类加载器。这是新的 ClassLoader （自定义类加载）实例的默认*委托父对象，并且*通常是用于启动应用程序的类加载器。

​	首先在运行时的启动序列中首先调用此方法，然后创建系统类加载器并将其设置为调用 Thread 的上下文类加载器。

​	默认的系统类加载器是此类的与实现相关的*实例。

​	如果在首次调用此方法时定义了系统属性“  java.system.class.loader ”，那么该属性的值将作为将返回的类的名称。系统*类加载器。该类使用默认的系统类加载器*加载，并且必须定义一个公共构造函数，该构造函数的参数类型为ClassLoader 的单个参数 作为委托父级。然后使用此构造函数以默认系统*类加载器作为参数创建*实例。生成的类加载器被定义为系统类加载器。

```java
@CallerSensitive
public static ClassLoader getSystemClassLoader() {
    initSystemClassLoader();//初始化系统类加载器
    if (scl == null) {
        return null;
    }
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkClassLoaderPermission(scl, Reflection.getCallerClass());
    }
    return scl;
}
//初始化是串行的，因为这里加了锁
private static synchronized void initSystemClassLoader() {
        if (!sclSet) {//如果系统类加载器还没有设置
            if (scl != null)//如果已经系统类加载器不为null.双重检查。锁的可重入机制
                throw new IllegalStateException("recursive invocation");
          //launcher 执行构造函数的时候已经指定类线程的上下文类加载器。但是如果这里系统
          //属性，则进行覆盖
            sun.misc.Launcher l = sun.misc.Launcher.getLauncher();
            if (l != null) {
                Throwable oops = null;
                scl = l.getClassLoader();//获取类加载器
                try {
                  //对系统类加载器进行一次处理
                    scl = AccessController.doPrivileged(
                        new SystemClassLoaderAction(scl));
                } catch (PrivilegedActionException pae) {
                    oops = pae.getCause();
                    if (oops instanceof InvocationTargetException) {
                        oops = oops.getCause();
                    }
                }
                if (oops != null) {
                    if (oops instanceof Error) {
                        throw (Error) oops;
                    } else {
                        // wrap the exception
                        throw new Error(oops);
                    }
                }
            }
            sclSet = true;
        }
    }
class SystemClassLoaderAction
    implements PrivilegedExceptionAction<ClassLoader> {
    private ClassLoader parent;

    SystemClassLoaderAction(ClassLoader parent) {
        this.parent = parent;
    }
		//执行此方法
    public ClassLoader run() throws Exception {
      //获取系统配置信息。如果设置了则覆盖线程的上下文类加载器
        String cls = System.getProperty("java.system.class.loader");
        if (cls == null) {
            return parent;
        }
				//获取系统类加载器的构造函数
        Constructor<?> ctor = Class.forName(cls, true, parent)
            .getDeclaredConstructor(new Class<?>[] { ClassLoader.class });
      	//实例化系统类加载器
        ClassLoader sys = (ClassLoader) ctor.newInstance(
            new Object[] { parent });
      	//设置线程的上下文加载器为系统类加载器
        Thread.currentThread().setContextClassLoader(sys);
        return sys;
    }
}

```

Launcher类在实例化的时候执行的操作

```java
 public Launcher() {
        Launcher.ExtClassLoader var1;
        try {
          //扩展类加载器
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }

        try {
          //系统类加载器
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }
				//设置线程原始的上下文类加载器为appclassloader
        Thread.currentThread().setContextClassLoader(this.loader);
        String var2 = System.getProperty("java.security.manager");
        if (var2 != null) {
            SecurityManager var3 = null;
            if (!"".equals(var2) && !"default".equals(var2)) {
                try {
                    var3 = (SecurityManager)this.loader.loadClass(var2).newInstance();
                } catch (IllegalAccessException var5) {
                    ;
                } catch (InstantiationException var6) {
                    ;
                } catch (ClassNotFoundException var7) {
                    ;
                } catch (ClassCastException var8) {
                    ;
                }
            } else {
                var3 = new SecurityManager();
            }

            if (var3 == null) {
                throw new InternalError("Could not create SecurityManager: " + var2);
            }

            System.setSecurityManager(var3);
        }

    }

```

