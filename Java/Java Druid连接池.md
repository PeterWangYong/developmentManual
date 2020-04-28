## 介绍

Druid是阿里出品的Java 数据库连接池，在性能，稳定性，扩展性和监控方面做得比较好，是国内流行的数据库连接池。



## SpringBoot整合

### pom.xml

```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>1.1.22</version>
    </dependency>
```

### application.yml

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/user_demo?serverTimezone=Asia/Shanghai
    username: root
    password: password

    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      # 初始化连接数
      initial-size: 1
      # 最小连接数
      min-idle: 1
      # 最大连接数
      max-active: 20
      # 连接最大等待时间
      max-wait: 60000

      # 定时检查并销毁空闲连接：如果连接空闲时间超过min-evictable-idle-time-millis则进行销毁
      # 这里每60s检查一次，如果连接空闲时间超过300s则进行销毁
      time-between-eviction-runs-millis: 60000
      # 保持空闲而不被销毁的最小时间
      min-evictable-idle-time-millis: 300000

      # 测试连接是否有效，要求是select语句
      validation-query: SELECT 1 FROM DUAL
      # 申请连接时如果连接空闲时间超过time-between-eviction-runs-millis则进行检测
      # 这里如果连接空闲时间超过60s则使用validation-query语句进行检查
      test-while-idle: true
      # 申请连接时测试连接是否有效（开启会降低性能）
      test-on-borrow: false
      # 归还连接时测试连接是否有效（开启会降低性能）
      test-on-return: false

      # 是否缓存preparedStatement即PSCache,Mysql不支持
      pool-prepared-statements: false
      # 如果>0则pool-prepared-statements自动为true
      max-pool-prepared-statement-per-connection-size: -1

      # 配置扩展插件（拦截器）：stat 用于监控统计 config 用于数据库密码加密 wall 用于防止SQL注入 slf4j 用于记录日志
      filters: stat,config,wall,slf4j
      # druid.stat.mergeSql=true 合并统计相同的SQL(避免因为参数不同而统计多条SQL)
      # druid.stat.slowSqlMillis=5000 统计慢查询SQL
      connect-properties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    open-in-view: false

```

### 实体类

```java
@Entity
@Table(name = "user")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserEntity {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private int id;
  private String name;
  @Column(updatable = false)
  @CreationTimestamp
  private Date createTime;
  @UpdateTimestamp
  private Date updateTime;
}
```

### 开启控制台

Druid默认不开启监控页面，在SpringBoot中可以使用ServletRegistrationBean对象注入对应的Servlet。

```java
@SpringBootApplication
public class DemoApplication {

  @Bean
  public ServletRegistrationBean<StatViewServlet> druidStatViewServlet() {
    ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<>(new StatViewServlet(),  "/druid/*");
    registrationBean.addInitParameter("allow", "127.0.0.1");// IP白名单 (没有配置或者为空，则允许所有访问)
    registrationBean.addInitParameter("deny", "");// IP黑名单 (存在共同时，deny优先于allow)
    registrationBean.addInitParameter("loginUsername", "admin");
    registrationBean.addInitParameter("loginPassword", "password");
    registrationBean.addInitParameter("resetEnable", "false"); // 是否允许计数器清零（对应控制台的重置按钮）
    return registrationBean;
  }

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

}
```



