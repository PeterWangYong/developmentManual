## 介绍

Jasypt是Java流行的加密库，使用Jasypt-spring-boot-starter可以和springboot整合



## 独立使用

### 配置依赖

```xml
    <dependency>
      <groupId>org.jasypt</groupId>
      <artifactId>jasypt</artifactId>
      <version>1.9.3</version>
    </dependency>
```

### 基本使用

```java
public class Basic {

  public static void main(String[] args) {
    StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
    // 配置加密解密的密码/salt值
    encryptor.setPassword("encrypt_password");
    // 对"raw_password"进行加密：S5kR+Y7CI8k7MaecZpde25yK8NKUnd6p
    String password = "raw_password";
    String encPwd = encryptor.encrypt(password);
    System.out.println(encPwd);
    // 再进行解密：raw_password
    String rawPwd = encryptor.decrypt(encPwd);
    System.out.println(rawPwd);
  }
}
```



## 整合SpringBoot

### 配置依赖

```xml
		<dependency>
      <groupId>com.github.ulisesbocchio</groupId>
      <artifactId>jasypt-spring-boot-starter</artifactId>
      <version>3.0.2</version>
    </dependency>
```

## 基本使用

- application.properties

```properties
jasypt.encryptor.password=encrypt_password
# 默认前缀后缀为ENC()，其中的内容会进行解密(原始密码为raw_password)
password=ENC(UhTCzia6QMJ/anHlLrIz4fdiMRhiYsvNIWfCpEHkcHNlZsJBsZHE7eN7zNKaDONC)
```

- Controller.java

```java
@RestController
public class Controller {
  
  // 自动解密为raw_password
  @Value("${password}")
  private String password;

  @GetMapping("/password")
  public String password() {
    return password;
  }
}
```



