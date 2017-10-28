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

3. Function传递
```java
    // 1. 静态方法调用
    Function<A, String> func1 = Test::f1;   // 静态方法
    String result1 = func1.apply(a); 

    // 2. 实例方法调用
    Test test = new Test();  
    Function<B, String> func2 = test::f2;  
    String result2 = func2.apply(b);  

    // 3. 定义function apply的方法
    private <T> String f(Function<T, String> func, T value) {
        return func.apply(value);
    }
    String result3 = f(func1, a);
    String result4 = f(func2, a);

    // 4. 无返回值 Method 到 Function的调用
    public static void f3(Object c) {
        System.out.println(c);
    }
    Consumer<Object> c = Test::f3;
    f3.accept("go");
```

4. `java.util.function` 包含了常用的函数式接口：
* `Predicate<T>`：接收 `T` 对象，返回 `boolean`
* `Consumer<T>`：接收 `T` 对象，不返回值
* `Function<T, R>`：接收 `T` 对象，返回 `R` 对象
* `Supplier<T>`：接收 `T` 对象（例如工厂），不接收值
* `UnaryOperation<T>`：接收 `T` 对象，返回 `T`对象
* `BinaryOperation<T>`：接收两个 `T` 对象，返回 `T` 对象

#### java.time: LocalDate、LocalTime、LocalDateTime、Period、Duration、Instant
```
    Date date = DateUtils.parseDate("2017-4-10 16:19:54", "yyyy-MM-dd HH:mm:ss");
    LocalDateTime bornDateTime = LocalDateTime.ofInstant(date.toInstant(), ZoneOffset.systemDefault()); // Date ==> LocalDateTime
    Duration.between(bornDateTime, LocalDateTime.now()).abs().toHours(); // 得到相差小时数
    Period.between(LocalDate.now(), LocalDate.MAX).getYears(); // 相差年数
```

#### Optional 容器防止空指针