本文介绍使用```@ControllerAdvice```对Controller抛出的异常进行统一拦截和处理。

## 定义返回格式

```java
@Getter
@Setter
@AllArgsConstructor
public class UnifyResponse {
  private int code;
  private String message;
  private String request;
}
```

首先定义一个统一的返回格式，所有的异常最终都按照统一格式返回给前端。



## 定义状态码

不同的异常对应不同的返回状态码

```properties
# exception-code.properties
demo.codes[0] = ok
demo.codes[9999] = 服务器未知异常
demo.codes[10000] = 通用错误
demo.codes[10001] = 通用参数错误
demo.codes[10002] = 资源未找到
```

首先将状态码集中在配置文件中进行管理

> properties的编码格式需要配置，否则可能出现中文乱码
>
> IDEA中Preferences -> File Encodings -> Default encoding for properties files 设置为UTF-8，同时勾上Transparent native-to-ascii conversion

```java
@Getter
@Setter
@ConfigurationProperties("demo")
@PropertySource("classpath:config/exception-code.properties")
@Component
public class ExceptionCodeConfiguration {
  private Map<Integer, String> codes = new HashMap<>();

  public String getMessage(int code) {
    return codes.get(code);
  }
}
```

其次使用类```ExceptionCodeConfiguration```实现对配置文件的访问,```@PropertySource```可以用来指定配置文件路径，```@ConfigurationProperties```用于指定配置前缀。

在配置类中，我们首先定义一个Map变量```codes```，这里的```codes```对应于配置文件中的codes，SpringBoot会在启动时自动将配置文件读入配置类中，最后需要使用```@Component```将配置类加入容器。

```java
public final class ExceptionCode {
  public static final int UNKNOWN = 9999;
  public static final int COMMON = 10000;
  public static final int PARAMS_ERROR = 10001;
  public static final int RESOURCE_NOT_FOUND = 10002;
}
```

最后定义一个”常量类“便于状态码的调用，避免直接填入数字。



##通用异常

```@ControllerAdvice```根据异常类型的匹配程度选择相应的异常处理类，如果其他的handler都没有匹配到则使用通用的异常处理类。

```java
@RestControllerAdvice
public class GlobalExceptionAdvice {
  @Autowired
  private ExceptionCodeConfiguration codeConfiguration;

  @ExceptionHandler(Exception.class)
  @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
  public UnifyResponse handleException(HttpServletRequest req, Exception e) {
    return new UnifyResponse(ExceptionCode.UNKNOWN, codeConfiguration.getMessage(ExceptionCode.UNKNOWN), this.getRequest(req));
  }
  
  // 获取请求方法和路径
  private String getRequest(HttpServletRequest req) {
    String url = req.getRequestURI();
    String method = req.getMethod();
    return method + " " + url;
  }
}
```

首先要在全局异常处理类上标注```@ControllerAdvice```注解，这里使用```@RestControllerAdvice```避免在方法上添加```@ResponseBody```。



然后定义一个方法```handleException```，同时在方法上添加```@ExceptionHandler(value=Class<? extends Throwable>)```注解，注解接收一个异常类型，表示该方法要处理什么异常，异常类型越具体，能够处理的范围越小。方法```handleException```接收```HttpServletRequest```和```Exception```两个参数类型，用于接收数据进行处理。



最终，对数据进行处理并打包成```UnifyResponse```进行返回。其中```codeConfiguration```是我们前面定义的异常码配置类，对应于我们自定义的异常码配置文件，通过这种方式我们可以获得code对应的message。



## 自定义异常

通用异常处理一般是处理意料之外的异常，对于开发者有意抛出的异常，我们可以单独定义相应的异常类型便于使用和处理。

### 定义异常

```java
@Getter
public class HttpException extends RuntimeException {
    protected Integer code;
    protected Integer httpStatusCode = 500;
}

public class NotFoundHttpException extends HttpException {
  public NotFoundHttpException(int code) {
    this.code = code;
    this.httpStatusCode = 404;
  }
}
```

这里我们自定义了```HttpException```用于开发者主动抛出，```HttpException```继承自```RuntimeException```，由于我们抛出异常后会被统一拦截处理，不希望编译期间进行检查，所以使用```RuntimeException```。



我们可以根据不同的HTTP状态码定义不同的异常类型，这里我们定义了```NotFoundHttpException```,他的状态码为404，实际调用时我们只需要抛出相应异常并填入自定义状态码即可。

### 抛出自定义异常

```java
@RestController
@Validated
public class UserController {
  @GetMapping("/user/{id}")
  public String getUser(@PathVariable @Max(20) Integer id, @RequestParam @Length(min = 2, max = 10) String name) {
    throw new NotFoundHttpException(ExceptionCode.RESOURCE_NOT_FOUND);
  }
```

假设没有找到用户，我们直接抛出```NotFoundHttpException```并填入自定义code即可完成错误处理。



### 全局异常处理

```java
@RestControllerAdvice
public class GlobalExceptionAdvice {
  @Autowired
  private ExceptionCodeConfiguration codeConfiguration;
  
  @ExceptionHandler(HttpException.class)
  public ResponseEntity<UnifyResponse> handleHttpException(HttpServletRequest req, HttpException e) {
    int code = e.getCode();
    UnifyResponse response = new UnifyResponse(code, codeConfiguration.getMessage(code), this.getRequest(req));
    HttpHeaders httpHeaders = new HttpHeaders();
    httpHeaders.setContentType(MediaType.APPLICATION_JSON);
    HttpStatus httpStatus = HttpStatus.resolve(e.getHttpStatusCode());
    return new ResponseEntity<>(response, httpHeaders, httpStatus);
  }
  
  // 获取请求方法和路径
  private String getRequest(HttpServletRequest req) {
    String url = req.getRequestURI();
    String method = req.getMethod();
    return method + " " + url;
  }
}
```

自定义```HttpException```的处理比较特殊，因为不同的HttpException要返回不同的HttpStatusCode，所有需要使用```ResponseEntity<UnifyResponse>```进行返回。



如上文所示，我们需要添加```@RestControllerAdvice```和```  @ExceptionHandler(HttpException.class)```注解，其次我们需要分别定义```UnifyResponse```，```httpHeaders```和```httpStatus```，最终将结果包装进```ResponseEntity```进行返回。



## 参数校验异常

参数校验分为两类，一类是```PathVariables,RequesetParams```的校验，返回```ConstraintViolationException```;另一类是```RequestBody```的校验，返回```MethodArgumentNotValidException```。



我们需要分别定义这两类异常的异常处理方法并获取信息返回```UnifyResponse```。

```java
@RestControllerAdvice
public class GlobalExceptionAdvice {
  @Autowired
  private ExceptionCodeConfiguration codeConfiguration;

  @ExceptionHandler(ConstraintViolationException.class)
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  public UnifyResponse handleConstraintException(HttpServletRequest req, ConstraintViolationException e) {
    StringBuilder messageBuilder = new StringBuilder();
    e.getConstraintViolations().forEach(violation -> {
      String param = violation.getPropertyPath().toString().replaceAll("\\w+\\.", "");
      String message = violation.getMessage();
      messageBuilder.append(param).append(":").append(message).append(";");
    });
    return new UnifyResponse(ExceptionCode.PARAMS_ERROR, messageBuilder.toString(), this.getRequest(req));
  }

  @ExceptionHandler(MethodArgumentNotValidException.class)
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  public UnifyResponse handleMethodArgumentException(HttpServletRequest req, MethodArgumentNotValidException e) {
    StringBuilder messageBuilder = new StringBuilder();
    e.getBindingResult().getFieldErrors().forEach(fieldError -> {
      String field = fieldError.getField();
      String message = fieldError.getDefaultMessage();
      messageBuilder.append(field).append(":").append(message).append(";");
    });
    return new UnifyResponse(ExceptionCode.PARAMS_ERROR, messageBuilder.toString(), this.getRequest(req));
  }

  // 获取请求方法和路径
  private String getRequest(HttpServletRequest req) {
    String url = req.getRequestURI();
    String method = req.getMethod();
    return method + " " + url;
  }
}
```

```ConstraintViolationException```通过```getConstraintViolations()```返回```Set<ConstraintViolation<?>>```集合，我们使用forEach和StringBuilder遍历和收集相应的信息。



```ConstraintViolation```的```getPropertyPath().toString()```会返回```MethodName.Param```格式，我们不希望给出```MethodName```，于是使用```replaceAll()```将其替换为空。```getMessage()```获得该Param的校验失败信息。



```MethodArgumentNotValidException```通过```getFieldErrors()```获得```List<FieldError>```列表，我们同样使用forEach和StringBuilder遍历和收集信息。



```FieldError```的```getField(),getDefaultMessage()```获得DTO对象内相应的字段和校验失败信息。



最终我们将信息打包成```UnifyResponse```进行返回。



> 源代码：https://github.com/PeterWangYong/blog-code/tree/master/handle-exception



