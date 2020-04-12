```java


/**
 * 表示一个参数的谓词（布尔值函数）。这是一个功能接口，其功能方法为test（Object）。
 *
 * @param <T> the type of the input to the predicate
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Predicate<T> {

    /**
     * 根据给定参数评估此谓词。 true或false
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);

     /**
     * 返回表示该谓词与另一个谓词的短路逻辑与的组合谓词。在评估组合谓词时，如果该谓词为假，则不会评估另一个谓词。
    在评估任一谓词过程中引发的任何异常都会中继给调用者；如果对此谓词的求值抛出异常，则不会对另一个谓词求值。
     *
     * @param other a predicate that will be logically-ANDed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * AND of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * 返回表示此谓词逻辑否定的谓词。
     * @return a predicate that represents the logical negation of this
     * predicate
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     *返回一个组成的谓词，该谓词表示此谓词和另一个谓词的短路逻辑或。在评估组合谓词时，如果该谓词为true，则不会评估另一个谓词。
     *
   在评估任一谓词过程中引发的任何异常都会中继给调用者；如果对此谓词的求值抛出异常，则不会对另一个谓词求值。
     * @param other a predicate that will be logically-ORed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * OR of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     *返回一个谓词，该谓词根据Objects.equals（Object，Object）测试两个参数是否相等。
     *
     * @param <T> the type of arguments to the predicate
     * @param targetRef the object reference with which to compare for equality,
     *               which may be {@code null}
     * @return a predicate that tests if two arguments are equal according
     * to {@link Objects#equals(Object, Object)}
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

