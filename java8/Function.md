```java

/**
 * 接收一个参数返回一个结果

 * @param <T> 输入参数类型
 * @param <R> 返回结果类型
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {

    /**
     * 应用此function到给定的参数
     * @param t 输入参数
     * @return 返回结果
     */
    R apply(T t);

    /**
     * 返回一个组合函数，该函数首先将before函数应用于其输入，然后将该函数应用于结果。如果对任一函数的求值抛			出异常，则将其中继到组合函数的调用方。
     * @param <V> the type of input to the {@code before} function, and to the
     *           composed function
     * @param before 应用此功能之前要应用的功能
     * @return 一个组合函数，首先应用before函数，然后再应用此函数
     * @throws NullPointerException if before is null
     *
     * @see #andThen(Function)
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
      //先执行before动作获得一个结果，让你后将获得的结果应用于compose function
        return (V v) -> apply(before.apply(v));
    }

    /**
     返回一个组合函数，该函数首先将此函数应用于其输入，然后将after函数应用于结果。如果对任一函数的求值抛出异常，则将其中继到组合函数的调用方。
     * @param <V> the type of output of the {@code after} function, and of the
     *           composed function
     * @param after 应用此功能后要应用的功能
     * @return 一个组合函数，首先应用此函数，然后应用after函数
     * @throws NullPointerException if after is null
     * @see #compose(Function)
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
      //首先应用andThen function,得到一个结果，然后将此结果应用于after
        return (T t) -> after.apply(apply(t));
    }

    /**
     * 返回始终返回其输入参数的函数。
     *
     * @param <T> 函数的输入和输出对象的类型
     * @return 始终返回其输入参数的函数
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

