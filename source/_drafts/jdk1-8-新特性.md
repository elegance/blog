---
title: jdk1.8-新特性
tags: java
---

结合已经使用的 与文章：[Java 8新特性终极指南](http://www.360doc.com/content/14/0620/11/1370831_388286071.shtml)

#### Stream
函数式编程，`filter、mapToxx、paralle、reduce`等
```java
msgList
    .stream()
    .filter(msg -> !"成功".equals(msg.getErrmsg())) // 过滤成功的数据
    .map(msg -> msg.getKeyfield() + ":" + msg.getErrmsg()) // 每项中：拼接错误字段、错误消息
    .reduce("", (acc, val) -> acc + val + ";"); // 把每项拼接好的错误消息累加成一个字符串
```

#### Lambda 应用
[java lambda方法引用总结——烧脑吃透](https://my.oschina.net/polly/blog/917214)

1. 接口实现
```java
    new Thread(() -> System.out.print("Hello")).start();
```

2. 方法引用
```java
    String[] datas = new String[] {"peng", "Zhao", "li"};
    Arrays.sort(datas, String::compareToIgnoreCase);
    Stream.of(datas).forEach(System.out::println);
    // 构造方法引用
    String str = "test";
    Stream.of(str).map(String::new).peek(System.out::println).findFirst();
```

#### java.time: LocalDate、LocalTime、LocalDateTime、Period、Duration、Instant
```
    Date date = DateUtils.parseDate("2017-4-10 16:19:54", "yyyy-MM-dd HH:mm:ss");
    LocalDateTime bornDateTime = LocalDateTime.ofInstant(date.toInstant(), ZoneOffset.systemDefault()); // Date ==> LocalDateTime
    Duration.between(bornDateTime, LocalDateTime.now()).abs().toHours(); // 得到相差小时数
    Period.between(LocalDate.now(), LocalDate.MAX).getYears(); // 相差年数
```

#### Optional 容器防止空指针