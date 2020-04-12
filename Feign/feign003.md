# Hystrix

该模块将Feign的http请求包装在Hystrix中，从而启用断路器模式。

要将Hystrix与Feign一起使用，请将Hystrix模块添加到您的类路径中。然后，配置Feign以使用HystrixInvocationHandler：

```java
GitHub github = HystrixFeign.builder()
        .target(GitHub.class, "https://api.github.com");
```

对于异步或反应式使用，请返回HystrixCommand <YourType>或CompletableFuture <YourType>。

为了与RxJava兼容，请使用rx.Observable <YourType>或rx.Single <YourType>。Rx类型很冷，这意味着直到有订阅者才进行http调用。

不返回HystrixCommand，CompletableFuture，rx.Observable或rx.Single的方法仍包装在HystrixCommand中，但是会自动为您调用execute（）。

```java
interface YourApi {
  @RequestLine("GET /yourtype/{id}")
  HystrixCommand<YourType> getYourType(@Param("id") String id);

  @RequestLine("GET /yourtype/{id}")
  Observable<YourType> getYourTypeObservable(@Param("id") String id);

  @RequestLine("GET /yourtype/{id}")
  Single<YourType> getYourTypeSingle(@Param("id") String id);

  @RequestLine("GET /yourtype/{id}")
  CompletableFuture<YourType> getYourTypeCompletableFuture(@Param("id") String id);

  @RequestLine("GET /yourtype/{id}")
  YourType getYourTypeSynchronous(@Param("id") String id);
}

YourApi api = HystrixFeign.builder()
                  .target(YourApi.class, "https://example.com");

// for reactive
api.getYourType("a").toObservable

// or apply hystrix to RxJava methods
api.getYourTypeObservable("a")

// for asynchronous
api.getYourType("a").queue();

// for synchronous
api.getYourType("a").execute();

// or for a CompletableFuture
api.getYourTypeCompletableFuture("a").thenApply(o -> "b");

// or to apply hystrix to existing feign methods.
api.getYourTypeSynchronous("a");
```

### 组和命令键

默认情况下，Hystrix组密钥与目标名称匹配，并且目标名称通常是基本URL。Hystrix命令键与日志记录键相同，等效于javadoc引用。

例如，对于规范的GitHub示例...

- 他的组密钥为“ https://api.github.com”，并且
- 该命令密钥将为“ GitHub＃contributors（String，String）”

您可以使用HystrixFeign.Builder＃setterFactory（SetterFactory）对此进行自定义，例如，从配置或注解中读取键映射。

```java
SetterFactory commandKeyIsRequestLine = (target, method) -> {
  String groupKey = target.name();
  String commandKey = method.getAnnotation(RequestLine.class).value();
  return HystrixCommand.Setter
      .withGroupKey(HystrixCommandGroupKey.Factory.asKey(groupKey))
      .andCommandKey(HystrixCommandKey.Factory.asKey(commandKey));
};

api = HystrixFeign.builder()
                  .setterFactory(commandKeyIsRequestLine)
                  ...
```

### Fallback support-支持

Fallback时间是已知值，当调用http方法时发生错误时会返回该值。例如，您可以返回缓存的结果，而不是向调用方引发错误。要使用此功能，请将目标接口的安全实现作为最后一个参数传递给HystrixFeign.Builder.target。

例子：

```java
// When dealing with fallbacks, it is less tedious to keep interfaces small.
interface GitHub {
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<String> contributors(@Param("owner") String owner, @Param("repo") String repo);
}

// This instance will be invoked if there are errors of any kind.
GitHub fallback = (owner, repo) -> {
  if (owner.equals("Netflix") && repo.equals("feign")) {
    return Arrays.asList("stuarthendren"); // inspired this approach!
  } else {
    return Collections.emptyList();
  }
};

GitHub github = HystrixFeign.builder()
                            ...
                            .target(GitHub.class, "https://api.github.com", fallback);
```

#### Considering the cause-考虑原因

默认情况下，回退的原因记录为FINE级别。您可以通过创建自己的FallbackFactory来以编程方式检查原因。在许多情况下，原因将是FeignException，其中包括http状态。

这是使用FallbackFactory的示例：

```java
// This instance will be invoked if there are errors of any kind.
FallbackFactory<GitHub> fallbackFactory = cause -> (owner, repo) -> {
  if (cause instanceof FeignException && ((FeignException) cause).status() == 403) {
    return Collections.emptyList();
  } else {
    return Arrays.asList("yogi");
  }
};

GitHub github = HystrixFeign.builder()
                            ...
                            .target(GitHub.class, "https://api.github.com", fallbackFactory);
```

