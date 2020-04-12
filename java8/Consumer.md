```java
/**
 * 表示一个接受单个输入参数且不返回结果的操作。与大多数其他功能接口不同，消费者有望通过副作用进行操作。
 (副作用，是指传入参数T 可能会被修改)
 *
 * 这是一个功能接口，其功能方法为accept（Object）。
 
 这是一个功能接口，因此可以用作lambda表达式或方法引用的分配目标
 *
 * @param <T> 操作输入的类型
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Consumer<T> {

    /**
     * 对给定的参数执行此操作。
     *
     * @param t 输入的参数
     */
    void accept(T t);

    /**
     返回一个组成的使用者，该使用者依次执行此操作，然后执行after操作。如果执行任何一个操作均引发异常，则将其中继到组合操作的调用方。如果执行此操作引发异常，则将不执行after操作。
     *
     * @param 该操作后要执行的操作
     * @return 一个组合Consumer，该使用者依次执行此操作，然后执行after操作
     * @throws NullPointerException if {@code after} is null
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```