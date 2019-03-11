### java.time.*

java.util.Date: 年从1900开始，月从0开始；时区支持不够

java.util.Calendar：月从0开始

DateFormat：不是线程安全的

Joda-Time:第三方库，java8参考了较多

#### <a name="fenced-code-block">java8提供的日期和时间</a>

- `LocalDate`该类的实例是一个不可变对象。只提供了简单的日期，不含时间信息，不含时区信息。
- `LocalTime`表示时间，如19:15:30
- `LocalDateTime`是LocalDate和LocalTime的合体，不含时区信息
- `Instant`面向机器，从1970.1.1开始所经历的秒数进行计算
- `Duration`主要用于以秒和纳秒衡量时间的长短
- `Period`以年月日的方式对多个时间单位建模

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