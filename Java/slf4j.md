## 介绍

Slf4j使用代理模式实现对多种日志框架的封装，类似JDBC，提供统一接口实现日志记录。

logback是一款高性能的日志框架，slf4j+logback是一个常用组合。

## 配置依赖

```xml
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.30</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-classic -->
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.2.3</version>
    </dependency>
```

## 基础使用

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Basic {
  public static void main(String[] args) {
    // 内部调用Basic.class.getName()作为Logger的名称
    Logger logger = LoggerFactory.getLogger(Basic.class);
    // 可以使用模板字符串
    logger.debug("this is debug log");
    logger.info("this is {} log", "info");
    logger.warn("this is {} log", "warn");
    logger.error("this is {} log", "error");
  }
}
```



