## Feign使编写Java HTTP客户端更加容易

​		Feign是受Retrofit，JAXRS-2.0和WebSocket启发的Java HTTP客户端绑定程序。Feign的第一个目标是减少与ReSTfulness无关的将Denominator统一绑定到HTTP API的复杂性。

## 路线图



### 短期-我们现在正在做什么

- 响应缓存
  - 支持缓存api响应。允许用户定义在什么条件下响应适合进行缓存以及应使用哪种类型的缓存机制。
  - 支持内存中缓存和外部缓存实现（EhCache，Google，Spring等）
- 完整的URI模板表达式支持
  - 支持1级到4级URI模板表达式。
  - 使用URI模板TCK验证符合性。
- 日志API重构
  - 重构Logger API，使其更接近SLF4J之类的框架，从而为Feign中的日志记录提供通用的思维模型。Feign本身将始终使用此模型，并为Logger的使用方式提供更清晰的指导。
- `Retry` API 重构
  - 重构Retry API以支持用户提供的条件并更好地控制退避策略。这可能会导致不向后兼容的重大更改

### 中期-接下来会发生什么

- Metric API
  - 提供一流的Metrics API，用户可以利用该API深入了解请求/响应生命周期。可能会提供更好的OpenTracing支持。
- 通过CompletableFuture支持异步执行
  - 允许在请求/响应生命周期中进行将来的链接和执行者管理。实施将需要非向后兼容的重大更改。但是，在考虑执行响应之前，需要此功能。

- 通过eactive Streams提供反应执行支持
  - 对于JDK 9+，请考虑使用java.util.concurrent.Flow的本机实现。
  - 支持JDK 8上的Project Reactor和RxJava 2+实现。

### 长期-未来

- 附加断路器支持

  支持其他断路器实现，例如Resilience4J和弹簧断路器



### 为什么是Feign而不是X

​		Feign使用Jersey和CXF之类的工具为ReST或SOAP服务编写Java客户端。此外，Feign允许您在诸如Apache HC之类的http库之上编写自己的代码。Feign通过可自定义的解码器和错误处理功能，以最小的开销和代码将代码连接到http API，可以将其写入任何基于文本的http API。

### Feign是如何工作的

​		Feign通过将注释处理为模板化请求来工作。在输出之前，参数以简单的方式应用于这些模板。尽管Feign仅限于支持基于文本的API，但它极大地简化了系统方面，例如重播请求。此外，Feign知道这一点后，就可以轻松对转换进行单元测试。

### Java版本兼容性

 		Feign 10.x及更高版本基于Java 8构建，并且应可在Java 9、10和11上运行。对于那些需要JDK 6兼容性的应用程序，请使用Feign9.x。

### 基本用法

使用通常看起来像这样，是对标准Retrofit示例的改编。

```java
interface GitHub {
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repo);

  @RequestLine("POST /repos/{owner}/{repo}/issues")
  void createIssue(Issue issue, @Param("owner") String owner, @Param("repo") String repo);

}

public static class Contributor {
  String login;
  int contributions;
}

public static class Issue {
  String title;
  String body;
  List<String> assignees;
  int milestone;
  List<String> labels;
}

public class MyApp {
  public static void main(String... args) {
    GitHub github = Feign.builder()
                         .decoder(new GsonDecoder())
                         .target(GitHub.class, "https://api.github.com");
  
    // Fetch and print a list of the contributors to this library.
    List<Contributor> contributors = github.contributors("OpenFeign", "feign");
    for (Contributor contributor : contributors) {
      System.out.println(contributor.login + " (" + contributor.contributions + ")");
    }
  }
}
```

### 接口注解

​		Feign 注解定义接口与基础客户端应如何工作之间的合同。Feign的默认合同定义了以下注释：

| 注解         | 接口中作用目标     | 用法                                                         |
| ------------ | ------------------ | ------------------------------------------------------------ |
| @RequestLine | 方法上             | 定义请求的HttpMethod和UriTemplate。表达式，用大括号{expression}括起来的值使用其相应的@Param注释参数进行解析。 |
| @Param       | 参数               | 定义一个模板变量，其名称将用于解析对应的模板表达式的值。     |
| @Header      | 方法或类 接口 枚举 | 定义一个HeaderTemplate；UriTemplate的变体。使用@Param带注释的值来解析相应的表达式。当在Type上使用时，该模板将应用于每个请求。在方法上使用时，模板将仅应用于带注释的方法。 |
| @QueryMap    | 参数上             | 定义一个名称-值对或POJO的映射，以扩展为查询字符串。          |
| @HeaderMap   | 参数上             | 定义名称-值对的映射，以扩展为Http Header                     |
| @Body        | 方法上             | 定义一个类似于UriTemplate和HeaderTemplate的模板，该模板使用@Param注释的值来解析相应的表达式。 |

#### 覆盖请求行

​		如果需要将请求定向到其他主机，则需要在创建Feign客户端时提供该请求，或者要为每个请求提供目标主机，请包含java.net.URI参数，Feign将使用该值作为请求目标。

```java
@RequestLine("POST /repos/{owner}/{repo}/issues")
void createIssue(URI host, Issue issue, @Param("owner") String owner, @Param("repo") String repo);
```

### 模板和表达式

​	feign表达式表示由URI模板-RFC 6570定义的简单字符串表达式（级别1）。使用其相应的带参数注释的方法参数来扩展表达式。

例

```java
public interface GitHub {
  
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repository);
  
  class Contributor {
    String login;
    int contributions;
  }
}

public class MyApp {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                         .decoder(new GsonDecoder())
                         .target(GitHub.class, "https://api.github.com");
    /*  owner和repository将被用于扩展请求行中的{owner}和{repository}表达式 
     * 结果 uri 是 https://api.github.com/repos/OpenFeign/feign/contributors
     */
    github.contributors("OpenFeign", "feign");
  }
}
```

表达式必须用大括号{}括起来，并且可能包含正则表达式模式，并用冒号：分隔以限制解析值。示例所有者必须是字母。{owner：[a-zA-Z] *}

#### 请求参数扩展

RequestLine和QueryMap模板遵循URI模板-用于1级模板的RFC 6570规范，该规范指定以下内容：

- 未解析的表达式被省略
- 如果尚未通过@Param注解对所有文字和变量值进行编码或标记为已编码，则所有文字和变量值均经过pct编码。

#### Undefined 与 空值

未定义的表达式是表达式的值是显式null或未提供值的表达式。根据URI模板-RFC 6570，可以为表达式提供一个空值。当Feign解析表达式时，它将首先确定是否定义了值，如果是，则保留查询参数。如果表达式未定义，则删除查询参数。有关完整的细分，请参见下文。

参数值是空字符

```
public void test() {
   Map<String, Object> parameters = new LinkedHashMap<>();
   parameters.put("param", "");
   this.demoClient.test(parameters);
}
```

结果

```
http://localhost:8080/test?param=
```

参数丢失

```
public void test() {
   Map<String, Object> parameters = new LinkedHashMap<>();
   this.demoClient.test(parameters);
}
```

结果

```
http://localhost:8080/test
```

*Undefined*

```
public void test() {
   Map<String, Object> parameters = new LinkedHashMap<>();
   parameters.put("param", null);
   this.demoClient.test(parameters);
}
```

结果

```java
http://localhost:8080/test
```

有关更多示例，请参见高级用法。

##### 斜杠呢？/

@RequestLine模板默认情况下不对斜杠/字符进行编码。若要更改此行为，请将@RequestLine上的encodeSlash属性设置为false。

##### 那加呢？+

根据URI规范，URI的路径和查询段中都允许使用+号，但是查询中符号的处理可能不一致。在某些旧系统中，+等于空格。Feign采用了现代系统的方法，其中+符号不应表示空格，并且在查询字符串上找到时会显式编码为％2B。

如果希望将+用作空格，请使用文字字符或将值直接编码为％20

#### 自定义扩展

Headers和HeaderMap模板遵循与“请求参数扩展”相同的规则，但有以下更改：

- 未解析的表达式将被省略。如果结果是空的标题值，则将删除整个标题。
- 不执行pct编码。

有关示例，请参见Header。

有关@Param参数及其名称的说明：

- 具有相同名称的所有表达式，无论它们在@ RequestLine，@ QueryMap，@ BodyTemplate或@Headers上的位置如何，都将解析为相同的值。在以下示例中，contentType的值将用于解析标头和路径表达式：

```java
public interface ContentService {
  @RequestLine("GET /api/documents/{contentType}")
  @Headers("Accept: {contentType}")
  String getDocumentByType(@Param("contentType") String type);
}
```

设计接口时请记住这一点。

#### 请求体扩展（Request Body Expansion）

Body模板遵循与“请求参数扩展”相同的规则，但有以下更改：

- 未解析的表达式将被省略。
- 扩展值在放入请求主体之前不会通过编码器传递
- 必须指定Content-Type标头。有关示例，请参见 [Body Templates](https://github.com/OpenFeign/feign#body-templates) 。

### 客制化

Feign有几个可以定制的方面。对于简单的情况，可以使用Feign.builder（）来使用自定义组件构造API接口。例如：

```java
interface Bank {
  @RequestLine("POST /account/{id}")
  Account getAccountInfo(@Param("id") String id);
}

public class BankService {
  public static void main(String[] args) {
    Bank bank = Feign.builder().decoder(
        new AccountDecoder())
        .target(Bank.class, "https://api.examplebank.com");
  }
}
```

### 多种接口

Feign可以产生多个api接口。这些被定义为Target <T>（默认HardCodedTarget <T>），允许在执行之前动态发现和修饰请求。例如，以下模式可能使用身份服务中的当前url和auth令牌修饰每个请求。

```java
public class CloudService {
  public static void main(String[] args) {
    CloudDNS cloudDNS = Feign.builder()
      .target(new CloudIdentityTarget<CloudDNS>(user, apiKey));
  }
  
  class CloudIdentityTarget extends Target<CloudDNS> {
    /* implementation of a Target */
  }
}
```

### 例子

Feign包括示例GitHub和Wikipedia客户端。分母项目在实践中也可以取消。特别是，请看一下其示例守护程序。



## 整合方式

Feign设计的目的是与其他开放源代码工具一起很好地工作。欢迎集成您喜欢的项目！

### Gson

Gson包括可用于JSON API的编码器和解码器。

将GsonEncoder和/或GsonDecoder添加到您的Feign.Builder中，如下所示：

```java
public class Example {
  public static void main(String[] args) {
    GsonCodec codec = new GsonCodec();
    GitHub github = Feign.builder()
                         .encoder(new GsonEncoder())
                         .decoder(new GsonDecoder())
                         .target(GitHub.class, "https://api.github.com");
  }
}
```

### Jackson

Jackson包含可与JSON API一起使用的编码器和解码器。

像这样将JacksonEncoder和/或JacksonDecoder添加到您的Feign.Builder中：

```java
public class Example {
  public static void main(String[] args) {
      GitHub github = Feign.builder()
                     .encoder(new JacksonEncoder())
                     .decoder(new JacksonDecoder())
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

### Sax

SaxDecoder允许您以与普通JVM以及Android环境兼容的方式来解码XML。

这是有关如何配置Sax响应解析的示例：

```java
public class Example {
  public static void main(String[] args) {
      Api api = Feign.builder()
         .decoder(SAXDecoder.builder()
                            .registerContentHandler(UserIdHandler.class)
                            .build())
         .target(Api.class, "https://apihost");
    }
}
```

### JAXB

JAXB包括可与XML API一起使用的编码器和解码器。

像这样将JAXBEncoder和/或JAXBDecoder添加到您的Feign.Builder中：

```java
public class Example {
  public static void main(String[] args) {
    Api api = Feign.builder()
             .encoder(new JAXBEncoder())
             .decoder(new JAXBDecoder())
             .target(Api.class, "https://apihost");
  }
}
```

### JAX-RS

JAXRSContract会覆盖注释处理，以改用JAX-RS规范提供的标准注释。当前针对1.1规范。

Here's the example above re-written to use JAX-RS:

```java
interface GitHub {
  @GET @Path("/repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@PathParam("owner") String owner, @PathParam("repo") String repo);
}

public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                       .contract(new JAXRSContract())
                       .target(GitHub.class, "https://api.github.com");
  }
}
```

### OkHttp

OkHttpClient将Feign的http请求定向到OkHttp，从而启用SPDY和更好的网络控制。

要将OkHttp与Feign一起使用，请将OkHttp模块添加到您的类路径中。然后，将Feign配置为使用OkHttpClient：

```java
public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                     .client(new OkHttpClient())
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

### Ribbon

RibbonClient覆盖了Feign客户端的URL解析，添加了Ribbon提供的智能路由和弹性功能。

集成需要您将功能区客户端名称作为URL的主机部分传递，例如myAppProd。

```java
public class Example {
  public static void main(String[] args) {
    MyService api = Feign.builder()
          .client(RibbonClient.create())
          .target(MyService.class, "https://myAppProd");
  }
}
```

### Java 11 Http2

Http2Client将Feign的http请求定向到实现HTTP / 2的Java11 New HTTP / 2 Client。

要将新的HTTP / 2客户端与Feign一起使用，请使用Java SDK11。然后，将Feign配置为使用Http2Client：

```java
GitHub github = Feign.builder()
                     .client(new Http2Client())
                     .target(GitHub.class, "https://api.github.com");
```

### Hystrix

HystrixFeign配置Hystrix提供的断路器支持。

要将Hystrix与Feign一起使用，请将Hystrix模块添加到您的类路径中。然后使用HystrixFeign构建器：

```java
public class Example {
  public static void main(String[] args) {
    MyService api = HystrixFeign.builder().target(MyService.class, "https://myAppProd");
  }
}
```

### SOAP

SOAP包括可与XML API一起使用的编码器和解码器。

该模块增加了对通过JAXB和SOAPMessage编码和解码SOAP Body对象的支持。通过将它们包装到原始javax.xml.ws.soap.SOAPFaultException中，它还提供了SOAPFault解码功能，因此您只需要捕获SOAPFaultException即可处理SOAPFault。

像这样将SOAPEncoder和/或SOAPDecoder添加到您的Feign.Builder中：

```java
public class Example {
  public static void main(String[] args) {
    Api api = Feign.builder()
	     .encoder(new SOAPEncoder(jaxbFactory))
	     .decoder(new SOAPDecoder(jaxbFactory))
	     .errorDecoder(new SOAPErrorDecoder())
	     .target(MyApi.class, "http://api");
  }
}
```

注意：如果由于错误HTTP代码（4xx，5xx等）而返回SOAP Faults，则可能还需要添加SOAPErrorDecoder。

### SLF4J

SLF4JModule允许将Feign的日志记录定向到SLF4J，从而使您可以轻松地使用自己选择的日志记录后端（Logback，Log4J等）

要将SLF4J与Feign一起使用，请将SLF4J模块和您选择的SLF4J绑定都添加到类路径中。然后，配置Feign以使用Slf4jLogger：

```java
public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                     .logger(new Slf4jLogger())
                     .logLevel(Level.FULL)
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

### Decoders

Feign.builder（）允许您指定其他配置，例如如何解码响应。

如果接口中的任何方法返回的类型除了Response，String，byte []或void外，您都需要配置一个非默认的Decoder。

以下是配置JSON解码的方法（使用feign-gson扩展名）：

```java
public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                     .decoder(new GsonDecoder())
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

如果需要在将响应提供给解码器之前进行预处理，则可以使用mapAndDecode构建器方法。一个示例用例处理的是仅提供jsonp的API，您可能需要先解开jsonp，然后再将其发送到您选择的Json解码器：

```java
public class Example {
  public static void main(String[] args) {
    JsonpApi jsonpApi = Feign.builder()
                         .mapAndDecode((response, type) -> jsopUnwrap(response, type), new GsonDecoder())
                         .target(JsonpApi.class, "https://some-jsonp-api.com");
  }
}
```

### Encoders

将请求正文发送到服务器的最简单方法是定义一个POST方法，该方法具有String或byte []参数，且上面没有任何注解。您可能需要添加Content-Type标头。

```java
interface LoginClient {
  @RequestLine("POST /")
  @Headers("Content-Type: application/json")
  void login(String content);
}

public class Example {
  public static void main(String[] args) {
    client.login("{\"user_name\": \"denominator\", \"password\": \"secret\"}");
  }
}
```

通过配置编码器，您可以发送类型安全的请求正文。这是使用feign-gson扩展的示例：

```java
static class Credentials {
  final String user_name;
  final String password;

  Credentials(String user_name, String password) {
    this.user_name = user_name;
    this.password = password;
  }
}

interface LoginClient {
  @RequestLine("POST /")
  void login(Credentials creds);
}

public class Example {
  public static void main(String[] args) {
    LoginClient client = Feign.builder()
                              .encoder(new GsonEncoder())
                              .target(LoginClient.class, "https://foo.com");
    
    client.login(new Credentials("denominator", "secret"));
  }
}
```

### @Body templates

@Body批注指示使用@Param批注的参数扩展的模板。您可能需要添加Content-Type标头。

```java
interface LoginClient {

  @RequestLine("POST /")
  @Headers("Content-Type: application/xml")
  @Body("<login \"user_name\"=\"{user_name}\" \"password\"=\"{password}\"/>")
  void xml(@Param("user_name") String user, @Param("password") String password);

  @RequestLine("POST /")
  @Headers("Content-Type: application/json")
  // json curly braces must be escaped!
  @Body("%7B\"user_name\": \"{user_name}\", \"password\": \"{password}\"%7D")
  void json(@Param("user_name") String user, @Param("password") String password);
}

public class Example {
  public static void main(String[] args) {
    client.xml("denominator", "secret"); // <login "user_name"="denominator" "password"="secret"/>
    client.json("denominator", "secret"); // {"user_name": "denominator", "password": "secret"}
  }
}
```

### Headers

根据使用情况，Feign支持将请求的设置标头作为api的一部分或作为客户端的一部分。

#### 使用API设置headers

在特定接口或调用应始终设置某些headers值的情况下，将headers定义为api的一部分是有意义的。

可以使用@Headers批注在api接口或方法上设置静态标头。

```java
@Headers("Accept: application/json")
interface BaseApi<V> {
  @Headers("Content-Type: application/json")
  @RequestLine("PUT /api/{key}")
  void put(@Param("key") String key, V value);
}
```

方法可以使用@Headers中的变量扩展为静态标头指定动态内容。

```java
public interface Api {
   @RequestLine("POST /")
   @Headers("X-Ping: {token}")
   void post(@Param("token") String token);
}
```

如果标头字段键和值都是动态的，并且可能无法提前知道可能的键的范围，并且可能在同一api /客户端中的不同方法调用之间有所不同（例如，自定义元数据标头字段，例如“ x-amz-meta- *“或” x-goog-meta- *“），可以使用HeaderMap注释Map参数，以构造将地图内容用作其标题参数的查询。

```java
public interface Api {
   @RequestLine("POST /")
   void post(@HeaderMap Map<String, Object> headerMap);
}
```

这些方法将标头条目指定为api的一部分，并且在构建Feign客户端时不需要任何自定义。

#### 为每个目标设置Headers

要为目标上的每个请求方法自定义标头，可以使用RequestInterceptor。RequestInterceptor可以在Target实例之间共享，并且应该是线程安全的。RequestInterceptor应用于Target上的所有请求方法。

如果您需要按方法自定义，则需要自定义Target，因为RequestInterceptor无法访问当前方法元数据。

有关使用RequestInterceptor设置标头的示例，请参见Request Interceptors部分。

Headers可以设置为自定义目标的一部分。

```java
  static class DynamicAuthTokenTarget<T> implements Target<T> {
    public DynamicAuthTokenTarget(Class<T> clazz,
                                  UrlAndTokenProvider provider,
                                  ThreadLocal<String> requestIdProvider);
    
    @Override
    public Request apply(RequestTemplate input) {
      TokenIdAndPublicURL urlAndToken = provider.get();
      if (input.url().indexOf("http") != 0) {
        input.insert(0, urlAndToken.publicURL);
      }
      input.header("X-Auth-Token", urlAndToken.tokenId);
      input.header("X-Request-ID", requestIdProvider.get());

      return input.request();
    }
  }
  
  public class Example {
    public static void main(String[] args) {
      Bank bank = Feign.builder()
              .target(new DynamicAuthTokenTarget(Bank.class, provider, requestIdProvider));
    }
  }
```

这些方法取决于在Feign客户端构建时在客户端上设置的自定义RequestInterceptor或Target，并且可以用作基于每个客户端在所有api调用上设置标头的方法。这对于执行诸如基于每个客户端在所有api请求的标头中设置身份验证令牌之类的操作很有用。当在调用api调用的线程上进行api调用时运行这些方法，这允许在调用时以特定于上下文的方式动态设置标头-例如，可以使用线程本地存储来根据调用线程设置不同的标头值，这对于诸如为请求设置特定于线程的跟踪标识符之类的事情可能很有用。



### 高级用法

#### Base Apis

在许多情况下，服务的api遵循相同的约定。Feign通过单继承接口支持此模式。

考虑示例：

```java
interface BaseAPI {
  @RequestLine("GET /health")
  String health();

  @RequestLine("GET /all")
  List<Entity> all();
}
```

您可以继承基本方法来定义和定位特定的api。

```java
interface CustomAPI extends BaseAPI {
  @RequestLine("GET /custom")
  String custom();
}
```

在许多情况下，资源表示形式也是一致的。因此，基本api接口上支持类型参数。

```java
@Headers("Accept: application/json")
interface BaseApi<V> {

  @RequestLine("GET /api/{key}")
  V get(@Param("key") String key);

  @RequestLine("GET /api")
  List<V> list();

  @Headers("Content-Type: application/json")
  @RequestLine("PUT /api/{key}")
  void put(@Param("key") String key, V value);
}

interface FooApi extends BaseApi<Foo> { }

interface BarApi extends BaseApi<Bar> { }
```

#### Logging

您可以通过设置Logger记录去往和来自目标的HTTP消息。这是最简单的方法：

```java
public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                     .decoder(new GsonDecoder())
                     .logger(new Logger.JavaLogger("GitHub.Logger").appendToFile("logs/http.log"))
                     .logLevel(Logger.Level.FULL)
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

> 关于JavaLogger的注释：避免使用默认的JavaLogger（）构造函数-它被标记为已弃用，并将很快被删除。

SLF4JLogger（参见上文）也可能是您感兴趣的。

#### Request Interceptors

当您需要更改所有请求时，无论它们是什么目标，都需要配置一个RequestInterceptor。例如，如果您充当中介，则可能要传播X-Forwarded-For标头。

```java
static class ForwardedForInterceptor implements RequestInterceptor {
  @Override public void apply(RequestTemplate template) {
    template.header("X-Forwarded-For", "origin.host.com");
  }
}

public class Example {
  public static void main(String[] args) {
    Bank bank = Feign.builder()
                 .decoder(accountDecoder)
                 .requestInterceptor(new ForwardedForInterceptor())
                 .target(Bank.class, "https://api.examplebank.com");
  }
}
```

拦截器的另一个常见示例是身份验证，例如使用内置的BasicAuthRequestInterceptor。

```java
public class Example {
  public static void main(String[] args) {
    Bank bank = Feign.builder()
                 .decoder(accountDecoder)
                 .requestInterceptor(new BasicAuthRequestInterceptor(username, password))
                 .target(Bank.class, "https://api.examplebank.com");
  }
}
```

#### 自定义@Param扩展

用Param注释的参数基于它们的toString展开。通过指定自定义的Param.Expander，用户可以控制此行为，例如格式化日期。

```java
public interface Api {
  @RequestLine("GET /?since={date}") Result list(@Param(value = "date", expander = DateToMillis.class) Date date);
}
```

#### 动态查询参数

可以使用QueryMap注释Map参数，以构造将地图内容用作其查询参数的查询。

```java
public interface Api {
  @RequestLine("GET /find")
  V find(@QueryMap Map<String, Object> queryMap);
}
```

这也可以用于使用QueryMapEncoder从POJO对象生成查询参数

```java
public interface Api {
  @RequestLine("GET /find")
  V find(@QueryMap CustomPojo customPojo);
}
```

当以这种方式使用时，无需指定自定义QueryMapEncoder，将使用成员变量名称作为查询参数名称来生成查询映射。以下POJO将生成查询参数“ / find？name = {name}＆number = {number}”（不保证所包含查询参数的顺序，并且照常，如果任何值为null，它将被忽略）。

```java
public class CustomPojo {
  private final String name;
  private final int number;

  public CustomPojo (String name, int number) {
    this.name = name;
    this.number = number;
  }
}
```

要设置自定义QueryMapEncoder，请执行以下操作：

```java
public class Example {
  public static void main(String[] args) {
    MyApi myApi = Feign.builder()
                 .queryMapEncoder(new MyCustomQueryMapEncoder())
                 .target(MyApi.class, "https://api.hostname.com");
  }
}
```

使用@QueryMap注释对象时，默认编码器使用反射检查提供的对象字段，以将对象值扩展为查询字符串。如果您希望使用Java Beans API中定义的getter和setter方法构建查询字符串，请使用BeanQueryMapEncoder

```java
public class Example {
  public static void main(String[] args) {
    MyApi myApi = Feign.builder()
                 .queryMapEncoder(new BeanQueryMapEncoder())
                 .target(MyApi.class, "https://api.hostname.com");
  }
}
```

### Error Handling-错误处理

如果您需要对处理意外响应的更多控制，则Feign实例可以通过构建器注册自定义ErrorDecoder。

```java
public class Example {
  public static void main(String[] args) {
    MyApi myApi = Feign.builder()
                 .errorDecoder(new MyErrorDecoder())
                 .target(MyApi.class, "https://api.hostname.com");
  }
}
```

所有导致HTTP状态不在2xx范围内的响应都将触发ErrorDecoder的解码方法，使您可以处理响应，将失败包装到自定义异常中或执行任何其他处理。如果要再次重试该请求，则抛出RetryableException。这将调用已注册的重试器。

### Retry-重试

默认情况下，Feign会自动重试IOException，而与HTTP方法无关，将其视为与网络临时相关的异常，以及从ErrorDecoder抛出的任何RetryableException。要自定义此行为，请通过构建器注册自定义Retryer实例。

```java
public class Example {
  public static void main(String[] args) {
    MyApi myApi = Feign.builder()
                 .retryer(new MyRetryer())
                 .target(MyApi.class, "https://api.hostname.com");
  }
}
```

重试程序负责通过从方法continueOrPropagate（RetryableException e）返回true或false来确定是否应进行重试。将为每个客户端执行创建一个Retryer实例，如果需要，您可以在每个请求之间维护状态。

如果确定重试不成功，则将抛出最后一个RetryException。要抛出导致重试失败的原始原因，请使用exceptionPropagationPolicy（）选项构建Feign客户端。

#### Static and Default Methods-静态和默认方法

Feign定位的接口可能具有静态或默认方法（如果使用Java 8+）。这些允许Feign客户端包含底层API未明确定义的逻辑。例如，静态方法使指定通用客户端构建配置变得容易。默认方法可用于组成查询或定义默认参数。

```java
interface GitHub {
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repo);

  @RequestLine("GET /users/{username}/repos?sort={sort}")
  List<Repo> repos(@Param("username") String owner, @Param("sort") String sort);

  default List<Repo> repos(String owner) {
    return repos(owner, "full_name");
  }

  /**
   * Lists all contributors for all repos owned by a user.
   */
  default List<Contributor> contributors(String user) {
    MergingContributorList contributors = new MergingContributorList();
    for(Repo repo : this.repos(owner)) {
      contributors.addAll(this.contributors(user, repo.getName()));
    }
    return contributors.mergeResult();
  }

  static GitHub connect() {
    return Feign.builder()
                .decoder(new GsonDecoder())
                .target(GitHub.class, "https://api.github.com");
  }
}
```

