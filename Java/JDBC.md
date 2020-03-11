## JDBC概述
Java DataBase Connectivity(java 数据库连接)
JDBC规范，MySQL等驱动实现了JDBC规范

## MySQL驱动
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.48</version>
</dependency>
```

## JDBC入门案例
```java
package com.imooc.jdbc.demo1;
import org.junit.Test;
import java.sql.*;
import com.mysql.jdbc.Driver;


public class JDBCDemo1 {

  @Test
  public void demo1() {
    Connection conn = null;
    Statement stmt = null;
    ResultSet resultSet = null;
    try {
      // 1. 加载驱动
      DriverManager.registerDriver(new Driver());
      // 2. 获得连接
      conn = DriverManager.getConnection("jdbc:mysql://192.168.2.130:3306/jdbctest", "root", "password");
      // 3. 创建执行SQL语句的对象并且执行SQL
      String sql = "SELECT * FROM user";
      stmt = conn.createStatement();
      resultSet = stmt.executeQuery(sql);
      while(resultSet.next()) {
        int uid = resultSet.getInt("uid");
        String username = resultSet.getString("username");
        String password = resultSet.getString("password");
        String name = resultSet.getString("name");

        System.out.println(uid + "  " + username + "  " + password + "  " + name);
      }

    } catch (SQLException e) {
      e.printStackTrace();
    } finally {
      // 4. 释放资源
      if (resultSet != null) {
        try {
          resultSet.close();
        } catch (SQLException e) {
          e.printStackTrace();
        }
        resultSet = null;
      }

      if (stmt != null) {
        try {
          stmt.close();
        } catch (SQLException e) {
          e.printStackTrace();
        }
        stmt = null;
      }

      if (conn != null) {
        try {
          conn.close();
        } catch (SQLException e) {
          e.printStackTrace();
        }
        conn = null; // 垃圾回收机制更早回收对象
      }
    }
  }

}

```

## JDBC常用API
### DriverManager()
驱动管理类

主要作用：
- 注册驱动
    ```java
    // DriverManager.registerDriver(new Driver()); // 会导致驱动加载两次
    // 实际开发中注册驱动会使用如下方式
    // 加载类的同时类内部的静态代码块会自动执行注册
    Class.forName("com.mysql.jdbc.Driver");
    ```
- 获得连接
    ```java
    Connection getConnection(String url, String username, String password);
    // url写法: jdbc:mysql://localhost:3306/db_name
    // jdbc: 协议  mysql: 子协议
    // url简写: jdbc:mysql:///jdbc  连接本地 localhost:3306 简写
    ```

### Connection
连接对象

主要作用：
- 创建执行SQL语句的对象
    ```java
    Statement craeteStatement() //执行SQL语句，有SQL注入的漏洞存在
    PreparedStatement prepareStatement(String sql) //预编译SQL语句，解决了SQL注入漏洞
    CallableStatement prepareCall(String sql) //执行SQL中存储过程
    ```
- 进行事务的管理
    ```java
    setAutoCommit(boolean autoCommit) //设置事务是否自动提交
    commit() //事务提交
    rollback() //事务回滚
    ```

### Statement
- 执行SQL语句
    ```java
    boolean execute(String sql) // 执行SQL语句,执行select语句返回true，否则返回false
    ResultSet executeQuery(String sql) // 执行SQL中select语句
    int executeUpdate(String sql) // 执行SQL中insert/update/delete 语句
    ```
- 执行批处理操作
    ```java
    addBatch(String sql) // 添加到批处理
    executeBatch() // 执行批处理
    clearBatch() // 情况批处理
    ```

### ResultSet
结果集：查询语句（select）查询到的结果
- 获取查询到的结果
    ```java
    next() //判断是否有下一条记录并将光标向后移动
    getXXX() // 获取字段的值 getInt getString getObject ...
    ```

### 资源释放
JDBC程序运行完后，切记要释放程序在运行过程中，创建的那些与数据库进行交互的对象，这些对象通常是ResultSet,Statement和Connection对象。

特别是Connection对象，它是非常稀有的资源，用完后必须马上释放，Connection的使用原则是尽量晚创建，尽量早释放。

## CRUD操作
### 插入操作
```java
stmt = conn.createStatement();
String sql = "INSERT INTO user VALUES(null, 'eee', '123', '张三')";
int i = stmt.executeUpdate(sql);
if (i > 0) {
    System.out.println("插入成功");
}
```

### 更新操作
```java
stmt = conn.createStatement();
String sql = "UPDATE user SET username='qqq',password='456',name='赵六' WHERE id=4";
int i = stmt.executeUpdate(sql);
if (i > 0) {
    System.out.println("修改成功");
}
```

### 删除操作
```java
stmt = conn.createStatement();
String sql = "DELETE FROM user WHERE uid=4";
int i = stmt.executeUpdate(sql);
if (i > 0) {
    System.out.println("删除成功");
}
```

### 查询操作
#### 查询所有记录
```java
stmt = conn.createStatement();
String sql = "SELECT * FROM user";
resultSet = stmt.executeQuery(sql);
while (resultSet.next()) {
    System.out.println(resultSet.getInt("uid") + "  " + resultSet.getString("username") + "  " + resultSet.getString("password") + "  " + resultSet.getString("name"));
}

```
#### 查询一条记录
```java
stmt = conn.createStatement();
String sql = "SELECT * FROM user WHERE uid=2";
resultSet = stmt.executeQuery(sql);
if (resultSet.next()) {
    System.out.println(resultSet.getInt("uid") + "  " + resultSet.getString("username") + "  " + resultSet.getString("password") + "  " + resultSet.getString("name"));
}
```

## 抽取工具类
```java
package com.imooc.jdbc.utils;

import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class JDBCUtils {
  private static final String driverClass;
  private static final String url;
  private static final String username;
  private static final String password;

  static {
    // 加载属性文件并解析
    Properties props = new Properties();
    // 使用类的加载器的方式获取文件的输入流
    InputStream is = JDBCUtils.class.getClassLoader().getResourceAsStream("jdbc.properties");
    try {
      props.load(is);
    } catch (IOException e) {
      e.printStackTrace();
    }
    driverClass = props.getProperty("driverClass");
    url = props.getProperty("url");
    username = props.getProperty("username");
    password = props.getProperty("password");
  }

  /*
  注册驱动
   */
  public static void loadDriver() throws ClassNotFoundException {
    Class.forName(driverClass);
  }

  /*
   获得连接
   */
  public static Connection getConnection() throws Exception {
    loadDriver();
    Connection conn = DriverManager.getConnection(url, username, password);
    return conn;
  }

  /*
  资源释放
   */
  public static void release(Statement stmt, Connection conn) {
    if (stmt != null) {
      try {
        stmt.close();
      } catch (SQLException e) {
        e.printStackTrace();
      }
      stmt = null;
    }

    if (conn != null) {
      try {
        conn.close();
      } catch (SQLException e) {
        e.printStackTrace();
      }
      conn = null;
    }
  }

  public static void release(ResultSet rs, Statement stmt, Connection conn) {
    if (rs != null) {
      try {
        rs.close();
      } catch (SQLException e) {
        e.printStackTrace();
      }
      rs = null;
    }

    if (stmt != null) {
      try {
        stmt.close();
      } catch (SQLException e) {
        e.printStackTrace();
      }
      stmt = null;
    }

    if (conn != null) {
      try {
        conn.close();
      } catch (SQLException e) {
        e.printStackTrace();
      }
      conn = null;
    }
  }
}

```

## SQL注入漏洞
### 演示
```java
package com.imooc.jdbc.demo1;

import com.imooc.jdbc.utils.JDBCUtils;
import org.junit.Test;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

public class JDBCDemo4 {
  /**
   * 测试SQL注入漏洞
   * 原因在于用户可以输入SQL关键字同时系统内采用了字符串拼接的方式
   */
  @Test
  public void demo1(){
    // SELECT * FROM user WHERE username='aaa' OR '1=1' AND password='adc'
    // '1=1' AND password='adc' False
    // username='aaa' True
    // True OR False  为  True
    // 只要知道用户名就可以不用密码访问
    boolean flag = JDBCDemo4.login("aaa' OR '1=1", "adc");
    // SELECT * FROM user WHERE username='aaa' -- ' AND password='adc'
    // -- ' AND password='adc' 被注释了
//    boolean flag = JDBCDemo4.login("aaa' -- ", "adc");
    if(flag == true){
      System.out.println("登录成功!");
    } else {
      System.out.println("登录失败!");
    }
  }

  public static boolean login(String username, String password) {
    Connection conn = null;
    Statement stmt = null;
    ResultSet rs = null;
    boolean flag = false;

    try {
      conn = JDBCUtils.getConnection();
      stmt = conn.createStatement();
      String sql = "SELECT * FROM user WHERE username='" + username +"' AND password='"+ password +"'";
      rs = stmt.executeQuery(sql);
      // 判断结果集是否有数据
      if (rs.next()) {
        flag = true;
      }

    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      JDBCUtils.release(rs, stmt, conn);
    }
    return flag;


  }
}

```

### 解决方案
使用PreparedStatement解决SQL注入漏洞

PreparedStatement是Statement的子接口，它的实例对象可以通过调用Connection.preparedStatement(sql)方法获得，相对于Statement而言有以下优势：
- PreparedStatement可以避免SQL注入的问题
- PreparedStatement可对SQL进行预编译（编译一次），提高数据库的执行效率，而Statement每次都会编译SQL
- PreparedStatement对于SQL中的参数，允许使用占位符的形式进行替换，简化sql语句的编写
```java
try {
    conn = JDBCUtils.getConnection();
    String sql = "SELECT * FROM user WHERE username=? AND password=?";
    // 预处理SQL
    pstmt = conn.prepareStatement(sql);
    // 设置参数
    pstmt.setString(1, username);
    pstmt.setString(2, password);
    // 执行SQL
    rs = pstmt.executeQuery();
    // 判断结果集是否有数据
    if (rs.next()) {
        flag = true;
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    JDBCUtils.release(rs, pstmt, conn);
}
return flag;
```

## PreparedStatement
### 插入数据
```java
package com.imooc.jdbc.demo1;

import com.imooc.jdbc.utils.JDBCUtils;
import java.sql.Connection;
import java.sql.PreparedStatement;

public class JDBCDemo5 {
  
  public void demo1() {
    Connection conn = null;
    PreparedStatement pstmt = null;

    try {
      // 获得连接
      conn = JDBCUtils.getConnection();
      // 编写SQL
      String sql = "INSERT user VALUES(null, ?, ?, ?)";
      // 预处理SQL
      pstmt = conn.prepareStatement(sql);
      // 设置参数的值
      pstmt.setString(1, "qqq");
      pstmt.setString(2, "123");
      pstmt.setString(3, "张武");
      // 执行SQL
      int num = pstmt.executeUpdate();
      if (num > 0) {
        System.out.println("插入成功");
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      JDBCUtils.release(pstmt, conn);
    }
  }
  
}

```

### 更新数据
```java
try {
    // 获得连接
    conn = JDBCUtils.getConnection();
    // 编写SQL
    String sql = "UPDATE user SET username=?,password=?,name=? WHERE uid=?";
    // 预处理SQL
    pstmt = conn.prepareStatement(sql);
    // 设置参数的值
    pstmt.setString(1, "www");
    pstmt.setString(2, "123456");
    pstmt.setString(3, "张六");
    pstmt.setInt(4, 5);
    // 执行SQL
    int num = pstmt.executeUpdate();
    if (num > 0) {
        System.out.println("更新成功");
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    JDBCUtils.release(pstmt, conn);
}
```

### 删除数据
```java
try {
    // 获得连接
    conn = JDBCUtils.getConnection();
    // 编写SQL
    String sql = "DELETE FROM user WHERE uid=?";
    // 预处理SQL
    pstmt = conn.prepareStatement(sql);
    // 设置参数的值
    pstmt.setInt(1, 5);
    // 执行SQL
    int num = pstmt.executeUpdate();
    if (num > 0) {
        System.out.println("删除成功");
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    JDBCUtils.release(pstmt, conn);
}

```

### 查询数据
#### 查询多条记录
```java
try {
    // 获得连接
    conn = JDBCUtils.getConnection();
    // 编写SQL
    String sql = "SELECT * FROM user";
    // 预处理SQL
    pstmt = conn.prepareStatement(sql);
    // 设置参数的值
    // 执行SQL
    rs = pstmt.executeQuery();
    while (rs.next()) {
        System.out.println(rs.getInt("uid") + "  " + rs.getString("username") + "  " + rs.getString("password") + "  " + rs.getString("name"));
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    JDBCUtils.release(rs, pstmt, conn);
}
```
#### 查询单条记录
```java
try {
    // 获得连接
    conn = JDBCUtils.getConnection();
    // 编写SQL
    String sql = "SELECT * FROM user";
    // 预处理SQL
    pstmt = conn.prepareStatement(sql);
    // 设置参数的值
    // 执行SQL
    rs = pstmt.executeQuery();
    while (rs.next()) {
        System.out.println(rs.getInt("uid") + "  " + rs.getString("username") + "  " + rs.getString("password") + "  " + rs.getString("name"));
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    JDBCUtils.release(rs, pstmt, conn);
}
```

## 数据库连接池
连接池是创建和管理一个连接的缓冲池的技术，这些连接准备好被任何需要它们的线程使用。

应用程序直接获取连接的缺点：
- 用户每次请求都需要向数据库获得连接，而数据库创建连接需要消耗较大的资源且创建时间较长
- 如果创建连接数量过大极易造成数据库服务器内存溢出

常用连接池
- DBCP
- C3P0

### 连接池C3P0


#### 依赖包
```xml
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.2</version>
</dependency>
```

#### 手动配置
```java
package com.imooc.jdbc.demo1;

import com.imooc.jdbc.utils.JDBCUtils;
import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.junit.Test;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class DataSourceDemo1 {
  /**
   * 手动设置了连接池
   */
  @Test
  public void demo1() {

    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    try {
      // 创建连接池
      ComboPooledDataSource dataSource = new ComboPooledDataSource();
      // 设置连接池参数
      dataSource.setDriverClass("com.mysql.jdbc.Driver");
      dataSource.setJdbcUrl("jdbc:mysql://192.168.2.130:3306/jdbctest");
      dataSource.setUser("root");
      dataSource.setPassword("password");
      dataSource.setMaxPoolSize(20);
      dataSource.setInitialPoolSize(3);
      // 获得连接
      conn = dataSource.getConnection();
      // 编写SQL
      String sql = "SELECT * FROM user";
      // 预编译SQL
      pstmt = conn.prepareStatement(sql);
      // 设置参数
      // 执行SQL
      rs = pstmt.executeQuery();
      while (rs.next()) {
        System.out.println(rs.getInt("uid") + "  " + rs.getString("username") + "  " + rs.getString("password") + "  " + rs.getString("name"));
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      JDBCUtils.release(rs, pstmt, conn);
    }
  }
}


```

#### 使用配置文件
```xml


```

```java
  public void demo2() {

    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    try {
      // 重复创建连接池
      // ComboPooledDataSource dataSource = new ComboPooledDataSource();
      // conn = dataSource.getConnection();
      // 抽取工具类保证连接池只初始化一次
      conn = JDBCUtils2.getConnection();
      // 编写SQL
      String sql = "SELECT * FROM user";
      // 预编译SQL
      pstmt = conn.prepareStatement(sql);
      // 设置参数
      // 执行SQL
      rs = pstmt.executeQuery();
      while (rs.next()) {
        System.out.println(rs.getInt("uid") + "  " + rs.getString("username") + "  " + rs.getString("password") + "  " + rs.getString("name"));
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      JDBCUtils2.release(rs, pstmt, conn);
    }
  }
```

```java
// 工具类
public class JDBCUtils2 {
  // 自动去classPath中读取c3p0-config.xml
  private static final ComboPooledDataSource dataSource = new ComboPooledDataSource();

  /*
   获得连接
   */
  public static Connection getConnection() throws Exception {
    Connection conn = dataSource.getConnection();
    return conn;
  }

  ...
}

```
