

### FunctionalInterface

------

#### 函数式接口详解

- 如果一个接口只有一个<font color=red>抽象</font>方法，那么该接口就是一个函数式接口，`@FunctionalInterface`

- 如果在某个接口上声明了FunctionalInterface注解，那么编译器就会按照函数式接口的定义来要求该接口

- 如果某个接口只有一个抽象方法，但我们并没有给该接口申明FunctionalInterface接口，那么编译器依旧会将该接口看做是函数式接口

- If an interface declares an abstract method overriding one of the public methods Object, still is functional interface

  ```java
  @FunctionalInterface
  public interface FunctionalInterface2 {
      void test();
      //String myString(); // 编译无法通过，函数式接口中只能包含一个抽象方法
      String toString();   // 是Object中的public方法的override，不是一个抽象方法，因此这是合法的函数式接口
  }
  ```

- instances of functional interfaces can be created with *lambda expressions, method references, or constructor references*

  ```java
  // 通过Lanbda表达式创建函数式接口的实例
  list.forEach(i -> System.out.println(i * 2)); // i的类型 ==> 类型推断机制
  
  // 通过方法引用创建函数式接口的实例
  list.forEach(System.out::println);		// 按住command点击::会跳转到具体的实现类——Consumer
  
  // 通过构造方法引用创建函数式接口的实例
  list.forEach(FunctionInterface::new);
  ```

#### 函数式接口使用实例

```java
// define a functional interface
@FunctionalInterface
public interface WorkerInterface {
    void doSomeWork();
}

// use this functional interface
class WorkerInterfaceTest {
    public void execute(WorkerInterface worker) {
        worker.doSomeWork();
    }

    public static void main(String [] args) {
        WorkerInterfaceTest test = new WorkerInterfaceTest();
	
        //JDK7: invoke doSomeWork using Annonymous class
        test.execute(new WorkerInterface() {
            @Override
            public void doSomeWork() {
                System.out.println("Worker invoked using Anonymous class");
            }
        });

        //JDK8: invoke doSomeWork using Lambda expression
        test.execute(() -> System.out.println("Worker invoked using Lambda expression"));

        WorkerInterface i = () -> System.out.println("hello");
        System.out.println(i.getClass()); 
	// class com.jianlongz.java8.WorkerInterfaceTest$$Lambda$2/1174361318
	
        System.out.println(i.getClass().getSuperclass()); 
	// class java.lang.Object
	
        System.out.println(i.getClass().getInterfaces()[0]);
	// interface com.jianlongz.java8.WorkerInterface
    }
}
```

#### 常用的函数式接口

---

以下方法均来自JDK `package java.util.function`

* Consumer\<T>, BiConsumer<T, U>

  `void accept(T t);` 根据给定的参数T，应用指定函数，返回结果为void

* Function<T, R>, BiFunction<T, U, R>

  `R apply(T t);` 根据给定的参数T，应用指定函数，返回结果为R

* Predicate<\T>, BiPredicate<T, U>

  `boolean test(T t);` 根据给定参数T，评估此谓词，返回结果为boolean

* Supplier<\T>

  `T get();` 不需要给定参数，应用指定函数，返回结果为T

#### Consumer\<T>

针对指定的元素执行指定的动作。只接收一个T类型的参数，并且没有返回值，有副作用(side-effects)

```java
// Consumer 源码
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

// Consumer 使用实例
List<Integer> list = Arrays.asList(1,2,3,4,5);
list.forEach(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
        System.out.println(integer * 2);	// 2,4,6,8,10
    }
});

// forEach的入参就是Consumer，源码如下：
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

#### Function<T, R>

<font color=red>**传递的是行为，不是值！**</font>

表示函数，针对指定的参数应用指定的函数。接收一个T类型的参数，返回一个R类型的结果

```java
// Stream.map() 方法的入参就是Function，方法体如下：
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
	
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

```java
Function<String, String> function = str -> str.toUpperCase();
Function<String, String> function = String::toUpperCase;

// function传递的是行为，这个行为需要一个Integer类型的参数(value)，会返回一个String类型的结果
public String compute(int value, Function<Integer, String> function) {
    return function.apply(value);
}

public static void main(String[] args) {
    // 匿名内部类
    System.out.println(functionTest.compute(1, new Function<Integer, String>() {
        @Override
        public String apply(Integer integer) {
            return "index is: " + integer * 2;
        }
    }));
    // statement：value -> {return value * 2;}
    System.out.println(functionTest.compute(2, value -> {return "index is: " + value * 2;}));
    
    // expression：value -> value * 2
    System.out.println(functionTest.compute(3, value -> "index is: " + value * 2));
}
```

##### R apply(T t)

Applies this function to the given argument.

```java
// toUpperCase(),是一个实例方法，它的入参是该方法的调用者！所以满足Function<T, R>
Function<String, String> function = String::toUpperCase;
```

##### compose(Function before)

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

##### andThen(Function after)

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

#### BiFunction<T, U, R>

bidirectional, 双向的

Represents a function that accepts two arguments and produces a result. 是Function的一种特化形式。

* T: the type of the first argument to the function

* U: the type of second argument to the function

* R: the type of the result of the function

##### R apply(T t, U u);

应用场景：对两个输入的数字做加减乘除，将结果返回

```java
public int compute3(int a, int b, BiFunction<Integer, Integer, Integer> biFunction) {
    return biFunction(a, b);
}
functionTest.compute3(1, 2, (v1, v2) -> v1 + v2); // 3
functionTest.compute3(1, 2, (v1, v2) -> v1 * v2); // 2
```

##### andThen

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

<BR>

- 为什么BiFunction没有compose方法？

  如果提供了的话，参数类型肯定是BiFunction(因为BiFunction接收两个参数，而Function只有一个参数)，而作为<font color=red>参数的BiFunction</font>的执行结果只有一个，再次对这个结果执行<font color=red>当前对象BiFunction</font>的apply时会报错(BiFunction接收两个参数)。

* 高阶函数：如果一个函数接收一个函数作为参数，或者返回一个函数作为返回值，那这个函数就是高阶函数。

<br>

#### Predicate<T>

```java
// filter 方法的入参就是Predicate，方法体定义如下：
Stream<T> filter(Predicate<? super T> predicate);
```

