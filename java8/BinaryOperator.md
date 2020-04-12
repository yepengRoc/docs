```java
/**
 表示对两个相同类型的操作数的运算，产生与该操作数相同类型的结果。对于操作数和结果均为相同类型的情况，这是BiFunction的特殊化。
 * 这是一个函数式接口，函数式方法是 {@link #apply(Object, Object)}.
 *
 * @param <T> 操作数的类型和运算符的结果
 *
 * @see BiFunction
 * @see UnaryOperator
 * @since 1.8
 */
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    /**
     *返回一个BinaryOperator，它根据指定的Comparator返回两个元素中的较小者
     * @param <T> 比较器的输入参数的类型
     * @param 比较器，用于比较两个值
     * @return 一个BinaryOperator，根据提供的Comparator返回较小的操作数
     * @throws NullPointerException if the argument is null
     */
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    /**
     返回BinaryOperator，该BinaryOperator根据指定的Comparator返回两个元素中的较大者。
     * @param <T> 比较器的输入参数的类型
     * @param 比较器，用于比较两个值
     * @return 一个BinaryOperator，根据提供的Comparator返回较大的操作数
     * @throws NullPointerException if the argument is null
     */
    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```

