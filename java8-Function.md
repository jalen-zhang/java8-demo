[TOC]

### Function<T, R>

---

T: the type of the input to the function

R: the type of the result to the function

<font color=red>**传递的是行为，不是值！**</font>

高阶函数：如果一个函数接收一个函数作为参数，或者返回一个函数作为返回值，那这个函数就是高阶函数。

#### R apply(T t)

Applies this function to the given argument.

```java
// toUpperCase(),是一个实例方法，它的入参是该方法的调用者！所以满足Function<T, R>
Function<String, String> function = String::toUpperCase;
```

#### compose

```java
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}
```

compose方法接收一个Function参数before，该方法说明是返回一个组合的函数，首先会应用输入参数before的apply；然后把before.apply的结果作为参数，应用到当前对象(因为compose方法是对象的实例方法 )的apply方法，*换句话说就是先执行before对象的apply，再执行当前对象的apply，将两个执行逻辑串起来*。

```JAVA
public int compute1(int i, Function<Integer,Integer> func1, Function<Integer,Integer> func2){
    // [看源码]先执行func2.apply() => 25; 再把25作为入参，执行func1.apply() => 25*2=50
    return func1.compose(func2).apply(i);
}
functionTest.compute1(5,i -> i * 2,i -> i * i); // 50
```

#### andThen

```java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```

andThen方法接收一个Function参数after，与compose方法相反，它是*先执行当前对象的apply方法，再将执行结果作为after的apply的入参*。

```java
public int compute2(int i, Function<Integer,Integer> func1,Function<Integer,Integer> func2){
    // [看源码]先执行当前对象func1的apply() => 5*2=10; 再把10作为func2的入参，执行func2.apply() => 10*10=100
    return func1.andThen(func2).apply(i);
}
functionTest.compute2(5,i -> i * 2,i -> i * i); //100
```



### BiFunction<T, U, R>

---

bidirectional, 双向的

Represents a function that accepts two arguments and produces a result. 是Function的一种特化形式。

* T: the type of the first argument to the function

* U: the type of second argument to the function

* R: the type of the result of the function

#### R apply(T t, U u);

应用场景：对两个输入的数字做加减乘除，将结果返回

```java
public int compute3(int a, int b, BiFunction<Integer, Integer, Integer> biFunction) {
    return biFunction(a, b);
}
functionTest.compute3(1, 2, (v1, v2) -> v1 + v2); // 3
functionTest.compute3(1, 2, (v1, v2) -> v1 * v2); // 2
```

#### andThen

```java
default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t, U u) -> after.apply(apply(t, u));
}
```

andThen方法接收一个Function(不是BiFunction！因为当前对象的BiFunction执行完只会返回一个结果，而BiFunction需要两个入参)参数after，它是先执行当前对象BiFunction的apply方法(两个入参一个结果)，再将执行结果作为after的apply的入参。

```java
public int compute4(int a, int b, 
                    BiFunction<Integer, Integer, Integer> func1, 
                    Function<Integer, Integer> func2) {
    // func1.andThen(func2) => func1.apply(v1,v2)=r => func2.apply(r)
    return func1.andThen(func2).apply(a, b);
}
// (v1, v2) -> v1 + v2,两个输入参数v1和v2，v1+v2作为返回结果
functionTest.compute4(1, 2, (v1, v2) -> v1 + v2, (v) -> v * v); // 9
```

- 为什么BiFunction没有compose方法？

  如果提供了的话，参数类型肯定是BiFunction(因为BiFunction接收两个参数，而Function只有一个参数)，而作为<font color=red>参数的BiFunction</font>的执行结果只有一个，再次对这个结果执行<font color=red>当前对象BiFunction</font>的apply时会报错(BiFunction接收两个参数)。