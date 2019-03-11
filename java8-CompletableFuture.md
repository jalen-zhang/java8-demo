### CompletableFuture

不希望因为等待某些服务的响应，阻塞应用程序的运行，浪费数十亿宝贵的CPU时钟周期

分支/合并框架、并行流，是实现**<font color="red">并行处理</font>**的宝贵工具；CompletableFuture是实现**<font color="red">并发处理</font>**的利器！

CompletableFuture 实现了 Future 接口！

![并发与并行](/Users/apple/jalen/java8/resources/bingxing.png)

#### <a name="fenced-code-block">Future接口</a>

Future接口在JDK5引入，设计初衷是对将来某个时刻会发生的结果建模，异步计算，返回计算结果的一个引用。相比Thread，Future更易用；相比JDK8提供的CompletableFuture，Future又存在较大的局限性。

#### <a name="fenced-code-block">将同步方法变为异步方法</a>

- 同步API：传统方法调用的别称，方法被调用后，调用方要等待方法执行的结果，拿到结果后才能执行后续动作。阻塞式调用
- 异步API：方法被调用后会直接返回，将它的剩余计算任务交由另一个线程去做，该线程和调用方式异步的，结果回调。非阻塞式调用

**对集合进行并行计算的两种方式：**

- 计算密集型任务：推荐使用Stream接口
- I/O密集型任务(包括网络连接)：推荐使用CompletableFuture，自己设定线程数。如果流的处理过程中发生io等待，流的延迟特性导致不易判断什么时候发生的等待

#### <a name="fenced-code-block">对多个异步任务进行流水线操作</a>

```java
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
```

> Java的线程分为两种：User Thread(用户线程)、Daemon Thread(守护线程)<br>
>
> - 只要当前JVM实例中尚存任何一个非守护线程没有结束，守护线程就全部工作；<br>
> - 只有当最后一个非守护线程结束是，守护线程随着JVM一同结束工作，Daemon作用是为其他线程提供便利服务，守护线程最典型的应用就是GC，他就是一个很称职的守护者。<br>
>   因为没有了被守护者，Daemon也就没有工作可做了，也就没有继续运行程序的必要了。