```java
/*
 
 表示一个接受两个参数并产生结果的函数。这是Function的两个领域。这是一个功能接口，其功能方法为apply（Object，Object）。
	这是一个功能接口，因此可以用作lambda表达式或方法引用的分配目标。

 * @param <T> 第一个参数
 * @param <U> 第二个参数
 * @param <R> 返回结果类型
 *
 * @see Function
 * @since 1.8
 */
@FunctionalInterface
public interface BiFunction<T, U, R> {

    /**
     * 将此函数应用于给定参数。
     *
     * @param t 第一个参数
     * @param u 第二个参数
     * @return 结果
     */
    R apply(T t, U u);

    /**
    返回一个组合函数，该函数首先将此函数应用于其输入，然后将after函数应用于结果。如果对任一函数的求值抛出异常，则将其中继到组合函数的调用方。
     * @param <V> after函数和组合函数的输出类型
     * @param after 应用此功能后要应用的功能
     * @return 一个组合函数，首先应用此函数，然后应用after函数
     * applies the {@code after} function
     * @throws NullPointerException if after is null
     */
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```

