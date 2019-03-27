#### 接口设计

java8之前的接口(Interface)，只能包含静态方法/属性，不能包含方法的实现。通常情况下，我们都会定义一个该接口的工具辅助类，该类定义了与接口实例协作的很多静态方法。比如Collections就是处理Collection的工具类。

java8对接口做了扩展，支持以下操作：

* 允许在接口内申明静态方法
* 允许在接口内创建默认方法

在java8中，由于静态方法可以存在于接口内部，原来的接口辅助工具类就没存在的必要了，可以把这些静态方法移到接口内部。

#### 默认方法试图解决什么问题？

向接口添加新方法是二进制兼容的，这意味着如果不重新编译该类，即使不实现新的方法，现有类的实现依然可以正常运行。但是如果再次对接口进行编译，就会抛出一个运行时异常。默认方法是以源码兼容的方式改进API。

对接口设计者来说，设计的局限性以及维护成本

对接口使用方来说，版本控制

向后兼容

#### 构造 no side-effects 方法

**no side-effects**: 如果一个方法既不修改它内嵌类的状态，也不修改其他对象的状态，使用return返回所有的计算结果，那么该方法就可以称其为纯粹的或者无副作用的。

no side-effects 至少需要满足：

* 不改变对调用者可见的变量(满足函数式的方法都只能修改本地变量，它引用的对象都应该是不可修改的)
* 不抛出异常(善用 `Optional<T>`)
* 不进行I/O

不可变对象：一旦完成初始化就不会被任何方法修改状态。此时可以放心地共享该对象，无需保留任何副本，也是线程安全的。

如果构成系统的各个方法都是 no side-effects的，那么该系统就能在完全无锁的情况下，使用多核的并发机制，因为任何一个方法都不会对其他方法造成干扰。

#### 引用透明性

如果一个函数只要传递同样的参数值，总是返回同样的结果，那这个函数就是引用透明的。

在函数式编程中，你应该选择使用引用透明的函数。

#### 高阶函数

high-order function，满足以下任一条件即可：

* 接收至少一个函数作为参数
* 返回的结果是一个函数

理想状态下，作为参数的函数，应该是no side-effects的，避免产生无法预知的错误！

#### 科里化

#### 静态导入

```java
import static java.util.stream.Collectors.*;

users.stream().collect(groupingBy(User::getName, counting()));
```

####  精简流的调用链

```java
collection.stream().forEach() → collection.forEach()
collection.stream().collect(toList/toSet/toCollection()) → new CollectionType<>(collection)
collection.stream().toArray() → collection.toArray()
Arrays.asList().stream() → Arrays.stream() or Stream.of()
IntStream.range(0, array.length).mapToObj(idx -> array[idx]) → Arrays.stream(array)
IntStream.range(0, list.size()).mapToObj(idx -> list.get(idx)) → list.stream()
Collections.singleton().stream() → Stream.of()
Collections.emptyList().stream() → Stream.empty()
stream.filter().findFirst().isPresent() → stream.anyMatch()
stream.collect(counting()) → stream.count()
stream.collect(maxBy()) → stream.max()
stream.collect(mapping()) → stream.map().collect()
stream.collect(reducing()) → stream.reduce()
stream.collect(summingInt()) → stream.mapToInt().sum()
stream.mapToObj(x -> x) → stream.boxed()
stream.map(x -> {...; return x;}) → stream.peek(x -> ...)
!stream.anyMatch() → stream.noneMatch()
!stream.anyMatch(x -> !(...)) → stream.allMatch()
stream.map().anyMatch(Boolean::booleanValue) -> stream.anyMatch()
IntStream.range(expr1, expr2).mapToObj(x -> array[x]) -> Arrays.stream(array, expr1, expr2)
Collection.nCopies(count, ...) -> Stream.generate().limit(count)
stream.sorted(comparator).findFirst() -> Stream.min(comparator)
  
Note that the replacements semantic may have minor difference in some cases. For example, Collections.synchronizedList(...).stream().forEach() is not synchronized while Collections.synchronizedList(...).forEach() is synchronized. Or collect(Collectors.maxBy()) would return an empty Optional if the resulting element is null while Stream.max() will throw NullPointerException in this case.
```



