### Lambda

----

> 在java8之前，我们无法将函数作为参数传递给一个方法，也无法申明一个返回函数的方法。
>
> 在java8中，Lambda表达式为java添加了缺失的函数式编程特性，使我们能将函数当做一等公民对待。
>
> 在将函数作为一等公民对待的语言中，Lambda表达式的类型是函数(如Python、js)，<font color=red>但在java中，Lambda表达式是对象</font>，它们必须依赖于一类特别的对象类型——函数式接口。如：
> `WorkerInterface i = () -> System.out.println("hello");`

<a name="fenced-code-block">**代码简洁**</a>：函数式编程写出的代码简洁且意图明确，使用stream接口让你从此告别for循环

<a name="fenced-code-block">**多核友好**</a>：Java函数式编程使得编写并行程序从未如此简单，你需要的全部就是调用一下parallel()方法

* JDK7 匿名内部类写法

````java
new Thread(new Runnable(){	// 接口名
	@Override
	public void run(){	// 方法名
		System.out.println("Thread run()");
	}
}).start();
````
* JDK8 Lambda表达式写法

````java
// 省略接口名和方法名
new Thread(() -> System.out.println("Thread run()")).start();
````

> **Lambda类型推断机制**：在上下文信息足够的情况下，编译器可以推断出参数的类型，而不需要显式指明

#### 匿名内部类 & Lambda表达式：

* Lambda 表达式不是匿名内部类的简写
* 匿名内部类：任然是一个类，编译时会自动给名，编译之后，会生成一个新的class文件
* Lambda：编译之后，不会生成新的类，是当前类的私有方法。通过`invokedynamic `指令实现

#### Lambda表达式作用

* 传递行为，而不仅仅是值
* 提升抽象层次
* API重用性更好
* 更加灵活

####  使用Lambda重构某些设计模式

- 策略模式
- 模板方法
- 观察者模式
- 责任链模式
- 工厂模式

#### Lambda表达式实例

````java
List<Integer> list = Arrays.asList(1,2,3,4,5);
// 外部迭代：利用一个外部的迭代器，循环指向不同的元素实现
for (Integer i : list) {
    System.out.println(i * 2);
}

// 内部迭代：没有外部迭代器，通过函数式接口实现
list.forEach(new Consumer<Integer>() {
    @Override
    public void accept(Integer integer) {
        System.out.println(integer * 2);
    }
});

// 进一步简化
list.forEach((Integer i) -> System.out.println(i * 2));

// 进一步简化：通过Lanbda表达式创建函数式接口的实例
list.forEach(i -> System.out.println(i * 2));

// 再进一步简化：通过方法引用创建函数式接口的一个实例
list.forEach(System.out::println);
````



### FunctionalInterface

----

#### 函数式接口详解

* 如果一个接口只有一个<font color=red>抽象</font>方法，那么该接口就是一个函数式接口，`@FunctionalInterface`

* 如果在某个接口上声明了FunctionalInterface注解，那么编译器就会按照函数式接口的定义来要求该接口

* 如果某个接口只有一个抽象方法，但我们并没有给该接口申明FunctionalInterface接口，那么编译器依旧会将该接口看做是函数式接口

* If an interface declares an abstract method overriding one of the public methods Object, still is functional interface

  ````java
  @FunctionalInterface
  public interface FunctionalInterface2 {
      void test();
      //String myString(); // 编译无法通过，函数式接口中只能包含一个抽象方法
      String toString();   // 是Object中的public方法的override，不是一个抽象方法，因此这是合法的函数式接口
  }
  ````

* instances of functional interfaces can be created with *lambda expressions, method references, or constructor references*

  ````java
  // 通过Lanbda表达式创建函数式接口的实例
  list.forEach(i -> System.out.println(i * 2)); // i的类型 ==> 类型推断机制
  
  // 通过方法引用创建函数式接口的实例
  list.forEach(System.out::println);		// 按住command点击::会跳转到具体的实现类——Consumer
  
  // 通过构造方法引用创建函数式接口的实例
  list.forEach(FunctionInterface::new);
  ````

#### 函数式接口使用实例

````java
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
````

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
```

#### Function<T, R>

表示函数，针对指定的参数应用指定的函数。接收一个T类型的参数，返回一个R类型的结果

````java
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
````

````java
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
````



###  默认方法

----

- java8中的接口支持在声明方法的同时提供默认实现：
  - 方式一：java8允许在接口内声明静态方法
  - 方式二：java8新引入的默认方法，可为接口中的方法提供默认实现。参考Iterable接口

```java
public interface Iterable<T> {
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

- 默认方法是以源码兼容的模式向接口内添加实现的方法
- 默认方法的使用模式
  - 可选方法：接口中定义了方法，但是实现类不需要做任何实现，仅保留了个方法体。可以在接口中把该方法申明为默认方法，并提供默认实现，这样实现类就无需再去定义该方法的实现体
  - 行为的多继承：类型的多继承、通过正交方法精简接口、组合接口

#### <a name="fenced-code-block">解决冲突</a>

java8中，有可能出现一个类集成了多个方法(如默认方法)，而这些方法有可能使用同样的签名。这种情况下，类会如何选择方法执行呢？

```
1. 类中的方法优先级最高
2. 如果无法依据第一条进行判断，那么子接口的优先级更高:函数签名相同时，优先选择拥有最具体实现的默认方法的接口，即如果B继承了A，那么B就比A更加具体
3. 最后，如果还是无法判断，继承了多个接口的类必须通过显式覆盖和调用期望的方法，显式地选择使用哪一个默认方法的实现
```

```java
// B 和 A 中都定义了默认方法： default void hello(){ }
public class C implements B, A {
    void hello(){
        B.super.hello(); // 显式的选择调用指定接口B中的hello()
    }
}
```

#### <a name="fenced-code-block">重要概念</a>

- java8 中的接口可以通过默认方法和静态方法提供方法的代码实现
- 默认方法的开通以关键字`default`修饰，方法体与常规的类方法相同
- 向发布的接口添加抽象方法不是源码兼容的
- 默认方法的出现能帮助库的设计者以后向兼容的方式演进API
- 默认方法可以用于创建可选方法和行为的多继承
- 有办法解决由于一个类从多个接口中继承了拥有相同函数签名额方法而导致的冲突



### MethodReference

---

方法引用实际上一个Lambda表达式的语法糖。可将方法应用看作是一个”函数指针“，指向另一个方法.`System.out::println`指向`System.out.println()`。方法引用包括以下4中形式：

* 类名::静态方法

  ```java
  定义一个对象Student，包含name、score；在类中定义一个静态方法，根据score将list<Student>排序
  // 不推荐！既然定义到实体类中，那这个方法应该跟Student对象有关系: s1.studentSortByScore(s2)
  public static int studentSortByScore(Student s1, Student s2) { 
  	return s1.getScore() - s2.getScore();
  }
  list.sort(Student::studentSortByScore);	// list接口在jdk1.8中新增了默认方法sort，接收一个Comparator对象
  ```

* 引用名(对象名)::实例方法名

  ```java
  定义一个对象Student，包含name、score；在类中定义一个实例方法，根据score将list<Student>排序
  public int studentSortByScore(Student s1, Student s2) {
  	return s1.getScore() - s2.getScore();
  }
  Student student = new Student();
  list.sort(student::studentSortByScore);
  ```

* 类名::实例方法名

  ```java
  public int studentSortByScore(Student s) {  // 推荐！
  	return this.getScore() - s.getScore();
  }
  list.sort(Student::studentSortByScore); 
  // 类不能直接调用实例方法，
  // 因此这里的sort中的lambda表达式的第一个参数调用了StudentSortByScore方法，第二个参数是作为studentSortByScore的入参！！！
  ```

* 构造方法引用, 类名::new

  

### 扩展学习

* ***[Java8，JCF中新增的接口方法逐一详解](https://github.com/CarpenterLee/JavaLambdaInternals/blob/master/3-Lambda%20and%20Collections.md)***
