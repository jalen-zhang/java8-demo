### java.time.*

java.util.Date：只能以毫秒的精度表示时间，且年从1900开始，月从0开始；时区支持不够

java.util.Calendar：月从0开始

DateFormat：不是线程安全的

Joda-Time：第三方库，java8参考了较多

#### <a name="fenced-code-block">java8提供的日期和时间</a>

- `LocalDate`

  - 有3个主要属性：year, month, day
  - represents a date，只提供了简单的日期，不含时间信息，不含时区信息。如2007-12-03；

  - 除了各种方法构建LocalDate之外，还提供了很多针对year-month-day的"加减运算"，如：`plus(), plusYears(), plusMonths(), plusDays(),minus****()`

  - Combines this date with a time to create a LocalDateTime: `LocalDateTime atTime()`

  - `formart()`: Formats this date using the specified formatter.

  - Compares this date to another date：`compareTo(), isAfer(), isBefore(), isEqual()`
  - `with()`：以当前对象为模板，对某些状态进行修改，并创建该对象的副本
  - `TemporalAdjuster`：类中提供了许多针对日期的复杂操作的工厂方法，比如firstDayOfYear()将返回一个新的日期，它的值为当年的第一天。也可以创建自己的TemporalAdjuster

- `LocalTime`

  - 有4个主要属性：hour, minute, second, nano.前3个位byte类型(getHour()返回的是int)，nano为int
  - 表示时间，如19:15:30 or 13:45.30.123456789
  - 这里面定义了很多常量，比如：`MILLIS_PER_DAY`
  - 主要方法：`now(), of(), of***(), parse(), plus(), minus(), format(), compare(),  `

- `LocalDateTime`

  - 面向人的日期时间，是LocalDate和LocalTime的合体，不含时区信息。如2007-12-03T10:15:30，

    the value "*2nd October 2007 at 13:45.30.123456789*" can be stored in a LocalDateTime

  - 有2个主要属性：date(LocalDate), time(LocalTime)

  - 主要方法：`now(), of(date, time), ofInstant(), parse(), get***(), with***(), plus(), minus(), formar(), compare(),  `

- `Instant`

  - 面向机器的时间戳，从1970.1.1开始所经历的秒数进行计算，精度包含nano
  - 可通过 Duration 和 Period 类来使用Instant

- `Duration`主要用于以秒和纳秒衡量时间的长短

- `Period`以年月日的方式对多个时间单位建模

为了更好地支持函数式编程，以上表示时间的对象，都是不可修改的，确保线程安全。如果需要修改对象，可以使用withAttribute()方法，该方法会创建对象的一个副本，并按要求修改指定的属性，如：`LocalDate.now().withYear(2018);` 返回的日期表示是2018年

- `DateTimeFormatter`
  - *TODO: source code!*
  - 该类的实例是线程安全的！

```java
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
```