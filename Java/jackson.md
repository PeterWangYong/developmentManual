## 介绍

Jackson是流行的Java Json解析器



## 基本使用

### Json和对象之间转换

```java
/**
 * json和对象之间序列化和反序列化
 * new ObjectMapper();
 * objectMapper.writeValue(new File("target/car.json"), basic);
 * objectMapper.writeValueAsString(basic);
 * objectMapper.readValue(json, Basic.class)
 * objectMapper.readValue(new File("target/car.json"), Basic.class);
 *
 */
@Data
@NoArgsConstructor(force = true)
@RequiredArgsConstructor
public class Basic {
  private final int id;
  private final String name;

  public static void main(String[] args) throws IOException {
    ObjectMapper objectMapper = new ObjectMapper();
    // {"id":1,"name":"John"}
    Basic basic = new Basic(1, "John");
    String json = objectMapper.writeValueAsString(basic);
    System.out.println(json);
    // Basic类必须有无参构造和getter/setter方法 Basic(id=1, name=John)
    Basic basic1 = objectMapper.readValue(json, Basic.class);
    System.out.println(basic1);
  }
}
```

### Json和JsonNode对象之间转换

```java
/**
 * 将json构造为JsonNode树
 * JsonNode jsonNode = objectMapper.readTree(json)
 * jsonNode.get(String fieldName)
 */
@Data
public class Basic1 {
  private final int id;
  private final String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    // John
    String json = "{\"id\":1,\"name\":\"John\"}";
    JsonNode jsonNode = objectMapper.readTree(json);
    String name = jsonNode.get("name").asText();
    System.out.println(name);
  }
}
```

### 自定义Json转换对象

```java
/**
 * 自定义Json转换的类型
 * objectMapper.readValue(String json, TypeReference<T> valueTypeRef)
 */
@Data
@NoArgsConstructor(force = true)
@RequiredArgsConstructor
public class Basic2 {
  private final int id;
  private final String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    // [Basic2(id=1, name=John), Basic2(id=2, name=Diana)]
    List<Basic2> list = Arrays.asList(new Basic2(1, "John"), new Basic2(2, "Diana"));
    String json = objectMapper.writeValueAsString(list);
    List<Basic2> list1 = objectMapper.readValue(json, new TypeReference<List<Basic2>>() {});
    System.out.println(list1);
    // {id=3, name=Susan}
    Basic2 basic2 = new Basic2(3, "Susan");
    String json2 = objectMapper.writeValueAsString(basic2);
    Map<String, Object> map = objectMapper.readValue(json2, new TypeReference<Map<String, Object>>() {});
    System.out.println(map);
  }
}
```

## 常用配置

### 忽略无法对应的属性

```java
/**
 * 自动忽略无法对应的属性
 * objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
 */
@Data
@NoArgsConstructor(force = true)
@RequiredArgsConstructor
public class Basic3 {
  private final int id;
  private final String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    // Basic3(id=1, name=John)
    objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    String json = "{\"id\":1,\"name\":\"John\",\"age\":\"18\"}";
    Basic3 basic3 = objectMapper.readValue(json, Basic3.class);
    System.out.println(basic3);

    // age: 18
    int age = objectMapper.readTree(json).get("age").asInt();
    System.out.println("age: " + age);
  }
}
```

### 设置日期格式

```java
/**
 * 设置日期格式
 * objectMapper.setDateFormat(DateFormat dateFormat);
 */
@Data
public class Basic4 {
  private final int id;
  private final String name;
  private final Date date;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    // {"id":1,"name":"John","date":"2020-04-25"}
    DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
    objectMapper.setDateFormat(df);
    Basic4 basic4 = new Basic4(1, "John", new Date());
    String json = objectMapper.writeValueAsString(basic4);
    System.out.println(json);
  }
}
```

### 开启Wrapping

正常是{"id": xx, "name": xx},开启Wrapping后在外面嵌套一层即{"Basic9": {"id": xx, "name": xx}}

```java
/**
 * 开启Wrapping并指定根名称
 * objectMapper.enable(SerializationFeature.WRAP_ROOT_VALUE);  开启Wrapping，默认根名为Class.getName() [Basic9]
 * @JsonRootName("basic") 自定义根名称
 *
 */
@Data
@JsonRootName("basic")
public class Basic9 {
  private final int id;
  private final String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
    Basic9 basic9 = new Basic9(1, "John");
    // {"basic":{"id":1,"name":"John"}}
    String json = objectMapper.writeValueAsString(basic9);
    System.out.println(json);
  }
}
```

### 停止使用注解

```java
/**
 * 停止使用注解
 * objectMapper.disable(MapperFeature.USE_ANNOTATIONS);
 */
@Data
public class Basic18 {
  private final int id;
  @JsonIgnore
  private final String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.disable(MapperFeature.USE_ANNOTATIONS);
    Basic18 basic18 = new Basic18(1, "John");
    //  {"id":1,"name":"John"}
    String json = objectMapper.writeValueAsString(basic18);
    System.out.println(json);
  }
}
```



## 常用注解

### @JsonAnyGetter,@JsonAnySetter

```java
/**
 * @JsonAnySetter 将JSON中不能对应的字段统一放在Map中，只能有一个
 * @JsonAnyGetter 将Map中的字段作为实例对象的属性展开到JSON，只能有一个
 */
@Data
@NoArgsConstructor(force = true)
@RequiredArgsConstructor
public class Basic5 {
  private final int id;
  private final Map<String, Object> map = new HashMap<>();

  @JsonAnyGetter
  public Map<String, Object> getMap() {
    return map;
  }

  @JsonAnySetter
  public void setMap(String key, Object value) {
    map.put(key, value);
  }


  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic5 basic5 = new Basic5(1);
    basic5.setMap("name", "John");
    basic5.setMap("age", 18);
    //  {"id":1,"name":"John","age":18}
    String json = objectMapper.writeValueAsString(basic5);
    System.out.println(json);
    //  Basic5(id=1, map={name=John, age=18})
    Basic5 basic51 = objectMapper.readValue(json, Basic5.class);
    System.out.println(basic51);
  }
}
```

### @JsonGetter,@JsonSetter

```java
/**
 * @JsonGetter 标记一个方法为Getter
 * @JsonSetter 标记一个方法为Setter
 */
@Data
@NoArgsConstructor(force = true)
@RequiredArgsConstructor
public class Basic6 {
  private final int id;
  @NonNull
  private String name;

  @JsonGetter("name")
  public String getThisName() {
    return name;
  }

  @JsonSetter("name")
  public void setThisName(String name) {
    this.name = name;
  }

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic6 basic6 = new Basic6(1, "John");
    //  {"id":1,"name":"John"}
    String json = objectMapper.writeValueAsString(basic6);
    System.out.println(json);
    //  Basic6(id=1, name=John)
    Basic6 basic61 = objectMapper.readValue(json, Basic6.class);
    System.out.println(basic61);
  }
}
```

### @JsonPropertyOrder

```java
/**
 * 定义属性在Json中的顺序
 * @JsonPropertyOrder(String[] value) 自定义顺序
 * @JsonPropertyOrder(boolean alphabetic) 按字母排序
 */
@Data
//@JsonPropertyOrder({"name", "id"})
@JsonPropertyOrder(alphabetic = true)
public class Basic7 {
  private final int id;
  @NonNull
  private String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic7 basic7 = new Basic7(1, "John");
    //  {"name":"John","id":1}
    String json = objectMapper.writeValueAsString(basic7);
    System.out.println(json);
  }
}
```

### @JsonValue

```java
/**
 * 定义一个方法用于生成Json
 * @JsonValue
 */
@Data
public class Basic8 {
  private final int id;
  private final String name;

  @JsonValue
  public Map<String, Object> getJson() {
    Map<String, Object> map = new HashMap<>();
    map.put("Basic8_id", id);
    map.put("Basic8_name", name);
    return map;
  }

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic8 basic8 = new Basic8(1, "John");
    //  {"Basic8_name":"John","Basic8_id":1}
    String json = objectMapper.writeValueAsString(basic8);
    System.out.println(json);
  }
}
```

### @JsonRootName

```java
/**
 * 开启Wrapping并指定根名称
 * objectMapper.enable(SerializationFeature.WRAP_ROOT_VALUE);  开启Wrapping，默认根名为Class.getName() [Basic9]
 * @JsonRootName("basic") 自定义根名称
 *
 */
@Data
@JsonRootName("basic")
public class Basic9 {
  private final int id;
  private final String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
    Basic9 basic9 = new Basic9(1, "John");
    String json = objectMapper.writeValueAsString(basic9);
    System.out.println(json);
  }
}
```

### @JsonCreator

```java
/**
 * 自定义反序列化构造器，如果属性不匹配可以使用@JsonProperty
 * @JsonCreator
 * @JsonProperty("id")
 *
 */
@Data
public class Basic10 {
  private final int id;
  private final String name;

  @JsonCreator
  public Basic10(@JsonProperty("id") int id, @JsonProperty("the_name") String name) {
    this.id = id;
    this.name = name;
  }

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    String json = "{\"id\":1,\"the_name\":\"John\"}";
    //  Basic10(id=1, name=John)
    Basic10 basic10 = objectMapper.readValue(json, Basic10.class);
    System.out.println(basic10);
  }
}
```

### @JsonAlias

```java
/**
 * 定义反序列化时和POJO成员变量匹配的Json属性别名
 * @JsonAlias({"the_name"}) Json中的the_name,name属性都可以映射到POJO
 */
@Data
public class Basic11 {
  private final int id;
  @JsonAlias({"the_name"})
  private final String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    String json = "{\"id\":1,\"the_name\":\"John\"}";
    //  Basic10(id=1, name=John)
    Basic10 basic10 = objectMapper.readValue(json, Basic10.class);
    System.out.println(basic10);
  }
}
```

### @JsonIgnoreProperties, @JsonIgnore

```java
/**
 * 忽略部分字段
 * @JsonIgnoreProperties({"id"})
 * @JsonIgnore
 */
@Data
@JsonIgnoreProperties({"id"})
@NoArgsConstructor(force = true)
@RequiredArgsConstructor
public class Basic12 {
  private final int id;
  private final String name;
  @JsonIgnore
  private final int age;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic12 basic12 = new Basic12(1, "John", 18);
    //  {"name":"John"}
    String json = objectMapper.writeValueAsString(basic12);
    System.out.println(json);
    //  Basic12(id=0, name=John, age=0)
    Basic12 basic121 = objectMapper.readValue(json, Basic12.class);
    System.out.println(basic121);
  }
}
```

### @JsonIgnoreType

```java
/**
 * 忽略某个类型字段
 * @JsonIgnoreType
 */
@Data
public class Basic13 {
  private final int id;
  private final String name;
  private final Role role;

  @Data
  @JsonIgnoreType
  private static class Role {
    private final int id;
    private final String name;
  }


  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic13 basic13 = new Basic13(1, "John", new Role(1, "admin"));
    //  {"id":1,"name":"John"}
    String json = objectMapper.writeValueAsString(basic13);
    System.out.println(json);
  }
}
```

### @JsonInclude

```java
/**
 * 忽略NULL值
 * @JsonInclude(JsonInclude.Include.NON_NULL)
 */
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
@NoArgsConstructor(force = true)
@RequiredArgsConstructor
public class Basic14 {
  private final int id;
  private String name;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic14 basic14 = new Basic14(1);
    String json = objectMapper.writeValueAsString(basic14);
    System.out.println(json);
    Basic14 basic141 = objectMapper.readValue(json, Basic14.class);
    System.out.println(basic141);
  }
}
```

### @JsonProperty

```java
/**
 * 自定义Json属性和Getter/Setter的匹配关系
 * @JsonProperty("name")
 * 作用同@JsonGetter @JsonSetter
 */
@Data
public class Basic15 {
  private final int id;
  @NonNull
  private String name;

  @JsonProperty("name")
  public String getTheName() {
    return name;
  }

  @JsonProperty("name")
  public void setTheName(String name) {
    this.name = name;
  }

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic15 basic15 = new Basic15(1, "John");
    // {"id":1,"name":"John"}
    String json = objectMapper.writeValueAsString(basic15);
    System.out.println(json);
  }
}
```

### @JsonFormat

```java
/**
 * 格式化日期时间类型
 * @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd")
 */
@Data
public class Basic16 {
  private final int id;
  private final String name;
  @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd")
  private final Date date;

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic16 basic16 = new Basic16(1, "John", new Date());
    // {"id":1,"name":"John","date":"2020-04-26"}
    String json = objectMapper.writeValueAsString(basic16);
    System.out.println(json);
  }
}
```

### @JsonUnwrapped

```java
/**
 * 展开嵌套
 * @JsonUnwrapped
 */
@Data
public class Basic17 {
  private final int id;
  private final String name;
  @JsonUnwrapped
  private final Role role;

  @Data
  private static class Role {
    private final int role_id;
    private final String role_name;
  }

  public static void main(String[] args) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    Basic17 basic17 = new Basic17(1, "John", new Role(1, "admin"));
    //  {"id":1,"name":"John","role_id":1,"role_name":"admin"}
    String json = objectMapper.writeValueAsString(basic17);
    System.out.println(json);
  }
}
```

