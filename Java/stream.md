## 介绍

Stream是JDK8引入的一个类型，类似Iterator迭代器，帮助开发者以流的方式处理数据。



Stream的操作分为“转换操作”和“聚合操作”，转换操作返回一个新的Stream，实现链式操作，聚合操作返回最终值并销毁Stream。



Stream通过”转换操作“构建了一个管道，每一个元素处理完毕后立刻向后传递；单独写”转换操作“是不会执行的，只有添加了“聚合操作”，整个管道才会执行。



Stream支持并行处理，类似大数据中map-reduce的模式。



Stream通常与Lamba表达式结合使用，以链式操作进行表达。



## 创建Stream

- Collection.stream()
- Collection.parallelStream()
- Arrays.stream(T[] array)
- Stream.of(T... values)



### 普通流

```java
public class Basic10 {
  private static List<Integer> integerList = Arrays.asList(1,2,3,4,5);
  private static String[] stringArray = {"a", "b", "c"};
  private static int[] intArray = {1,2,3,4,5};
  private static long[] longArray = {100L, 200L, 300L};
  private static double[] doubleArray = {1.1, 1.2, 1.3};

  public static void main(String[] args) {
    // 面向对象类型
    Stream<Integer> integerStream = integerList.stream();
    Stream<Integer> integerStream1 = integerList.parallelStream();
    Stream<String> stringStream = Arrays.stream(stringArray);
    Stream<String> stringStream1 = Stream.of("a", "b", "c");

    // 面向基本类型
    IntStream intStream = Arrays.stream(intArray);
    LongStream longStream = Arrays.stream(longArray);
    DoubleStream doubleStream = Arrays.stream(doubleArray);
  }
}
```



### 并行流

```java
/**
 * parallel, parallelStream
 * 并行处理，无法保证顺序
 */
public class Basic9 {
  private static List<Integer> integerList = Arrays.asList(1,2,3,4,5,6,7,8,9,10);

  public static void main(String[] args) {
    System.out.println(integerList.stream().parallel().map(x -> x * x).collect(Collectors.toList()));
    System.out.println(integerList.parallelStream().map(x -> x * x).collect(Collectors.toList()));
  }
}
```



## 转换操作

### map和flatMap

- map:进行数据的一对一映射和转换
- flatMap:将元素转换分别为Stream然后合并为一个Stream

```java
/**
 * map和flatMap
 */
public class Basic {
  private static List<String> list = Arrays.asList("hello", "word");
  private static List<Integer> nums = Arrays.asList(1,2,3,4,5);
  private static List<List<Integer>> listNums = Arrays.asList(
    Arrays.asList(1,2,3),
    Arrays.asList(4,5,6)
  );
  public static void main(String[] args) {
    // 转换大写 [HELLO, WORD]
    		System.out.println(Basic.list.stream().map(String::toUpperCase).collect(Collectors.toList()));
    // 求平方  [1, 4, 9, 16, 25]
    System.out.println(Basic.nums.stream().map(x -> x * x).collect(Collectors.toList()));
    // 展开合并 [1, 2, 3, 4, 5, 6]
    System.out.println(Basic.listNums.stream().flatMap(Collection::stream).collect(Collectors.toList()));
  }
}
```



### filter

根据条件过滤元素

```java
/**
 * filter
 */
public class Basic2 {
  private static Integer[] nums = {1, 2, 3, 4, 5};
  private static String[] strings = {"hello", "word"};

  public static void main(String[] args) {
    // 过滤偶数 [2, 4]
    System.out.println(Arrays.stream(nums).filter(x -> x % 2 == 0).collect(Collectors.toList()));
    // 过滤字符串 [word]
    System.out.println(Stream.of(strings).filter(x -> x.length() == 4).collect(Collectors.toList()));
  }
}
```



### peek

对每一个元素执行相同操作，同forEach

```java
/**
 * peek
 */
public class Basic3 {
  private static String[] strings = {"hello", "world"};

  public static void main(String[] args) {
    // peek是一个转换操作，功能和forEach一致，但会返回Stream
    // hello,HELLO,world,WORD 
    // Stream转换操作:每一个元素完成后立刻往后送，不是等待全部元素转换完成一起往后送
 						Arrays.stream(strings).peek(System.out::println).map(String::toUpperCase).peek(System.out::println).count();
  }
}
```



### skip和limit

- skip：忽略前n个元素
- limit：截取前n个元素

```java
/**
 * skip, limit
 */
public class Basic5 {
  private static Stream<Integer> integerStream = Stream.of(1,2,3,4,5);

  public static void main(String[] args) {
    // 忽略前两个，然后从后面的元素中截取前两个进行计算 [3,4]
    System.out.println(integerStream.skip(2).limit(2).collect(Collectors.toList()));
  }
}
```



### sorted

对元素进行排序

```java
/**
 * sorted
 */
public class Basic6 {
  private static Stream<Integer> integerStream = Stream.of(1,3,2,4,5);
  private static Stream<String> stringStream = Stream.of("b", "a", "d", "c");
  private static Stream<Person> personStream = Stream.of(new Person("John"), new Person("Diana"));

  public static void main(String[] args) {
    // 对Comparable对象进行排序 [1, 2, 3, 4, 5] [a, b, c, d]
    System.out.println(integerStream.sorted().collect(Collectors.toList()));
    System.out.println(stringStream.sorted().collect(Collectors.toList()));
    // 对非Comparable对象进行排序 [Basic6.Person(name=Diana), Basic6.Person(name=John)]
    System.out.println(personStream.sorted(Comparator.comparing(p -> p.name)).collect(Collectors.toList()));
  }

  @Data
  private static class Person {
    private final String name;
  }
}
```



### distinct

```java
/**
 * distinct
 * 通过equals方法
 */
public class Basic7 {
  private static Stream<String> stringStream = Stream.of("a", "a", "b", "B", "c");

  public static void main(String[] args) {
    // 去重 [a, b, B, c]
    System.out.println(stringStream.distinct().collect(Collectors.toList()));
  }
}
```





## 聚合操作

### forEach

对每一个元素执行相同操作

```java
/**
 * forEach
 */
public class Basic3 {
  private static String[] strings = {"hello", "world"};

  public static void main(String[] args) {
    // forEach是一个聚合操作，不再返回Stream
    // hello world
    Arrays.stream(strings).forEach(System.out::println);
  }
}
```



### reduce

对元素进行叠加聚合，通常提供一个初始值，后续元素向初始值叠加

```java
/**
 * reduce
 */
public class Basic4 {
  private static List<Integer> nums = Arrays.asList(1,2,3,4,5);
  private static List<String> strings = Arrays.asList("a", "b", "c");

  public static void main(String[] args) {
    // 求和 15
    System.out.println(nums.stream().reduce(0, Integer::sum));
    // 拼接字符串 abc
    System.out.println(strings.stream().reduce("", String::concat));
    // 最小值 1
    System.out.println(nums.stream().reduce(Integer.MAX_VALUE, Integer::min));
    // 最大值 5
    System.out.println(nums.stream().reduce(Integer.MIN_VALUE, Integer::max));
  }
}
```



### allMatch,anyMatch,noneMatch

根据条件进行匹配判断，返回布尔值

```java
/**
 * allMatch, anyMatch, noneMatch
 */
public class Basic8 {
  private static List<Integer> integerList = Arrays.asList(1,2,3,4,5);

  public static void main(String[] args) {
    System.out.println("全部是偶数吗? " + integerList.stream().allMatch(x -> x % 2 == 0));
    System.out.println("存在偶数吗？" + integerList.stream().anyMatch(x -> x % 2 == 0));
    System.out.println("全都不是偶数吗？" + integerList.stream().noneMatch(x -> x % 2 == 0));
  }
}
```



### collect

将Stream转换为Collection

```java
/**
 * collect
 */
public class Basic1 {
  private static List<Integer> integerList = Arrays.asList(1,2,3,4,5);

  public static void main(String[] args) {
    List<Integer> resList = integerList.stream().map(x -> x * x).collect(Collectors.toList());
    Map<String, Integer> resMap = integerList.stream().map(x -> x * x).collect(Collectors.toMap(Object::toString, x -> x));
    Set<Integer> resSet = integerList.stream().map(x -> x * x).collect(Collectors.toSet());
    // [1, 4, 9, 16, 25]
    System.out.println(resList);
    // {1=1, 25=25, 4=4, 16=16, 9=9}
    System.out.println(resMap);
    // [16, 1, 4, 9, 25]
    System.out.println(resSet);
  }
}
```

### toArray

将Stream转换为Array

```java
public class Basic11 {
  private static List<Integer> integerList = Arrays.asList(1,2,3,4,5);
  private static int[] intArray = {1,2,3,4,5};

  public static void main(String[] args) {
    // 对象类型返回Object[]
    Object[] resArray = integerList.stream().map(x -> x * x).toArray();
    // 基本类型返回相同的基本类型数组
    int[] resArray1 = Arrays.stream(intArray).map(x -> x * x).toArray();
  }
}
```



