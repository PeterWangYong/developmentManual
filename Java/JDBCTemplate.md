## 介绍
由于JDBC存在大量样板代码,SpringJDBCTemplate对JDBC进行了封装，提供更简洁的方法供开发者使用。

## 常用方法
### 查询类

#### 查询简单数据
简单数据即查询结果中只有一个“字段名”
```java
// 查询一个
T queryForObject(String sql, Class<T> type)
T queryForObject(String sql, Object[] args, Class<T> type)
T queryForObject(String sql, Class<T> type, Object... args)

// 查询多个
List<T> queryForList(String sql, Class<T> type)
List<T> queryForList(String sql, Object[] args, Class<T> type)
List<T> queryForList(String sql, Class<T> type, Object... args)
```

#### 查询复杂数据
复杂数据即查询结果中存在多个“字段名”
##### Map封装
使用Map封装”字段名“和”字段值“
```java
// 查询一个
Map<String,Object> queryForMap(String sql);
Map<String,Object> queryForMap(String sql, Object[] args);
Map<String,Object> queryForMap(String sql, Object... args);

// 查询多个
List<Map<String,Object>> queryForList(String sql);
List<Map<String,Object>> queryForList(String sql, Object[] args);
List<Map<String,Object>> queryForList(String sql, Object... args);
```

##### 实体对象封装
如果要将查询结果封装为一个对象，则需要提供映射关系；JDBCTemplate提供了RowMapper接口供开发者实现自己的映射
```java
// 查询一个
T queryForObject(String sql, RowMapper<T> mapper);
T queryForObject(String sql, Object[] args, RowMapper<T> mapper);
T queryForObject(String sql, RowMapper<T> mapper, Object... args);
// 查询多个
List<T> query(String sql, RowMapper<T> mapper);
List<T> query(String sql, Object[] args, RowMapper<T> mapper);
List<T> query(String sql, RowMapper<T> mapper, Object... args);
```

### 增删改
#### 执行一条语句
执行一条语句，返回值为“影响的行数”
```java
int update(String sql, Object[] args)
int update(String sql, Object... args)
```

#### 执行多条语句
批量执行多条语句，返回值为“每条语句影响的行数”

##### 多条独立语句
```java
int[] batchUpdate(String[] sql)
```

##### 多条同构语句
同构语句即SQL主体相同但传入的值不同

```java
int[] batchUpdate(String sql, List<Object[]> args)
```

### DDL
DDL为数据库定义语句，比如创建表等
```java
void execute(String sql)
```
> 实际上execute可以执行任何语句，但考虑便捷性通常只用来执行不需要返回值的DDL语句

## 使用案例
### 配置依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

### 定义实体类
```java
public class User {

  private Integer id;
  private String username;
  private String password;

  public Integer getId() {
    return id;
  }

  public void setId(Integer id) {
    this.id = id;
  }

  public String getUsername() {
    return username;
  }

  public void setUsername(String username) {
    this.username = username;
  }

  public String getPassword() {
    return password;
  }

  public void setPassword(String password) {
    this.password = password;
  }

  @Override
  public String toString() {
    return "User{" +
      "id=" + id +
      ", username='" + username + '\'' +
      ", password='" + password + '\'' +
      '}';
  }
}
```

### 获取对象
```java
@RestController
public class UserController {
  
  @Autowired
  private JdbcTemplate jdbcTemplate;
```

### 执行语句
```java
@RestController
public class UserController {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  // 查询简单数据：单条数据
  @GetMapping("/user/count")
  public int count() {
    int count = jdbcTemplate.queryForObject("select count(*) from user", Integer.class);
    return count;
  }

  // 查询简单数据：单条数据
  @GetMapping("/user/name")
  public String name() {
//    String username = jdbcTemplate.queryForObject("select user_name from user where user_id=?", new Object[]{1}, String.class);
    String username = jdbcTemplate.queryForObject("select user_name from user where user_id=?", String.class, 1);
    return username;
  }

  // 查询简单数据：多条数据
  @GetMapping("/user/name_list")
  public List<String> nameList() {
//    List<String> nameList = jdbcTemplate.queryForList("select user_name from user", String.class);
//    List<String> nameList = jdbcTemplate.queryForList("select user_name from user where user_id = ?", new Object[]{1},String.class);
    List<String> nameList = jdbcTemplate.queryForList("select user_name from user where user_id = ?", String.class, 1);
    return nameList;
  }

  // 查询复杂数据：单条数据 Map封装
  @GetMapping("/user/info")
  public Map<String, Object> info() {
    Map<String, Object> info = jdbcTemplate.queryForMap("select * from user where user_id = 1");
//    Map<String, Object> info = jdbcTemplate.queryForMap("select * from user where user_id = ?", new Object[]{1});
//    Map<String, Object> info = jdbcTemplate.queryForMap("select * from user where user_id = ?", 1);
    return info;
  }

  // 查询复杂数据：多条数据 Map封装
  @GetMapping("/user/infoList")
  public List<Map<String, Object>> infoList() {
//    List<Map<String, Object>> infoList = jdbcTemplate.queryForList("select * from user");
//    List<Map<String, Object>> infoList = jdbcTemplate.queryForList("select * from user where user_id=?", new Object[]{1});
    List<Map<String, Object>> infoList = jdbcTemplate.queryForList("select * from user where user_id=?", 1);
    return infoList;
  }

  // 查询复杂数据：单条数据 实体对象封装
  @GetMapping("/user/entity")
  public User entity() {
//    User entity = jdbcTemplate.queryForObject("select * from user where user_id=1", new UserMapper());
//    User entity = jdbcTemplate.queryForObject("select * from user where user_id=?", new Object[]{1}, new UserMapper());
    User entity = jdbcTemplate.queryForObject("select * from user where user_id=?", new UserMapper(), 1);
    return entity;
  }

  // 查询复杂数据：多条数据 实体对象封装
  @GetMapping("/user/entityList")
  public List<User> entityList() {
//    List<User> entityList = jdbcTemplate.query("select * from user", new UserMapper());
//    List<User> entityList = jdbcTemplate.query("select * from user where user_id=?", new Object[]{1}, new UserMapper());
    List<User> entityList = jdbcTemplate.query("select * from user where user_id=?", new UserMapper(), 1);
    return entityList;
  }

  // 查询复杂数据：映射关系内部类
  class UserMapper implements RowMapper<User> {
    @Override
    public User mapRow(ResultSet resultSet, int i) throws SQLException {
      User user = new User();
      user.setId(resultSet.getInt("user_id"));
      user.setUsername(resultSet.getString("user_name"));
      user.setPassword(resultSet.getString("user_name"));
      return user;
    }
  }

  // 执行一条语句：插入数据
  @GetMapping("/user/insert")
  public int insert() {
//    jdbcTemplate.update("insert into user(user_id, user_name, pass_word) value(3, 'Diana', '123')");
//    int row = jdbcTemplate.update("insert into user(user_id, user_name, pass_word) value(?, ?, ?)", new Object[]{3, "Diana", "123"});
    int row = jdbcTemplate.update("insert into user(user_id, user_name, pass_word) value(?, ?, ?)", 3, "Diana", "123");
    return row;
  }

  // 执行一条语句：更新数据
  @GetMapping("/user/update")
  public int update() {
//    int row = jdbcTemplate.update("update user set pass_word=? where user_id=?", new Object[]{"123", 3});
    int row = jdbcTemplate.update("update user set pass_word=? where user_id=?", "123", 3);
    return row;
  }

  // 执行一条语句：删除数据
  @GetMapping("/user/delete")
  public int delete() {
//    int row = jdbcTemplate.update("delete from user where user_id=?", new Object[]{3});
    int row = jdbcTemplate.update("delete from user where user_id=?", 3);
    return row;
  }

  // 执行多条语句：多条独立语句
  @GetMapping("/user/insert_and_delete")
  public int[] insertAndDelete() {
//    int[] rows = jdbcTemplate.batchUpdate(new String[]{"insert into user value(4, 'peter', '123')", "delete from user where user_id=4"});
    int[] rows = jdbcTemplate.batchUpdate("insert into user value(4, 'peter', '123')", "delete from user where user_id=4");
    return rows;
  }

  // 执行多条语句：多条重构语句
  @GetMapping("/user/insert_many")
  public int[] insertMany() {
    List<Object[]> args = new ArrayList<>();
    args.add(new Object[]{4, "peter", "123"});
    args.add(new Object[]{5, "angel", "123"});
    int[] rows = jdbcTemplate.batchUpdate("insert into user value(?, ?, ?)", args);
    return rows;
  }

  // 执行DDL语句: 创建一张表
  @GetMapping("/create_table")
  public String createTable() {
    jdbcTemplate.execute("create table user_rank(id int(11), user_name varchar(255), rank int(11))");
    return "success";
  }
```

