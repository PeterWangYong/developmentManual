## 介绍

Lombok帮助开发者自动生成getter,setter,toString等样板代码



## 配置依赖

```xml
    <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.12</version>
      <!-- 只用于编译，不进行打包 -->
      <scope>provided</scope>
    </dependency>
```

## IDEA插件

安装插件让IDEA可以识别Lombok注解，防止检查报错。

![](https://img2020.cnblogs.com/blog/1192583/202004/1192583-20200424143800245-1594379082.png)

## 常用注解

IDEA插件下面列出了几乎全部的注解

![](https://img2020.cnblogs.com/blog/1192583/202004/1192583-20200424143825930-434509947.png)

### @Getter @Setter

创建getter和setter方法。

```java
@Getter @Setter
public class Basic3 {
  private int id;
  private String name;

  public static void main(String[] args) {
    Basic3 basic3 = new Basic3();
    basic3.setId(1);
    basic3.setName("Diana");
    System.out.println(basic3.getId());
    System.out.println(basic3.getName());
  }
}
```

### @ToString

创建toString方法，同时可以指定包括和不包括哪些字段。

```java
@ToString
public class Basic5 {
  private int id;
  private String name;
  @ToString.Exclude private int age;

  public static void main(String[] args) {
    Basic5 basic5 = new Basic5();
    System.out.println(basic5);
  }
}
```



### @RequiredArgsConstructor

创建带参构造，参数包括所有未初始化的final成员和@NonNull修饰的成员。

```java
@RequiredArgsConstructor
public class Basic6 {
  private final int id;
  @NonNull private String name;
  private int age;

  public static void main(String[] args) {
    Basic6 basic6 = new Basic6(1, "Diana");
  }

}
```



### @Data

复合注解：包括@Getter、@Setter、@ToString、@RequiredArgsConstructor和@EqualsAndHashCode

```java
@Data
public class Basic2 {
  private final int id;
  private final String name;
  @NonNull private int age;
  private String role;

  public static void main(String[] args) {
    Basic2 basic2 = new Basic2(1, "Diana", 18);
    basic2.setRole("admin");
    System.out.println(basic2.getId());
    System.out.println(basic2.getName());
    System.out.println(basic2);
  }
}
```

> 如果已经存在构造器则@Data中的@RequiredArgsConstructor不生效（只能单独打上注解）

```java
@Data
@NoArgsConstructor(force = true)
@RequiredArgsConstructor
public class Basic {
  private final int id;
  private final String name;
}
```



### @Slf4j

自动创建一个Slf4J Logger对象，名称为当前类名。

```java
// Create
private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
```

```java
@Slf4j
public class Basic4 {
  public static void main(String[] args) {
    log.info("hello {}", "info");
  }
}
```



