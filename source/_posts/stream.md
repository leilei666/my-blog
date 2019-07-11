---
layout: 简单又迅速的处理集合-java8
title: stream
date: 2019-07-11 14:31:31
tags:
---
## 一.什么是Stream?

Stream 中文称为 “流”，通过将集合转换为这么一种叫做 “流” 的元素序列，通过声明性方式，能够对集合中的每个元素进行一系列并行或串行的流水线操作。
   
## 二.流操作
   
整个流操作就是一条流水线，将元素放在流水线上一个个地进行处理。其中数据源便是原始集合，然后将如 List 的集合转换为 Stream 类型的流，并对流进行一系列的中间操作，比如过滤保留部分元素、对元素进行排序、类型转换等；最后再进行一个终端操作，可以把 Stream 转换回集合类型，也可以直接对其中的各个元素进行处理，比如打印、比如计算总数、计算最大值等等.
   
## 三.一般实现方法

   首先我们先创建一个 Person 泛型的 List
  
    List<Person> list = new ArrayList<>();
    list.add(new Person("jack", 20));
    list.add(new Person("mike", 25));
    list.add(new Person("tom", 30));
    
   Person 类包含年龄和姓名两个成员变量
   
    private String name;
    private int age;
    
   ### 3.1.stream() / parallelStream()
   
   最常用到的方法，将集合转换为流
   
    List list = new ArrayList();
    // return Stream<E>
    list.stream();
    
   而 parallelStream() 是并行流方法，能够让数据集执行并行操作
   
   ### 3.2.filter(T -> boolean)
   
   保留 boolean 为 true 的元素
   
    保留年龄为 20 的 person 元素
    list = list.stream()
               .filter(person -> person.getAge() == 20)
               .collect(toList());
   
    打印输出 [Person{name='jack', age=20}]
    
   collect(toList()) 可以把流转换为 List 类型
   
   ### 3.3.distinct()
   
   去除重复元素，这个方法是通过类的 equals 方法来判断两个元素是否相等的
   
    如例子中的 Person 类，需要先定义好 equals 方法，不然类似[Person{name='jack', age=20}, Person{name='jack', age=20}] 这样的情况是不会处理的
   
   ### 3.4.sorted() / sorted((T, T) -> int)
   
   如果流中的元素的类实现了 Comparable 接口，即有自己的排序规则，那么可以直接调用 sorted() 方法对元素进行排序，如 Stream
   反之, 需要调用 sorted((T, T) -> int) 实现 Comparator 接口
   
    根据年龄大小来比较：
    list = list.stream()
              .sorted((p1, p2) -> p1.getAge() - p2.getAge())
              .collect(toList());
    当然这个可以简化为
    list = list.stream()
              .sorted(Comparator.comparingInt(Person::getAge))
              .collect(toList());
   ### 3.5.limit(long n)
   
   返回前 n 个元素
   
    list = list.stream()
               .limit(2)
               .collect(toList());
   
    打印输出 [Person{name='jack', age=20}, Person{name='mike', age=25}]
   ### 3.6. skip(long n)
   
   去除前 n 个元素
   
    list = list.stream()
               .skip(2)
               .collect(toList());
   
    打印输出 [Person{name='tom', age=30}]
   tips:
   
   用在 limit(n) 前面时，先去除前 m 个元素再返回剩余元素的前 n 个元素
   limit(n) 用在 skip(m) 前面时，先返回前 n 个元素再在剩余的 n 个元素中去除 m 个元素.
   
     list = list.stream()
               .limit(2)
               .skip(1)
               .collect(toList());
   
    打印输出 [Person{name='mike', age=25}]
   ### 3.7. map(T -> R)
   
   将流中的每一个元素 T 映射为 R（类似类型转换）
   
    List<String> newlist = list.stream().map(Person::getName).collect(toList());
    newlist 里面的元素为 list 中每一个 Person 对象的 name 变量
   
   ### 3.8. flatMap(T -> Stream)
   
   将流中的每一个元素 T 映射为一个流，再把每一个流连接成为一个流
   
    List<String> list = new ArrayList<>();
    list.add("aaa bbb ccc");
    list.add("ddd eee fff");
    list.add("ggg hhh iii");
   
    list = list.stream().map(s -> s.split(" ")).flatMap(Arrays::stream).collect(toList());
    
   上面例子中，我们的目的是把 List 中每个字符串元素以" "分割开，变成一个新的 List
   
   ### 3.9. anyMatch(T -> boolean)
   
   流中是否有一个元素匹配给定的 T -> boolean 条件
   
    是否存在一个 person 对象的 age 等于 20：
    boolean b = list.stream().anyMatch(person -> person.getAge() == 20);
   ### 3.10. allMatch(T -> boolean)
   
   流中是否所有元素都匹配给定的 T -> boolean 条件
   
   ### 3.11. noneMatch(T -> boolean)
   
   流中是否没有元素匹配给定的 T -> boolean 条件
   
   ### 3.12. findAny() 和 findFirst()
   
   findAny()：找到其中一个元素 （使用 stream() 时找到的是第一个元素；使用 parallelStream() 并行时找到的是其中一个元素）.
   
   findFirst()：找到第一个元素
   
   值得注意的是，这两个方法返回的是一个 Optional，它是一个容器类，能代表一个值存在或不存在.
   
   ### 3.13. reduce((T, T) -> T) 和 reduce(T, (T, T) -> T)
   
   用于组合流中的元素，如求和，求积，求最大值等
   
    计算年龄总和：
    int sum = list.stream().map(Person::getAge).reduce(0, (a, b) -> a + b);
    与之相同:
    int sum = list.stream().map(Person::getAge).reduce(0, Integer::sum);
    其中，reduce 第一个参数 0 代表起始值为 0，lambda (a, b) -> a + b 即将两值相加产生一个新值
   
    同样地：
   
    计算年龄总乘积：
    int sum = list.stream().map(Person::getAge).reduce(1, (a, b) -> a * b);
    当然也可以
   
    Optional<Integer> sum = list.stream().map(Person::getAge).reduce(Integer::sum);
    即不接受任何起始值，但因为没有初始值，需要考虑结果可能不存在的情况，因此返回的是 Optional 类型
   
   ### 3.13. count()
   
   返回流中元素个数，结果为 long 类型
   
   ### 3.14.collect()
   
   收集方法，我们很常用的是 collect(toList())，当然还有 collect(toSet()) 等.
   
   ### 3.15. forEach()
   
   返回结果为 void，很明显我们可以通过它来干什么了，比方说：
   
    打印各个元素：
    list.stream().forEach(System.out::println);