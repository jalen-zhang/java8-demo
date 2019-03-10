[TOC]



### Optional
java 一直试图避免让程序猿意识到指针的存在，唯一的例外是：null 指针！

#### <a name="fenced-code-block">java.util.Optional<T></a>

变量存在时，Optional类只是对类的简单封装；变量不存在时，缺失的值会被建模成一个“空”的Optional对象，由方法`Optional.empty()`返回。

如果你尝试引用一个null，一定会触发NPE，不过使用Optional.empty()就完全没事儿，它返回的是一个Optional类的有效对象。

引入Optional类的意图并非要消除每一个null引用。与此相反，它的目标是帮助开发者设计出普适的API，让开发者看到方法签名，就能了解它是否接受一个Optional的值。这种强制会让你更积极地将变量从Optional中解包出来，直面缺失的变量值。

#### <a name="fenced-code-block">应用Optional的几种模式</a>

* 创建Optional对象

  Optional对象是一个value-base class，它的构造方式私有的，只能通过以下三种静态工厂方法创建：

````java
Optional<Car> optCar = Optional.empty();  // 声明一个空的Optional
Optional<Car> optCar = Optional.of(car);  // 依据一个非空值创建Optional。如果car为null，则抛出NPE
Optional<Car> optCar = Optional.ofNullable(car);  // 可接受null的Optional。如果car是null，则得到的Optional对象就是个空对象，不是null
````

* 使用map从Optional对象中提取和转换值

````java
Optional<Insurance> opt = Optional.ofNullable(insurance);
Optional<String> name = opt.map(Insurance::getName);  // 如果opt为空，则什么都不做
name.ifPresent(n -> System.out.println(n)); // 不要使用 if(name.isPresent()) {}
````

* 使用flatMap链接Optional对象

````java
private String getCarInsuranceName(Person person) {
	return person.filter(p -> p.getAge > 5)	// 使用filter对Optional对象过滤
				.flatMap(Person::getCar)	// map()返回的是Optional<Optional<Car>>，为嵌套optional
				.flatMap(Car::getInsurance)	// flatMap()返回的是Optional<Car>
				.map(Insurance::getName)	// Optional对象不支持getName()
				.orElse("Unknown");			// 如果Optional的结果值为空，设置默认值
}
````
![使用Optional解引用串接的Person/Car/Insurance](md_imgs/optional_check.png)

* 基本类型：OptionalInt、OptionalDouble、OptionalLong
	* 扩展提供了 `public int getAsInt()`方法；
	* 基本类型的Optional不支持map()、flatMap()、filter()方法；

#### <a name="fenced-code-block">解引用Optional对象</a>

* `get()` : 最简单但又最不安全
* `orElse(T other)` : Optional对象不存在值时，提供一个默认值
* `orElseGet(Supplier other)` ：Optional对象不存在值时，执行other方法
* `orElseThrow(Supplier excpOther)` ： Optional对象不存在值时，抛出定制的other异常
* `ifPresent(Consumer other)`：Optional对象值存在时，执行作为参数传入的other方法，否则不做任何操作

<br>

### java.time.*
java.util.Date: 年从1900开始，月从0开始；时区支持不够

java.util.Calendar：月从0开始

DateFormat：不是线程安全的

Joda-Time:第三方库，java8参考了较多

#### <a name="fenced-code-block">java8提供的日期和时间</a>

* `LocalDate`该类的实例是一个不可变对象。只提供了简单的日期，不含时间信息，不含时区信息。

* `LocalTime`表示时间，如19:15:30
* `LocalDateTime`是LocalDate和LocalTime的合体，不含时区信息
* `Instant`面向机器，从1970.1.1开始所经历的秒数进行计算
* `Duration`主要用于以秒和纳秒衡量时间的长短
* `Period`以年月日的方式对多个时间单位建模

````java
LocalDate date = LocalDate.of(2018, 9, 11);
System.out.println(date.getYear());         // date.get(ChronoField.YEAR) => 2018
System.out.println(date.getMonth());        // SEPTEMBER
System.out.println(date.getMonthValue());   // date.get(ChronoField.MONTH_OF_YEAR) => 9
System.out.println(date.getDayOfYear());    // 254
System.out.println(date.getDayOfMonth());   // date.get(ChronoField.DAY_OF_MONTH) => 11
System.out.println(date.getDayOfWeek());    // TUESDAY

System.out.println(LocalDate.now());        		// 2018-09-25
System.out.println(LocalDate.ofYearDay(2018, 255)); // 2018-09-12

LocalDate now = LocalDate.parse("2018-09-25");
LocalTime time = LocalTime.parse("19:54:38.840");
LocalDateTime now1 = LocalDateTime.parse("2011-12-03T10:15:30", DateTimeFormatter.ISO_LOCAL_DATE_TIME);
LocalDateTime now2 = LocalDateTime.parse("2011/12/03 19:15:30", DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss"));

System.out.println(now2.toLocalDate()); // 2011-12-03
System.out.println(now2.toLocalTime()); // 19:15:30
````
<br>

### CompletableFuture

不希望因为等待某些服务的响应，阻塞应用程序的运行，浪费数十亿宝贵的CPU时钟周期

分支/合并框架、并行流，是实现**<font color="red">并行处理</font>**的宝贵工具；CompletableFuture是实现**<font color="red">并发处理</font>**的利器！

CompletableFuture 实现了 Future 接口！

![并发与并行](md_imgs/bingxing.png)
![](md_imgs/thread_num.png)
#### <a name="fenced-code-block">Future接口</a>
Future接口在JDK5引入，设计初衷是对将来某个时刻会发生的结果建模，异步计算，返回计算结果的一个引用。相比Thread，Future更易用；相比JDK8提供的CompletableFuture，Future又存在较大的局限性。

#### <a name="fenced-code-block">将同步方法变为异步方法</a>

* 同步API：传统方法调用的别称，方法被调用后，调用方要等待方法执行的结果，拿到结果后才能执行后续动作。阻塞式调用
* 异步API：方法被调用后会直接返回，将它的剩余计算任务交由另一个线程去做，该线程和调用方式异步的，结果回调。非阻塞式调用


**对集合进行并行计算的两种方式：**

* 计算密集型任务：推荐使用Stream接口

* I/O密集型任务(包括网络连接)：推荐使用CompletableFuture，自己设定线程数。如果流的处理过程中发生io等待，流的延迟特性导致不易判断什么时候发生的等待

#### <a name="fenced-code-block">对多个异步任务进行流水线操作</a>

````java
Executor executor = Executors.newFixedThreadPool(
        Runtime.getRuntime().availableProcessors(),//Math.min(shops.size, 100),
        r -> {
            Thread t = new Thread();
            t.setDaemon(true); // 守护线程
            return t;
        }
);

CompletableFuture<Double> futurePriceUSD =
        // 创建第一个任务：根据产品获取价格
        CompletableFuture.supplyAsync(() -> this.getPrice(product))
        // 合并操作比较简单，同步处理即可。否则可通过 thenCombineAsync() 异步处理合并操作
        .thenCombine(
                // 创建第二个独立任务：查询美元和欧元之间的转换汇率
                CompletableFuture.supplyAsync(() -> remoteService.getRate(MONEY.EUR, MONEY.USD), executor),
                // 定义了两个CompletableFuture完成计算后，结果如何合并
                (price, rate) -> price * rate
        );
````

> Java的线程分为两种：User Thread(用户线程)、Daemon Thread(守护线程)<br>
> * 只要当前JVM实例中尚存任何一个非守护线程没有结束，守护线程就全部工作；<br>
> * 只有当最后一个非守护线程结束是，守护线程随着JVM一同结束工作，Daemon作用是为其他线程提供便利服务，守护线程最典型的应用就是GC，他就是一个很称职的守护者。<br>
> 因为没有了被守护者，Daemon也就没有工作可做了，也就没有继续运行程序的必要了。