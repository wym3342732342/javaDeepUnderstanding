#### JAVA 8 Stream简单介绍

&emsp;&emsp;Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。

&emsp;&emsp;Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

&emsp;&emsp;Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。

&emsp;&emsp;这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

&emsp;&emsp;元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

##### 1.什么是Stram？

- Stream(流)是一个来自数据源的元素队列并支持聚合操作。
  - 元素是特定类型的对象，形成一个队列。Java中的Stream并不会存储元素，而是按需计算。
  - **数据源（流）** 的来源。可以是集合，数组，I/O channel，产生器generator等。
  - **聚合操作** 类似SQL语句一样的操作，比如filter，map,reduce,find,match,sorted等。
- 和以前的Collection操作不同，Stream操作还有两个基础的特征：
  - **Pipelining**：中间操作都会返回流对象本身。这样多个操作可以串联成一个管道，如同流式风格（fluent style）。这样做可以对操作进行优化，比如延迟执行（laziness）和短路（short-circuiting）。
  - **内部迭代**：以前的集合遍历都是通过Iterator或者foreach的方式，显式的在集合外部进行迭代，这叫外部迭代。Stream提供了内部迭代的方式，通过访问者模式（Visitor）实现。

##### 2.生成流

&emsp;&emsp;在Java8中，集合接口有两个方法生成流：

- stream():为集合创建串行流。
- ParallelStream():为集合创建并行流。



#### 1. Stream流的分类

> 此处提供lambda的部分函数式接口，方便Stream流的学习；[java.util.function](#)

```java
//功能型函数式接口
public interface Function<T, R> {
    R apply(T t);
}

//供给型函数式接口
public interface Supplier<T> {
    T get();
}

//消费型函数式接口
public interface Consumer<T> {
    void accept(T t);
}

//断言型函数式接口
public interface Predicate<T> {
    boolean test(T t);
}
```



- **中间操作（intermediate）：**（此类型的方法返回的都是Stream对象） 
  - map (处理后返回)：`<R> Stream<R> map(Function<? super T, ? extends R> mapper)`
  - **flatMap（流元素如果是数组或者集合，将转换为元素处理）**：
  -  filter（过滤）：`Stream<T> filter(Predicate<? super T> predicate)`
  -  distinct（去重）：`Stream<T> distinct()`
  -  sorted（排序，可自定义排序规则）：
    - `Stream<T> sorted(Comparator<? super T> comparator)`
    - `Stream<T> sorted()`
  -  peek（详情见使用）：`Stream<T> peek(Consumer<? super T> action)`
  -  limit（返回指定的前几个元素）：`Stream<T> limit(long maxSize)`
  -  skip（删除指定的前几个元素）：`Stream<T> skip(long n)`
  -  parallel：
  -  sequential：
  -  unordered ：
- **终端操作（terminal） ：**（都是对集合流的一些便捷处理方法）
  - forEach（遍历每个元素）：`void forEach(Consumer<? super T> action)`
  - forEachOrdered（并行中输出顺序保持一致）：`void forEachOrdered(Consumer<? super T> action)`
  - toArray（转化成object数组或指定数组）：
    - `Object[] toArray()`
    - `<A> A[] toArray(IntFunction<A[]> generator)`
  - reduce：（元素聚合【计算或处理】）
    - `T reduce(T identity, BinaryOperator<T> accumulator)`
    - `Optional<T> reduce(BinaryOperator<T> accumulator)`
    - `<U> U reduce(U identity,BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)`
  - collect：(将流转换成集合或聚合元素 )
    - `<R> R collect(Supplier<R> supplier,BiConsumer<R, ? super T> accumulator,BiConsumer<R, R> combiner)`
    - `<R, A> R collect(Collector<? super T, A, R> collector)`
  - min（最大值）：`Optional<T> min(Comparator<? super T> comparator)`
  - max（最小值）：`Optional<T> max(Comparator<? super T> comparator)`
  - count（元素个数）：`long count()`
  - anyMatch（任意符合条件为真）：`boolean anyMatch(Predicate<? super T> predicate)`
  - allMatch（全部符合条件为真）：`boolean allMatch(Predicate<? super T> predicate)`
  - noneMatch（全不符合为真）：`boolean noneMatch(Predicate<? super T> predicate)`
  - findFirst（有序集合，反第一个元素的Optional对象 ，无序反任意）：`Optional<T> findFirst()`
  - findAny（返回任意元素的Optional对象）：`Optional<T> findAny()`
  - iterator ：



####2. Stream流的使用

##### 2.1 filter、foreach的使用

```java
List<String> list = Arrays.asList("aaa","ddd","bbb","ccc","a2a","d2d","b2b","c2c","a3a","d3d","b3b","c3c");

list.stream().filter((s)->s.contains("a")).forEach(s -> System.out.println(s));
//此时指挥输出带有a的字符串
```

##### 2.2 map的使用（处理后返回）

```java
List<String> list = Arrays.asList("aaa","ddd","bbb","ccc","a2a","d2d","b2b","c2c","a3a","d3d","b3b","c3c");
list.stream()
        .filter((s)->s.contains("a"))
        .map((s)-> s + "---map")
        .forEach(s -> System.out.println(s));
```

##### 2.3  fiatmap的集合类型转换

>Stream<String[]> 转换为 Stream\<String> 
Stream\<Set\> 转换为 Stream\<String> 
Stream\<List> 转换为 Stream\<String> 
Stream\<List> 转换为 Stream\<Object>

```java
List<String[]> setList =  new ArrayList<>();
setList.add(new String[]{"aa","bb"});
setList.add(new String[]{"cc","dd"});
setList.add(new String[]{"ee","ff"});
//使用map方法
setList.stream()
        .map(s->Arrays.stream(s))
        .forEach(s-> System.out.println("map==" + s));
//使用flatMap方法
setList.stream()
        .flatMap(s->Arrays.stream(s))
        .forEach(s-> System.out.println("flatMap==" + s));
```

##### 2.4 **peek：**生成一个包含元Stream的所有元素的新Stream。

> 【说白了就是可以给新生成的Stream流中的元素创建一个方法，如果元素被使用了就会调用这个方法】
>
> 注意：**一个流只能用一次**
>
> 生成的同时提供了一个消费函数。
>
> 当Stream每个元素被消费的时候会先执行新Stream给定的方法A。
>
> peek是中间操作，如果peek没有最终操作，那么它将不会执行。

```java
integerList.stream()
        .peek(s-> System.out.println("peek = "+s))
        .forEach(s-> System.out.println("forEach = "+s));
```

##### 2.5 toArray：

```java
Object[] array = integerList.stream().toArray();
String[] strArr = integerList.stream().toArray(String[]::new);
```

##### 2.6 reduce将集合中每个元素聚合成一条数据

- reduce(BinaryOperator accumulator)：此处需要一个参数，返回Optional对象：

```java
Optional<Integer> reduce = integerList.stream().reduce((a, b) -> a + b);
```

- reduce(T identity, BinaryOperator accumulator)： 

> 此处需要两个参数，第一个参数为起始值，第二个参数为引用的方法 。
>
> 从起始值开始，每个元素执行一次引用的方法
>
> - 方法引用的两个参数：第一个参数为上个元素执行方法的结果，第二个参数为当前元素）。 

```java
int integer = integerList.stream().reduce(5,(a, b) -> a + b);
System.out.println(integer);
```

- reduce(U **identity**, BiFunction<U, ? super T, U> **accumulator**, BinaryOperator<U> **combiner**)： 

> 此处需要三个参数。此方法用在并发流（parallelStream）中，启动多个子线程使用accumulator进行并行计算，最终使用combiner对子线程结果进行合并，返回identity类型的数据，看到有篇文章对这个解释比较清楚：java8中3个参数的reduce方法怎么理解？



##### 2.7 collect:

>有两种情况。接受一个参数和接受三个参数（三个参数在并发流parallelStream中使用） 
>
>详见：[java8之collector](https://www.jianshu.com/p/c0d5c3094324) 
>
>Collectors.toList():他返回了一个函数式接口，其应该定义了如何将流转换成list

```java
List<Integer> collects = integerList.stream()
        .filter(a -> a > 1)
        .collect(Collectors.toList());
System.out.println(collects);
```



##### 2.8 排序、最大、最小 Comparator的使用

```java
//============================排序===================================//
integerList.stream()
        .sorted((s1,s2)->s2.compareTo(s1))
        .forEach(s-> System.out.println(s));
//============================最大===================================//
Integer max = integerList.stream()
        .filter(a -> a > 1)
        .max((Integer a, Integer b) -> a.compareTo(b))
        .get();
System.out.println(max);
//============================最小===================================//
Integer min = integerList.stream()
        .filter(a -> a > 1)
        .min((Integer a, Integer b) -> a.compareTo(b))
        .get();
System.out.println(min);
```



##### 2.9 Match：根据条件判断是否符合

```java
boolean b = integerList.stream()
        .anyMatch(s -> s > 0);
boolean b1 = integerList.stream()
        .allMatch(s -> s > 0);
boolean b2 = integerList.stream()
        .noneMatch(s -> s > 0);
System.out.println("anyMatch = " + b);
System.out.println("allMatch = " + b1);
System.out.println("noneMatch = " + b2);
```

