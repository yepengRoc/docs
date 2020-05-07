```java
/**
 * 代表结果的提供者。
 * 不需要每次调用supplier都返回新的或不同的结果。
 * @param <T> 该supplier提供的结果类型
 * @since 1.8
 */
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

