mybatisplus



## 什么是mybatisplus？

MybatisPlus是一款优秀的国产持久层框架，基于Mybatis实现了基础SQL和CRUD操作的封装，帮助开发者简化数据库操作。MybatisPlus基于Mybatis实现了功能的扩展和增强，同时兼容现有的Mybatis操作。



## 基础准备

>  案例基于SpringBoot来实现

### pom.xml

```xml
    <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.3.1.tmp</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.11</version>
    </dependency>
```

### 项目配置

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mybatisplus?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=UTF8
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

### 数据表

```sql
create table user(
    id int primary key auto_increment,
    name varchar(11),
    age int
);

insert into user values (1, '张三', 20), (2, '李四', 22);
```

### 实体类

```java
@Data
public class User {
  private Integer id;
  private String name;
  private Integer age;
}
```

### Mapper接口

```java
@Component
public interface UserMapper {
  User findById(int id);
}
```

### @MapperScan

> Mabatisplus需要根据MapperScan知道到哪里找到Mapper接口从而生成动态代理对象注入IOC容器

```java
@SpringBootApplication
@MapperScan("com.peter.mybatisplus.mapper")
public class MybatisPlusApplication {

  public static void main(String[] args) {
    SpringApplication.run(MybatisPlusApplication.class, args);
  }
}
```



## 兼容Mybatis操作

1. 定义resources/mapper/userMapper.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <mapper namespace="com.peter.mybatisplus.mapper.UserMapper">
     <select id="findById" resultType="com.peter.mybatisplus.model.User">
      select * from user where id = #{id}
     </select>
   </mapper>
   
   ```

2. 测试userMapper接口

   ```java
   @SpringBootTest
   public class UserMapperTest {
   
     @Autowired
     UserMapper userMapper;
   
     @Test
     public void findByIdTest() {
       User user = userMapper.findById(1);
       System.out.println(user);
     }
   }
   ```

## 继承BaseMapper<>类

如果不想自己编写SQL语句，可以继承BaseMapper中预定义的方法实现CRUD操作。

我们通过使用各种**实体类注解**来告诉MP如何生成SQL语句实现数据库操作。

```java
@Component
public interface UserMapper extends BaseMapper<User> {}
```



## 实体类注解

实体类注解（字段注解）用于标记实体类成员变量，从而影响实体对象和数据库之间的映射以及MP对数据库的操作。

### @TableName

> value: 映射数据库表名称

```java
@TableName("user")
public class Account {
  @TableId()
  private Integer id;
  private String name;
  private Integer age;
}
```



### @TableId

> value: 映射主键字段
>
> type: 定义主键类型

```markdown
# type
1. AUTO：ID自增
2. NONE：默认，同INPUT
3. INPUT：用户自定义，如果未定义则同ASSIGN_ID
4. ASSIGN_ID：使用雪花算法生成ID（注意使用Long类型，数据库使用bigint）
5. ASSIGN_UUID：使用UUID进行赋值，主键类型为String
```

```java
public class User {
  @TableId(type = IdType.AUTO)
  private Long id;
  private String name;
  private Integer age;
}
```



### @TableField

> value: 映射非主键字段
>
> exist: 是否为数据库表字段
>
> select: 是否查询该字段
>
> fill: 字段自动填充策略

#### 如何自动填充create_time和update_time?

1. 实体类添加字段和注解

   ```java
   @Data
   public class User {
     @TableId(type = IdType.AUTO)
     private Long id;
     private String name;
     private Integer age;
     @TableField(fill = FieldFill.INSERT)
     private Date createTime;
     @TableField(fill = FieldFill.INSERT_UPDATE)
     private Date updateTime;
   }
   ```

2. 创建填充处理器

   > MybatisPlus检测到@TableField(fill) 后会注入MetaObjectHandler接口对象

   ```java
   @Component
   public class MyMetaObjectHandler implements MetaObjectHandler {
     @Override
     public void insertFill(MetaObject metaObject) {
       this.setFieldValByName("createTime", new Date(), metaObject);
       this.setFieldValByName("updateTime", new Date(), metaObject);
     }
   
     @Override
     public void updateFill(MetaObject metaObject) {
       this.setFieldValByName("updateTime", new Date(), metaObject);
     }
   }
   ```

3. 测试

   ```java
     @Test
     public void saveTest() {
       User user = new User();
       user.setName("王五");
       user.setAge(18);
       System.out.println("userMapper = " + userMapper.insert(user));
     }
   ```

### @Version

> 添加乐观锁机制
>
> 乐观锁的本质就是这么一条SQL语句：update table set field = value, version = version+1 where id = ? and version = ?
>
> 乐观锁是为了保证业务多线程运行情况下数据准确性，是业务层面的逻辑，和数据库机制无关。

1. 数据库添加version字段并赋值为1

2. 实体类添加version成员变量并添加@Version

   ```java
   @Data
   public class User {
     @TableId(type = IdType.AUTO)
     private Integer id;
     private String name;
     private Integer age;
     @TableField(fill = FieldFill.INSERT)
     private Date createTime;
     @TableField(fill = FieldFill.INSERT_UPDATE)
     private Date updateTime;
     @Version
     private Integer version;
   }
   ```

3. 将OptimisticLockerInterceptor对象添加到IOC容器

   ```java
   @Configuration
   public class MyBatisPlusConfig {
   
     @Bean
     public OptimisticLockerInterceptor optimisticLockerInterceptor() {
       return new OptimisticLockerInterceptor();
     }
   }
   ```

4. 测试

   ```java
     @Test
     void updateTest() {
       User user = userMapper.selectById(3);
       user.setName("赵四");
   
       User user1 = userMapper.selectById(3);
       user1.setName("赵武");
   
       userMapper.updateById(user1); // 成功
       userMapper.updateById(user);  // 失败
     }
   ```

   > 对于MP来说会生成这么一条语句,所以我们传入的user对象里面如果没有version变量(null)则乐观锁不会生效
   >
   > 不过这里存个疑问：MP居然知道我们更新了什么字段然后只SET那个字段，这是怎么做到的？
   >
   > ==>  Preparing: UPDATE user SET name=?, update_time=?, version=? WHERE id=? AND version=? 
   > ==> Parameters: 赵武(String), 2020-05-26 09:55:07.715(Timestamp), 2(Integer), 3(Integer), 1(Integer)

### @EnumValue

> 当需要将数据库字段映射为枚举对象时，使用该注解指定要映射的枚举类型成员变量

1. 数据库添加字段status

2. 定义枚举类型并在需要映射的成员变量上添加@EnumValue

   > 如果映射的成员变量出现了多个重复值，那么取第一个定义的枚举对象(例如：WORK(1, "xxx"), WORK2(1, "xxx1"))

   ```java
   public enum StatusEnum {
   
     WORK(1, "工作"),
     REST(0, "休息");
   
     @EnumValue
     private Integer code;
     private String msg;
   
     StatusEnum(Integer code, String msg) {
       this.code = code;
       this.msg = msg;
     }
   }
   ```

3. 添加实体类成员变量

   ```java
   @Data
   public class User {
     @TableId(type = IdType.AUTO)
     private Integer id;
     private String name;
     private Integer age;
     @TableField(fill = FieldFill.INSERT)
     private Date createTime;
     @TableField(fill = FieldFill.INSERT_UPDATE)
     private Date updateTime;
     @Version
     private Integer version;
     private StatusEnum status;   // 添加StatusEnum类型的成员变量
   }
   ```

4. 添加枚举包配置

   > MP在注入枚举对象的时候需要定位到具体是哪个类

   ```yml
   mybatis-plus:
     configuration:
       log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
     type-enums-package: com.peter.mybatisplus.enums  # 添加枚举包配置
   ```

5. 测试

   ```java
     @Test
     public void findByIdTest() {
       User user = userMapper.selectById(1);
       System.out.println(user);
     }
    	// User(id=1, name=张三, age=20, createTime=null, updateTime=null, version=null, status=WORK)
   
     @Test
     public void saveTest() {
       User user = new User();
       user.setName("王五");
       user.setAge(18);
       user.setStatus(StatusEnum.REST);  // 使用枚举对象
       System.out.println("userMapper = " + userMapper.insert(user));
     }
   ```

> 我们也可以不使用@EnumValue而让枚举类继承IEnum\<T\>接口获得相同的效果
>
> 重写getValue()方法指定映射的变量

```java
public enum StatusEnum2 implements IEnum {

  WORK(1, "工作"),
  REST(0, "休息");

  private Integer code;
  private String msg;

  StatusEnum2(Integer code, String msg) {
    this.code = code;
    this.msg = msg;
  }

  @Override
  public Serializable getValue() {
    return this.code;
  }
}
```

### @TableLogic

> 映射逻辑删除字段

1. 数据库添加deleted字段

2. 实体类添加成员变量

   ```java
   @Data
   public class User {
     @TableId(type = IdType.AUTO)
     private Integer id;
     private String name;
     private Integer age;
     @TableField(fill = FieldFill.INSERT)
     private Date createTime;
     @TableField(fill = FieldFill.INSERT_UPDATE)
     private Date updateTime;
     @Version
     private Integer version;
     private StatusEnum status;
     @TableLogic
     private Integer deleted;  // 添加逻辑删除映射变量
   }
   ```

3. 配置逻辑删除要赋予的值

   ```yml
   mybatis-plus:
     configuration:
       log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
     type-enums-package: com.peter.mybatisplus.enums
     global-config:
       db-config:
         logic-delete-value: 1  # 删除时赋1
         logic-not-delete-value: 0 # 不删除赋0
   ```

4. 测试

   ```java
   // UPDATE user SET deleted=1 WHERE id=? AND deleted=0   
   @Test
     void deleteTest() {
       userMapper.deleteById(3);
     }
   	
   	// SELECT id,name,age,create_time,update_time,version,status,deleted FROM user WHERE deleted=0 
     @Test
     void findAll() {
       List<User> users = userMapper.selectList(null);
       System.out.println("users = " + users);
     }
   ```

   

## CRUD操作

### 查询操作

```java
// 根据Wrapper条件查询
selectList()  		// 根据条件<wrapper>查询，返回List<T>
selectOne()   		// 根据条件<wrapper>查询，返回T
selectCount() 		// 根据条件<wrapper>查询，返回Integer
selectMaps()			// 根据条件<wrapper>查询，返回List<Map<String, Object>>
selectObjs()			// 根据条件<wrapper>查询，返回List<Object>  Object仅为第一个字段的值

// 根据Map条件查询
selectByMap()		// 根据Map<String, Object>查询，返回List<T>

// 根据主键(ID)查询
selectById()			// 根据id查询，返回T
selectBatchIds() // 根据idList查询，返回List<T>
  
// 根据Wrapper分页查询
selectPage()			// 根据条件<wrapper>查询，返回<E extends IPage<T>> E
selectMapsPage()	// 根据条件<wrapper>查询，返回<E extends IPage<Map<String, Object>>> E
```

#### Wrapper条件查询

Wrapper是MP中的条件构造器，通过Wrapper对象我们构造SQL中的各种条件，MP再将这些条件解析为SQL进行查询。

```java
  /**
   * Wrapper条件查询
   * Wrapper用于条件构造
   */
  @Test
  void selectByWrapper() {
    // Wrapper条件构造器
    QueryWrapper wrapper = new QueryWrapper();
    wrapper.eq("name", "张三");
//    wrapper.gt("age", 18);
//    wrapper.lt("age", 22);
//    Map<String, Object> map = new HashMap<>();
//    map.put("name", "张三");
//    map.put("age", 20);
//    wrapper.allEq(map);
//    wrapper.likeLeft("name", "三");
//    wrapper.orderByAsc("age")

    userMapper.selectList(null).forEach(System.out::println);
    userMapper.selectList(wrapper).forEach(System.out::println);
    userMapper.selectOne(wrapper);
    userMapper.selectCount(wrapper);
    userMapper.selectMaps(wrapper).forEach(System.out::println);
    userMapper.selectObjs(wrapper).forEach(System.out::println);
  }
```



#### Map条件查询

简化Wrapper.allEq()方法将Map直接传入作为条件查询。

```java
  /**
   * 根据Map条件查询
   * 类似Wrapper的allEq查询
   */
  @Test
  void selectByMap() {
    Map<String, Object> map = new HashMap<>();
    map.put("name", "张三");
    map.put("age", 20);
    userMapper.selectByMap(map).forEach(System.out::println);
  }
```



#### 根据主键(ID)进行查询

传入主键值进行查询。

```java
  /**
   * 根据主键(ID)值进行查询
   */
  @Test
  void selectById() {
    System.out.println(userMapper.selectById(3));
    userMapper.selectBatchIds(Arrays.asList(1,2,3)).forEach(System.out::println);
  }
```



#### Wrapper分页查询

使用Wrapper条件的同时传入Page分页模型对象，MP会使用LIMIT语句进行部分查询，LIMIT语句中的值随Page中配置而变化，业务上只需不断调整Page参数并传入查询语句即可实现分页查询。

1. 添加分页查询插件

   ```java
   @Configuration
   public class MyBatisPlusConfig {
   
     @Bean
     public PaginationInterceptor paginationInterceptor() {
       return new PaginationInterceptor();
     }
   }
   ```

2. 测试

   ```java
     /**
      * 分页查询
      * Page用于提供分页模型
      * 分页本质利用数据库的 LIMIT 语句
      */
     @Test
     void selectInPage() {
   
   //    IPage<User> page = new Page<>(2, 1);
   //    IPage<User> ret = userMapper.selectPage(page, null);
   //    System.out.println(ret.getSize());
   //    System.out.println(ret.getTotal());
   //    System.out.println(ret.getPages());
   //    ret.getRecords().forEach(System.out::println);
   
       IPage<Map<String, Object>> page = new Page<>(2, 1, false);
       IPage<Map<String, Object>> ret = userMapper.selectMapsPage(page, null);
       ret.getRecords().forEach(System.out::println);
     }
   }
   ```

#### 自定义SQL查询

如果默认查询无法满足要求（比如多表查询），可以自定义SQL语句实现查询。

```java
@Component
public interface UserMapper extends BaseMapper<User> {

  @Select("select name from user where id = #{id}")
  User findBySql(int id);
}

```



### 插入操作

```java
insert(T entity)
```

#### 插入一条数据

```java
  @Test
  public void saveTest() {
    User user = new User();
    user.setName("王五");
    user.setAge(18);
    user.setStatus(StatusEnum.REST);
    System.out.println("userMapper = " + userMapper.insert(user));
  }
```



### 修改操作

```java
updateById(T entity) // 根据entity中的主键(ID)更新
update(T entity, Wrapper) // 根据Wrapper条件进行更新
```

#### 根据主键(ID)更新

```java
  @Test
  void updateTest() {
    User user = userMapper.selectById(3);
    user.setName("赵四");

    User user1 = userMapper.selectById(3);
    user1.setName("赵武");

    userMapper.updateById(user1); // 成功
    userMapper.updateById(user);  // 失败
  }
```

#### 根据Wrapper条件进行更新

```java
  @Test
  void updateByWrapper() {
    User user = userMapper.selectById(2);
    user.setName("赵四");
    QueryWrapper wrapper = new QueryWrapper();
    wrapper.eq("age", 20);
    userMapper.update(user, wrapper);
  }
```



### 删除操作

```java
deleteById()
deleteBatchIds()
delete(Wrapper<T> wrapper)
deleteByMap(Map<String, Object> map)
```

#### 根据主键(ID)删除

```java
  @Test
  void deleteTest() {
    userMapper.deleteById(3);
    userMapper.deleteBatchIds(Arrays.asList(1,2));
  }
```

#### 根据Wrapper条件删除

```java
  @Test
  void deleteByWrapperTest() {
    QueryWrapper wrapper = new QueryWrapper();
    wrapper.eq("name", "张三");
    userMapper.delete(wrapper);
  }
```

#### 根据Map条件删除

```java
  @Test
  void deleteByMapTest() {
    Map<String, Object> map = new HashMap<>();
    map.put("name", "张三");
    userMapper.deleteByMap(map);
  }
```

## 代码生成器

MP支持根据数据表反向生成 **实体类、Mapper、Service、Controller** 代码，帮助开发者节省时间。

我们只需要引入生成器并做一些配置即可生成代码。

1. 引入依赖

   ```xml
       <dependency>
         <groupId>com.baomidou</groupId>
         <artifactId>mybatis-plus-generator</artifactId>
         <version>3.3.1.tmp</version>
       </dependency>
   
       <dependency>
         <groupId>org.apache.velocity</groupId>
         <artifactId>velocity</artifactId>
         <version>1.7</version>
       </dependency>
   ```

   > velocity是基于Java的模板引擎，MP默认使用velocity

2. 配置和执行

   ```java
   public class Executor {
     public static void main(String[] args) {
       // 创建generator对象
       AutoGenerator generator = new AutoGenerator();
   
       // 数据源配置
       DataSourceConfig dataSourceConfig = new DataSourceConfig();
       dataSourceConfig.setDbType(DbType.MYSQL);
       dataSourceConfig.setUrl("jdbc:mysql://localhost:3306/mybatisplus?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=UTF8");
       dataSourceConfig.setUsername("root");
       dataSourceConfig.setPassword("password");
       dataSourceConfig.setDriverName("com.mysql.cj.jdbc.Driver");
       generator.setDataSource(dataSourceConfig);
   
       // 全局配置
       GlobalConfig globalConfig = new GlobalConfig();
       globalConfig.setOutputDir(System.getProperty("user.dir") + "/src/main/java");
       globalConfig.setOpen(false);
       globalConfig.setAuthor("peter");
       globalConfig.setServiceName("%sService");
       generator.setGlobalConfig(globalConfig);
   
       // 包命名配置
       PackageConfig packageConfig = new PackageConfig();
       packageConfig.setParent("com.peter.mybatisplus");
       packageConfig.setModuleName("generator");
       packageConfig.setController("controller");
       packageConfig.setService("service");
       packageConfig.setServiceImpl("service.impl");
       packageConfig.setMapper("mapper");
       packageConfig.setEntity("entity");
       generator.setPackageInfo(packageConfig);
   
       // 代码生成策略配置
       StrategyConfig strategyConfig = new StrategyConfig();
       strategyConfig.setEntityLombokModel(true);
       strategyConfig.setNaming(NamingStrategy.underline_to_camel);
       strategyConfig.setColumnNaming(NamingStrategy.underline_to_camel);
       generator.setStrategy(strategyConfig);
   
       // 开始执行
       generator.execute();
   
     }
   ```

## 总结

MybatisPlus作为Mybatis的扩展和增强，提供了一组常用方法帮助开发者简化数据库操作，同时保留了对Mybatis的兼容性，当自带的功能无法满足时，开发者仍然可以使用Mybatis的功能进行开发。



> 源码：https://github.com/PeterWangYong/blog-code/tree/master/mybatis-plus



















