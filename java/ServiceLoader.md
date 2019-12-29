​	是一个简单的服务提供者加载工具。 

​	服务是一组接口或者抽象类。服务提供者通常会实现服务。服务提供者类通常会实现接口，子类的定义来源于服务本身。服务提供者以扩展名的形式安装在java平台实现中，也就是说，将jar文件放置在任何常用的扩展目录中。也可以将服务提供者添加到应用程序的类路径中或其它特定于平台的可用的方式。 

​	为了加载，服务由单一的类型表示，要么是接口要么是类。（可以使用具体的类，但不建议这样做。）服务实现者提供包含一个或多个具体的类，这些具体的类用数据和提供特定的代码扩展次服务类型。provider类通常不是提供程序本身，而是代理，其中包含足够的信息，以决定提供程序是否能够满足特定请求以及可以在其上创建使劲提供程序的代码需求。提供程序类的细节往往是高度特定于服务的；没有单个类或接口可能统一他们，因此此处未定义此类。此功能强制执行的唯一要求是，提供程序类必须具有无参的构造函数，以便可以在加载的过程中实例化。  

​		通过在资源目录META-INF / services 放置服务提供配置文件来标识服务提供。配置文件名称是标准的二进制名称（详见ClassLoader中的二进制名称说明），该文件包含一个具体的实现类的标准的二进制名称列表，每行一个。每个名称周围的空格或制表符以及空白行都将被忽略。注释字符为 ＃ 数字符在每一行上，第一个注释字符之后的所有字符都将被忽略。 文件必须以UTF-8编码。

​	如果一个特定的具体提供程序类在多个配置文件中被命名，或者在同一配置文件中被命名多次，则重复项将被忽略。命名特定提供程序的配置文件不必与提供程序本身位于同一jar文件或其他分发单元中。必须从最初为查找配置文件而查询的同一类加载器中可以访问该提供程序； 请注意，这不一定是实际加载文件的类加载器。  

​	服务提供者的定位和实例化是按需加载（懒加载）。服务加载器维护到目前为止已加载的提供者的缓存。每次调用{@link #iterator iterator}方法时，都会返回一个*迭代器，该迭代器首先以实例化顺序生成缓存的所有元素，然后懒惰地查找和实例化所有剩余的*提供程序，并将每个提供程序添加到缓存中转。可以通过{@link #reload reload}方法清除缓存。 

​	服务加载程序始终在调用方的安全上下文中执行。 *受信任的系统代码通常应在特权*安全上下文中调用此类中的方法，以及它们返回的迭代器的方法。  

​	多线程下，类的实例化并不是安全的。

​	除非有特殊说明，否则空参数传递给此类的任何方法都会引用空指针异常。   

​		

```java
/**
	Example
* Suppose we have a service type <tt>com.example.CodecSet</tt> which is
* intended to represent sets of encoder/decoder pairs for some protocol.  In
* this case it is an abstract class with two abstract methods:
*
* public abstract Encoder getEncoder(String encodingName);
* public abstract Decoder getDecoder(String encodingName);
*
* Each method returns an appropriate object or <tt>null</tt> if the provider
* does not support the given encoding.  Typical providers support more than
* one encoding.
* <p> If <tt>com.example.impl.StandardCodecs</tt> is an implementation of the
* <tt>CodecSet</tt> service then its jar file also contains a file named
*
* META-INF/services/com.example.CodecSet</pre></blockquote>
*
* <p> This file contains the single line:
*
* <blockquote><pre>
* com.example.impl.StandardCodecs    # Standard codecs</pre></blockquote>
*
* <p> The <tt>CodecSet</tt> class creates and saves a single service instance
* at initialization:
*
* <blockquote><pre>
* private static ServiceLoader<CodecSet> codecSetLoader
*     = ServiceLoader.load(CodecSet.class);</pre></blockquote>
*
* <p> To locate an encoder for a given encoding name it defines a static
* factory method which iterates through the known and available providers,
* returning only when it has located a suitable encoder or has run out of
* providers.
*
* public static Encoder getEncoder(String encodingName) {
*     for (CodecSet cp : codecSetLoader) {
*         Encoder enc = cp.getEncoder(encodingName);
*         if (enc != null)
*             return enc;
*     }
*     return null;
* }
*
* <p> A <tt>getDecoder</tt> method is defined similarly.
*
*使用说明,如果用于提供程序加载的类加载器的类路径包括远程网络URL，则这些URL将在搜索提供程序配置文件的过程中被取消引用。
	此活动是正常的，尽管它可能导致在Web服务器日志中创建令人费解的条目。但是，如果未正确配置Web服务器，那么此活动可能会导致提供者加载算法失败虚假地。
	
	当*请求的资源不存在时，Web服务器应返回HTTP 404（未找到）响应。但是，在某些情况下，有时会将Web服务器错误地配置为返回HTTP 200（OK）响应以及有用的HTML错误页面。当此类尝试将HTML页面解析为提供程序配置文件时，这将引发{@link * ServiceConfigurationError}。解决此问题的最佳方法是修复配置错误的Web服务器，以返回正确的响应代码（HTTP 404）和HTML错误页面。
* @param  <S>
*         The type of the service to be loaded by this loader
*
* @author Mark Reinhold
* @since 1.6
*/
public final class ServiceLoader<S>
    implements Iterable<S>
{

    private static final String PREFIX = "META-INF/services/";
  /**
  构造函数。
  **/
   public static <S> ServiceLoader<S> load(Class<S> service) {
     //获取线程上下文类加载器为当前类加载器
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
  /**
  初始化一个LazyIterator.在迭代器中记录
  **/
   public void reload() {
        providers.clear();
        lookupIterator = new LazyIterator(service, loader);
    }
```