## 规约(reduction operation)

* `collect()`  

  收集器，是一个方法

* `Collector`

  - Collector 作为collect方法的参数。是一个接口，它是一个**可变(mutable)**的汇聚操作，将输入元素累积到一个可变的结果容器(mutable result container)中；它会在所有元素都处理完毕之后，将累积的结果转换为一个最终的表示(这是一个可选操作)；它支持串行与并行两种方式。

  ```java
  A Collector is specified by four functions that work together to accumulate entries into a mutable result container, and optionally perform a final transform on the result.  They are: 
     <1>creation of a new result container ({@link #supplier()})
     <2>incorporating(合并) a new data element(流中的元素) into a result container ({@link #accumulator累加器()})
     <3>combining two result containers into one ({@link #combiner()})
     <4>performing an optional final transform on the container ({@link #finisher()})
       
   * A sequential implementation of a reduction using a collector would
   * create a single result container using the supplier function, and invoke the
   * accumulator function once for each input element.  A parallel implementation
   * would partition the input, create a result container for each partition,
   * accumulate the contents of each partition into a subresult for that partition,
   * and then use the combiner function to merge the subresults into a combined
   * result.
   *     A a2 = supplier.get();
   *     accumulator.accept(a2, t1);
   *     A a3 = supplier.get();
   *     accumulator.accept(a3, t2);
   *     R r2 = finisher.apply(combiner.apply(a2, a3));
  // 构造方式
  Collector<Widget, ?, TreeSet<Widget>> intoSet = Collector.of(
  					TreeSet::new, 
    				TreeSet::add,
            (left, right) -> { left.addAll(right); return left; }
  );
  // Performing a reduction operation with a Collector should produce a result equivalent to:等价于
   *     R container = collector.supplier().get();  // 获得一个R类型结果容器
   *     for (T t : data) // 遍历流中每个T类型的参数
   *         collector.accumulator().accept(container, t); // 将每个t累加到结果容器中
   *     return collector.finisher().apply(container); // 返回最终结果
  ```

  - 为了保证并行操作和串行操作结果的等价性，Collector函数需要满足两个条件：identity(同一性)和associativity(结合性)。
    - identity: a == combiner.apply(a, supplier.get())
    - associativity

* `Collectors` 

  `public final class Collectors`，是一个final修饰的工厂类，实现了Collector接口中的一系列有用的mutable reduce operation。

* `reduce()` 规约

  这是一个 **最终操作** ，允许通过指定的函数来将stream中的多个元素规约为一个元素，规约后的结果是通过Optional 接口表示的：

  ```java
          //测试 Reduce (规约)操作
          Optional<String> reduced = stringList
                          .stream()
                          .sorted()
                          .reduce((s1, s2) -> s1 + "#" + s2);
          reduced.ifPresent(System.out::println);
  ```

  **译者注：** 这个方法的主要作用是把 Stream 元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合。从这个意义上说，字符串拼接、数值的 sum、min、max、average 都是特殊的 reduce。例如 Stream 的 sum 就相当于`Integer sum = integers.reduce(0, (a, b) -> a+b);`也有没有起始值的情况，这时会把 Stream 的前面两个元素组合起来，返回的是 Optional。

  ```java
  // 字符串连接，concat = "ABCD"
  String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat); 
  // 求最小值，minValue = -3.0
  double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min); 
  // 求和，sumValue = 10, 有起始值
  int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
  // 求和，sumValue = 10, 无起始值
  sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
  // 过滤，字符串连接，concat = "ace"
  concat = Stream.of("a", "B", "c", "D", "e", "F").
   		filter(x -> x.compareTo("Z") > 0).
   		reduce("", String::concat);
  ```

  上面代码例如第一个示例的 reduce()，第一个参数（空白字符）即为起始值，第二个参数（String::concat）为 BinaryOperator。这类有起始值的 reduce() 都返回具体的对象。而对于第四个示例没有起始值的 reduce()，由于可能没有足够的元素，返回的是 Optional，请留意这个区别。更多内容查看： [IBM：Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)

* `sum()`、`max()`、`count()`...

参考：[流的规约操作汇总介绍](https://github.com/CarpenterLee/JavaLambdaInternals/blob/master/5-Streams%20API(II).md)

#### <a name="fenced-code-block">收集器：定义collect用来生成结果集合的标准</a>

**1. 将流元素规约和汇总为一个值**

* `Collectors.counting()`

````java
long count1 = menu.stream().collect(Collectors.counting());
long count2 = menu.stream().count();
````

* `Collectors.minBy()` / `.maxBy()`

````java
menu.stream().collect(Collectors.minBy(Comparator.comparingInt(Dish::getCalories))).ifPresent(System.out::println);
````
* `Collectors.summingInt()` / `.summingDouble()` / `.summingLong()`

````java
long sum = menu.stream().collect(Collectors.summingInt(Dish::getCalories));
````
* `Collectors.averagingInt()` / `.averagingDouble()` / `.averagingLong()`

````java
double avr = menu.stream().collect(Collectors.averagingInt(Dish::getCalories));
````
* `Collectors.summarizingInt()` / `.summarizingDouble()` / `.summarizingLong()`

````java
IntSummaryStatistics statistics = menu.stream().collect(Collectors.summarizingInt(Dish::getCalories));
System.out.println(statistics.getAverage());   // 466.6666666666667
System.out.println(statistics.toString());     // IntSummaryStatistics{count=9, sum=4200, min=120, average=466.666667, max=800}
````

* `Collectors.join()`

````java
String names = menu.stream().map(Dish::getName).collect(Collectors.joining(","));
````

注：*以上规约操作，其实是常用规约操作的特殊化(源码实现)，为了方便使用、可读性*，**下面来看规约操作的一般情况：**

* `reducing(U identity, Function<? super T, ? extends U> mapper, BinaryOperator<U> op)`

identity：初始值；mapper：转换函数；op：累积函数。

利用累积函数，把一个初始化为初始值的累加器，和把转换函数应用到流中每个元素上得到的结果不断迭代合并起来。

````java
long commonSum = menu.stream().collect(Collectors.reducing(0, Dish::getCalories, (i, j) -> i + j));
long commonSum = menu.stream().collect(Collectors.reducing(0, Dish::getCalories, Integer::sum));
````

**2. 元素分组**

* `Collectors.groupingBy()`

````java
Map<Type, List<Dish>> group = menu.stream().collect(Collectors.groupingBy(Dish::getType));
// {FISH=[prawns, salmon], OTHER=[french fries, rice, season fruit, pizza], MEAT=[pork, beef, chicken]}

// 自定义分组规则
Map<CaloricLevel, List<Dish>> group1 = menu.stream().collect(Collectors.groupingBy(d -> {
    if (d.getCalories() <= 400) {
        return CaloricLevel.DIET;
    } else if (d.getCalories() <= 700) {
        return CaloricLevel.NORMAL;
    } else {
        return CaloricLevel.FAT;
    }
}));
````

多级分组：把第二个groupBy收集器传递给第一个收集器，实现多级分组

````java
Map<Type, Map<CaloricLevel, List<Dish>>> group2 = menu.stream().collect(Collectors.groupingBy(Dish::getType,
        Collectors.groupingBy(d -> {
            if (d.getCalories() <= 400) {
                return CaloricLevel.DIET;
            } else if (d.getCalories() <= 700) {
                return CaloricLevel.NORMAL;
            } else {
                return CaloricLevel.FAT;
            }
        })
));
// {FISH={DIET=[prawns], NORMAL=[salmon]}, OTHER={DIET=[rice, season fruit], NORMAL=[french fries, pizza]}, MEAT={FAT=[pork], DIET=[chicken], NORMAL=[beef]}}
````

按子组收集数据

````java
Map<Type, Long> group3 = menu.stream().collect(Collectors.groupingBy(Dish::getType, Collectors.counting()));
// {FISH=2, OTHER=4, MEAT=3}
````

````java
Map<Type, Optional<Dish>> group4 = menu.stream().collect(Collectors.groupingBy(
        Dish::getType,	// 以此对流中元素进行一级分组
        Collectors.maxBy(Comparator.comparing(Dish::getCalories)) // 对同一分组中的流元素进行进一步规约操作
));
// {FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
````

`Collectors.collectingAndThen()`、`.collectingAndThen()`

````java
Map<Type, Dish> group5 = menu.stream().collect(Collectors.groupingBy(
        Dish::getType,
        Collectors.collectingAndThen(
                Collectors.maxBy(Comparator.comparing(Dish::getCalories)),
                Optional::get
        )
));
// {FISH=salmon, OTHER=pizza, MEAT=pork}

````

```java
// select name, count(*) from User group by name;
Map<String, Long> count = users.stream().collect(Collectors.groupingBy(User::getName, Collectors.counting()));
System.out.println(count);

Map<String, Double> average = users.stream().collect(Collectors.groupingBy(User::getName, Collectors.averagingLong(User::getAge)));
System.out.println(average);

Map<Boolean, List<User>> partition = users.stream().collect(Collectors.partitioningBy(user -> user.getAge() > 20));
System.out.println(partition);
```

*嵌套收集器分组效果流程如下图：*
![嵌套收集器分组效果流程](resources/group_muti.png)

**3. 元素分区**


* `Collectors.partitioningBy(predicate)`、`.partitioningBy(predicate, downstream)`

````java
Map<Boolean, List<Dish>> partition = menu.stream()
        .collect(
                Collectors.partitioningBy(Dish::isVegetarian)
        );
// {false=[pork, beef, chicken, prawns, salmon], true=[french fries, rice, season fruit, pizza]}

Map<Boolean, Map<Type, List<Dish>>> partiton2 = menu.stream()
        .collect(
                Collectors.partitioningBy(Dish::isVegetarian, Collectors.groupingBy(Dish::getType))
        );
// {false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]}, true={OTHER=[french fries, rice, season fruit, pizza]}}

// 分别输出20以内的质数和非质数
Map<Boolean, List<Integer>> partitions = IntStream.rangeClosed(2, n).boxed()
        .collect(Collectors.partitioningBy(
                item -> IntStream.rangeClosed(2, (int) Math.sqrt((double)item)).noneMatch(i -> item % i == 0)
        ));
````