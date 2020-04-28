## 介绍

H2是Java编写的一款内嵌式数据库，支持内存和文件两种方式存储数据。



## SpringBoot整合

### pom.xml

```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
    </dependency>
```

### application.yml

```yml
spring:
  datasource:
  	# url: jdbc:h2:mem:testdb
    url: jdbc:h2:file:./src/main/resources/data.sql
    driver-class-name: org.h2.Driver
    username: sa
    password: password
  h2:
    # web控制台
    console:
      enabled: true
      path: /h2-console
      settings:
        # 是否打印trace级别日志
        trace: false
        # 是否开放外部访问，如果为false则只允许localhost:8080/h2-console进行访问
        web-allow-others: false
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
```

### 访问控制台

> http://localhost:8080/h2-console

![](https://img2020.cnblogs.com/blog/1192583/202004/1192583-20200428112741443-1477475911.png)

## 命令行执行

```shell
java -jar h2-1.4.200.jar
```

执行后将自动打开浏览器到控制台页面