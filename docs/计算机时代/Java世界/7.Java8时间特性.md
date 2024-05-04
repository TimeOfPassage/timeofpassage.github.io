# Java-java8时间特性

## 获取当前时间(不考虑时区)
>当前电脑时间：2020-11-10 15:11

```java
LocalDateTime now = LocalDateTime.now();
System.out.println(now); // 2020-11-10T15:11:09.566
LocalDate localDate = now.toLocalDate();
System.out.println(localDate);// 2020-11-10

Month month = now.getMonth();
System.out.println(month);// 月: NOVEMBER
int dayOfMonth = now.getDayOfMonth();
System.out.println(dayOfMonth);// 日:10
int second = now.getSecond();
System.out.println(second);// 秒:56

// 设置时间为2018年，11月15号
LocalDateTime localDateTime = now.withDayOfMonth(15).withYear(2018);
System.out.println(localDateTime); // 2018-11-15T15:15:18.829

// 设置时间为2018年，3月20号
LocalDate of = LocalDate.of(2018, Month.MARCH, 20);
System.out.println(of);// 2018-03-20

// 设置时间为20点20分
LocalTime time = LocalTime.of(20, 20);
System.out.println(time);// 20:20

// 解析字符串
LocalTime parse = LocalTime.parse("18:15:01");
System.out.println(parse);// 18:15:01
```