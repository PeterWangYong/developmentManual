在Java体系中，Bean Validation 2.0(JSR380)是当前的数据校验规范，Hibernate Validator是JSR380的参考实现，也是事实标准。SpringBoot整合了Hibernator Validator作为数据校验的实现。



## 引入依赖

```java
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
```

```spring-boot-starter-web```已经包含了hibernate-validator依赖，如果不做web项目，也可以使用```spring-boot-starter-validation```引入依赖。



## 校验Controller

在Web开发中，最常用的就是对前端传入的数据进行校验。

### PathVariables和RequestParameters

PathVariables指的是请求路径中的变量，比如/book/{id}, id即为PathVariables;

RequestParameters指的是请求参数，比如/book/1?name=diana, name即为RequestParameters。

> 对于专业术语，英文表述更为准确，文章中更多使用英文术语。

对于这两类数据，可以直接对Controller的方法参数进行校验。

```java
@RestController
@Validated
public class BookController {
  @GetMapping("/book/{id}")
  public String getById(@PathVariable @Max(50) Integer id) {
    return "bookById";
  }

  @GetMapping("/bookByName")
  public String getByName(@RequestParam @Length(min = 2, max = 20) String name) {
    return "bookByName";
  }
}
```

在方法参数前面加上相应的注解(annotations)即可添加相关约束(constraint)。



另外需要在Controller上添加```@Validated```注解告诉Spring需要校验参数

> Hibernate Validator自带了很多基础注解，见后文。



### RequestBody

RequestBody指定是客户端通过POST或PUT方法在请求体中传递过来的JSON格式的数据。

对于RequestBody数据的校验，我们首先需要定义一个DTO对象作为容器来接收数据。

SpringBoot会自动将RequestBody映射到DTO对象，我们需要校验DTO对象是否符合约束条件。

> DTO: 数据传输对象

#### 定义DTO对象

```java
@Getter
@Setter
public class BookDTO {
  @Max(50)
  private Integer id;
  @Length(min = 2, max = 20)
  private String name;
}
```

校验DTO对象即校验对象的成员变量是否满足条件，所以我们需要在DTO对象的成员变量上加上相应注解。

> 注意DTO需要添加Getter和Setter方法用于序列化和反序列化，此处使用Lombok添加

#### 添加校验

```java
  @PostMapping("/book/add")
  public BookDTO addBook(@RequestBody @Validated BookDTO bookDTO) {
    return bookDTO;
  }
```

需要将DTO对象添加到Controller方法参数中，同时添加```@Validated```注解

> BookDTO前面的@Validated也可以换成@Valid，@Validated是Spring定义的对标准@Valid的扩展，此处为了方便统一使用@Validated注解

#### 关于嵌套的RequestBody

很多时候，RequestBody具有多层嵌套结构，相应的DTO对象也要有多层嵌套。

```java
@Getter
@Setter
public class BookDTO {
  @Max(50)
  private Integer id;
  @Length(min = 2, max = 20)
  private String name;
  @Valid
  private PublisherDTO publisher;
}

@Getter
@Setter
public class PublisherDTO {
  @Length(min = 2, max = 20)
  private String name;
}

```

比如PublisherDTO是嵌套在BookDTO内的一层对象，我们首先需要做两件事：

- 在PublisherDTO的成员变量上添加校验注解
- 在BookDTO的publisher成员变量上添加```@Valid```注解

## 校验Service和Entity

在Service和Entity中也可以使用校验，Service的校验和Controller类似

```java
@Service
@Validated
public class BookService {
  public String getById(@Max(20) Integer id) {
    return "book";
  }
}
```

在Entity中添加了```@Entity```注解后不需要再添加```@Validate```，因为校验过程由JPA调用Validator完成

```java
@Entity
public class Book {
  @Id
  @Max(20)
  private Integer id;
  @Length(min = 2, max = 20)
  private String name;
}
```

> 通常来说，Bean Validation只需要在Controller层完成即可，不需要每一层都进行校验。



## 内置校验注解

### 标准JSR注解

```java
@NotBlank
@NotEmpty
@NotNull
@Max(value=)
@Min(value=)
@AssertFalse
@AssertTrue
@Size(min=, max=)
@Positive
@Negative
@Past
@Email
@Future
@Pattern(regex=, flags=)
@DecimalMax(value=, inclusive=)
@DecimalMin(value=, inclusive=)
@Digits(integer=, fraction=)
@FutureOrPresent
@NegativeOrZero
@Null
@PastOrPresent
@PositiveOrZero
```

### Hibernate扩展注解

```java
@Range(min=, max=)
@Length(min=, max=)
@URL(protocol=, host=, port=, regexp=, flags=)

@CreditCardNumber(ignoreNonDigitCharacters=)
@Currency(value=)
@DurationMax(days=, hours=, minutes=, seconds=, millis=, nanos=, inclusive=)
@DurationMin(days=, hours=, minutes=, seconds=, millis=, nanos=, inclusive=)
@EAN
@ISBN
@CodePointLength(min=, max=, normalizationStrategy=)
@LuhnCheck(startIndex= , endIndex=, checkDigitIndex=, ignoreNonDigitCharacters=)
@Mod10Check(multiplier=, weight=, startIndex=, endIndex=, checkDigitIndex=, ignoreNonDigitCharacters=)
@Mod11Check(threshold=, startIndex=, endIndex=, checkDigitIndex=, ignoreNonDigitCharacters=, treatCheck10As=, treatCheck11As=)
@SafeHtml(whitelistType= , additionalTags=, additionalTagsWithAttributes=, baseURI=)
@ScriptAssert(lang=, script=, alias=, reportOn=)
@UniqueElements
```

> 官方文档：https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-defineconstraints-spec



## 如果不使用SpringBoot，如何进行校验？

Bean Validation 分为Annotation和Validator两部分，前者用于添加约束条件，后者用于校验。

SpringBoot扫描到```@Validated```之后就会帮我们调用Validator进行参数校验，如果没有SpringBoot，我们也可以自己调用Validator进行校验。

```java
public static void main(String[] args) {
    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    Validator validator = factory.getValidator();
    PublisherDTO publisher = new PublisherDTO();
    publisher.setName("a");
    Set<ConstraintViolation<PublisherDTO>> violations = validator.validate(publisher);
    // 长度需要在2和20之间
    violations.forEach(violation -> System.out.println(violation.getMessage()));
  }
```

当然，SpringBoot也可以帮我们注入validator实例

```java
@Component
public class InvokeValidator {

  private Validator validator;

  public InvokeValidator(Validator validator) {
    this.validator = validator;
  }

  public String validate() {
    PublisherDTO publisher = new PublisherDTO();
    publisher.setName("a");
    Set<ConstraintViolation<PublisherDTO>> violations = this.validator.validate(publisher);
    // 长度需要在2和20之间
    StringBuilder message = new StringBuilder();
    violations.forEach(violation -> message.append(violation.getMessage()).append(";"));
    return message.toString();
  }
```



## 如何自定义注解和校验器？

如果内置校验注解无法满足需要，我们也可以自定义校验器。自定义校验器需要分别定义Annotation和Validator两部分，然后将两者加以关联。



比如我们需要校验“两次密码输入是否相同”，就可以使用自定义校验器。

### 自定义注解

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Constraint(validatedBy = PasswordEqualValidator.class)
public @interface PasswordEqual {
	int min() default 5;
  String message() default "两次密码不一致";

  Class<?>[] groups() default { };

  Class<? extends Payload>[] payload() default { };
}
```

- 首先定义注解@PasswordEqual，注解必须包含message,groups,payload，此外我们添加了参数min要求密码不少于5个字符。
- 注解定义完成后需要使用```@Constraint(validatedBy=)```和校验器相关联



### 自定义校验器

```java
public class PasswordEqualValidator implements ConstraintValidator<PasswordEqual, UserDTO> {
  
  private int min;
  
  @Override
  public void initialize(PasswordEqual constraintAnnotation) {
    this.min = constraintAnnotation.min();
  }

  @Override
  public boolean isValid(UserDTO userDTO, ConstraintValidatorContext constraintValidatorContext) {
    String password1 = userDTO.getPassword1();
    String password2 = userDTO.getPassword2();
    return password1 != null && password1.length() >= this.min && password1.equals(password2);
  }
}
```

- 首先定义一个类PasswordEqualValidator，要求实现```ConstraintValidator<PasswordEqual, UserDTO>```接口，该接口是一个泛型接口，类型分别为“关联的注解类型”和“注解标注的对象类型”，由于我们的注解标注在```UserDTO```这个类上，所以此处填写UserDTO。
- 校验器需要实现两个方法```initialize()```和```isValid()```,前者用于关联注解，从注解中获取参数值；后者用于关联“被标注的对象”同时完成校验逻辑，最终返回boolean值。



### 测试自定义注解

```java
@Getter
@Setter
@PasswordEqual(min = 10)
public class UserDTO {
  private String name;
  private String password1;
  private String password2;
}

@RestController
@Validated
public class UserController {
  @PostMapping("/login")
  public String login(@RequestBody @Validated UserDTO userDTO) {
    return "login success";
  }
}
```

我们将```@PasswordEqual(min = 10)```标注在UserDTO上即可添加约束，和内置注解使用方式相同。



## 返回校验失败信息

对于RequestBody，校验失败会抛出```MethodArgumentNotValidException```异常

对于PathVariables和RequestParameters，校验失败会抛出```ConstraintViolationException```异常

我们可以使用```@ControllerAdvice```和```@ExceptionHandler```捕获和处理特定的Controller异常并进行结构化返回。

具体内容见我的另一篇文章：”SpringBoot 格式化响应“。

> 目前可能还没写



源代码：https://github.com/PeterWangYong/blog-code/tree/master/validation







