## Mybatis持久层框架

### 概述

Mybatis前身是apache的一个开源项目iBatis，2010年这个项目由apache software foundation迁移到了google code并且改命为MyBatis，2013年11月迁移到Github。

MyBatis是一个实现了数据持久化的开源框架，简单理解就是对JDBC的上层封装。

#### Mybatis的优点

- 与JDBC相比，减少了50%以上的代码量。
- MyBatis是最简单的持久化框架，小巧并且简单易学。
- MyBatis相当灵活，不会对应用程序或者数据库的现有设计强加任何影响，SQL写在XML里，从程序代码中彻底分离，降低耦合度，便于统一管理和优化，并可重用。
- 提供XML标签，支持编写动态SQL语句。
- 提供映射标签，支持对象与数据库的ORM字段关系映射。

#### Mybatis的缺点

- SQL语句的编写工作量较大，尤其是字段多、关联表多时更是如此，对开发人员编写SQL语句的功底有一定要求。
- SQL语句依赖数据库，导致数据库移植性差，不能随意更换数据库。

#### Mybatis的核心接口和类

```flow
builder=>operation: SqlSessionFactoryBuilder
factory=>operation: SqlSessionFactory
session=>operation: SqlSession
builder->factory->session
```

### 依赖配置

#### pom.xml

```xml
  <dependencies>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.5</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.11</version>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.6</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
```

#### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <!-- 配置Mybatis运行环境 -->
  <environments default="development">
    <environment id="development">
      <!-- 配置JDBC事务管理 -->
      <transactionManager type="JDBC"/>
      <!-- 配置数据库连接池 -->
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;serverTimezone=Asia/Shanghai&amp;characterEncoding=UTF8"/>
        <property name="username" value="root"/>
        <property name="password" value="password"/>
      </dataSource>
    </environment>
  </environments>
</configuration>
```

### 调用方式

> 数据准备

```sql
create table account
(
    id int primary key auto_increment,
    username varchar(11) not null,
    password varchar(11) not null,
    age int
);

insert into account(username, password, age) values('张三', '123123', 22),('李四', '456456', 24);
```

#### 原生接口

MyBatis 框架需要开发者⾃定义 SQL 语句，写在 Mapper.xml ⽂件中，实际开发中，会为每个实体类创建对应的 Mapper.xml ，定义管理该对象数据的 SQL。

1. 定义实体类型

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Account {
     private Long id;
     private String username;
     private String password;
     private int age;
   }
   ```

2. 编写mapper配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="mapper.AccountMapper">
     <select id="findAll" resultType="com.peter.model.Account">
       select * from account;
     </select>
     <select id="findById" parameterType="long" resultType="com.peter.model.Account">
       select * from account where id = #{id};
     </select>
   </mapper>
   ```

   > 1. namespace: 命名空间，namespace+id组成唯一地址供mybatis查询，使用原生接口时这里不限制命名方式，此处命名为mapper.xml文件路径。
   > 2. 标签：select,update,insert,delete。
   > 3. id: 参数，单参数时不区分名称。
   > 4. parameterType: 参数类型。

3. 将mapper配置文件注册进mybatis-config.xml

   ```xml
     <mappers>
       <mapper resource="mapper/accountMapper.xml"></mapper>
     </mappers>
   ```

4. 使用原生接口进行调用

   ```java
   public class Test1 {
     public static void main(String[] args) throws IOException {
       InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
       SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
       SqlSessionFactory factory = builder.build(inputStream);
       SqlSession session = factory.openSession();
       // 查询所有数据，statement: namespace+id 组成唯一地址查询相应SQL语句
       String statement = "mapper.accountMapper.findAll";
       List<Account> list = session.selectList(statement);
       System.out.println(list);
       // 查询单条数据
       Account account = session.selectOne("mapper.accountMapper.findById", 1L);
       System.out.println(account);
       session.close();
     }
   }
   ```

#### Mapper代理

相比原生接口通过“namespace+id"查找SQL语句并执行的方式，Mapper提供了类的方式定位SQL和传参。

- 开发者需要为每一个mapper.xml定义一个Mapper接口。
- Mybatis通过接口的全类名定位"namespace", 通过方法名定位"id",通过方法入参传递参数。
- Mybatis将为每个接口实现一个类并生成代理对象，对象内部的逻辑仍然是原生接口调用的那一套。

> Mybatis定义了一套规范化的约束要求开发者按照约定定义一些文件，然后Mybatis根据约定来进行内部实现。

1. 定义实体类型，同原生接口调用

2. 定义Mapper接口

   ```java
   public interface AccountMapper {
     List<Account> findAll();
     Account findById(long id);
   }
   ```

3. 定义Mapper.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.peter.mapper.AccountMapper">
     <select id="findAll" resultType="com.peter.model.Account">
       select * from account;
     </select>
     <select id="findById" parameterType="long" resultType="com.peter.model.Account">
       select * from account where id = #{id};
     </select>
   </mapper>
   ```

   > mapper配置文件的定义基本上没有区别，但要注意：
   >
   > 1. namespace必须为Mapper接口的全类名
   > 2. id必须为Mapper接口中某一个方法的名字

4. 将Mapper.xml注册到mybatis-config.xml,同原生接口调用

5. 使用Mapper代理对象进行调用

   ```java
   public class Test2 {
     public static void main(String[] args) throws IOException {
       InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
       SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
       SqlSessionFactory factory = builder.build(inputStream);
       SqlSession session = factory.openSession();
       // 获得Mapper代理对象
       AccountMapper accountMapper = session.getMapper(AccountMapper.class);
       List<Account> list = accountMapper.findAll();
       System.out.println(list);
       Account account = accountMapper.findById(1L);
       System.out.println(account);
       session.close();
     }
   }
   ```

   

### Mapper.xml详解

#### 常用标签

1. select: 查询

   ```xml
     <select id="findById" parameterType="java.lang.Long" resultType="com.peter.model.Account">
       select * from account where id = #{id};
     </select>
   ```

2. update: 修改

   ```xml
     <update id="updateAccount" parameterType="com.peter.model.Account">
       update account set username = #{username} where id = #{id};
     </update>
   ```

3. insert: 新增

   ```xml
     <insert id="insertAccount" parameterType="com.peter.model.Account">
       insert into account(username, password, age) value (#{username}, #{password}, #{age});
     </insert>
   ```

4. delete: 删除

   ```xml
     <delete id="deleteAccount" parameterType="long">
       delete from account where id = #{id};
     </delete>
   ```

#### parameterType

参数类型

1. 基本数据类型

   ```xml
     <select id="findById" parameterType="long" resultType="com.peter.model.Account">
       select * from account where id = #{id};
     </select>
   ```

2. String类型

   ```xml
     <select id="findByUsername" parameterType="java.lang.String" resultType="com.peter.model.Account">
       select * from account where username = #{username}
     </select>
   ```

3. 包装类型

   ```xml
     <select id="findById" parameterType="java.lang.Long" resultType="com.peter.model.Account">
       select * from account where id = #{id};
     </select>
   ```

4. 多个参数: 不需要填写parameterType

   ```xml
     <select id="findByUsernameAndAge" resultType="com.peter.model.Account">
       select * from account where username = #{arg0} and age = #{arg1}
     </select>
   ```

5. javaBean

   ```xml
     <select id="findByAccount" parameterType="com.peter.model.Account" resultType="com.peter.model.Account">
       select * from account where username = #{username} and age = #{age};
     </select>
   ```

   

#### resultType

结果类型

1. 基本数据类型

   ```xml
     <select id="findCount" resultType="int">
       select count(id) from account;
     </select>
   ```

2. String类型

   ```xml
     <select id="findUsernameById" parameterType="long" resultType="java.lang.String">
       select username from account where id = #{id};
     </select>
   ```

3. 包装类型

   ```xml
     <select id="findAgeById" parameterType="long" resultType="java.lang.Integer">
       select age from account where id = #{id};
     </select>
   ```

4. JavaBean

   ```xml
     <select id="findById" parameterType="long" resultType="com.peter.model.Account">
       select * from account where id = #{id};
     </select>
   ```

### 级联查询

#### 一对多

1. 数据准备

   ```sql
   create table student
   (
       id int primary key auto_increment,
       name varchar(11) not null,
       cid int
   );
   
   create table classes
   (
       id int primary key auto_increment,
       name varchar(11) not null
   );
   
   insert into classes(name) values ('一班'), ('二班');
   insert into student(name, cid) values ('小明', 1), ('小红', 1), ('小张', 2);
   ```

2. 定义实体类

   ```java
   @Data
   public class Student {
     private Long id;
     private String name;
     private Classes classes;
   }
   
   @Data
   public class Classes {
     private Long id;
     private String name;
     private List<Student> students;
   }
   ```

3. 定义Mapper接口

   ```java
   public interface StudentMapper {
     Student findById(long id);
   }
   
   public interface ClassesMapper {
     List<Student> findById(long id);
   }
   ```

   

4. 定义Mapper.xml

   - studentMapper.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="com.peter.mapper.StudentMapper">
       <!-- 对于嵌套关系的查询需要通过resultMap来实现间接映射-->
       <resultMap id="studentMap" type="com.peter.model.Student">
         <id column="sid" property="id"></id>
         <result column="sname" property="name"></result>
         <association property="classes" javaType="com.peter.model.Classes">
           <id column="cid" property="id"></id>
           <result column="cname" property="name"></result>
         </association>
       </resultMap>
       <select id="findById" parameterType="long" resultMap="studentMap">
        select s.id sid, s.name sname, c.id cid, c.name cname from student s,classes c where s.id = #{id} and s.cid = c.id;
       </select>
     </mapper>
     ```

   - classesMapper.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="com.peter.mapper.ClassesMapper">
       <resultMap id="classesMap" type="com.peter.model.Classes">
         <id column="cid" property="id"></id>
         <result column="cname" property="name"></result>
         <!-- 这里使用collection和ofType -->
         <collection property="students" ofType="com.peter.model.Student">
           <id column="sid" property="id"></id>
           <result column="sname" property="name"></result>
         </collection>
       </resultMap>
       <select id="findById" parameterType="long" resultMap="classesMap">
            select s.id sid, s.name sname, c.id cid, c.name cname from student s,classes c where c.id = #{id} and s.cid = c.id;
       </select>
     </mapper>
     ```

     

5. 添加Mapper.xml到mybatis-config.xml

   ```xml
     <mappers>
       <mapper resource="mapper/accountMapper.xml"></mapper>
       <mapper resource="mapper/studentMapper.xml"></mapper>
       <mapper resource="mapper/classesMapper.xml"></mapper>
     </mappers>
   ```

6. 开始查询

   ```java
   public class Test3 {
     public static void main(String[] args) throws IOException {
       InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
       SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
       SqlSessionFactory factory = builder.build(inputStream);
       SqlSession session = factory.openSession();
       StudentMapper studentMapper = session.getMapper(StudentMapper.class);
       System.out.println(studentMapper.findById(1));
       ClassesMapper classesMapper = session.getMapper(ClassesMapper.class);
       System.out.println(classesMapper.findById(1));
       session.close();
     }
   }
   ```

   

#### 多对多

多对多的查询行为和一对多类似，只不过两边都存在一个List。

1. 数据准备

   ```sql
   create table customer
   (
       id int primary key auto_increment,
       name varchar(11) not null
   );
   
   create table goods
   (
       id int primary key auto_increment,
       name varchar(11) not null
   );
   
   create table customer_goods
   (
       id int primary key auto_increment,
       cid int not null,
       gid int not null
   );
   
   insert into customer(name) values ('小明'), ('小张');
   insert into goods(name) values ('电视机'), ('电冰箱'), ('电磁炉');
   insert into customer_goods(cid, gid) values (1, 1), (1, 2), (2, 2), (2, 3);
   ```

2. 定义实体类

   ```java
   @Data
   public class Customer {
     private Long id;
     private String name;
     private List<Goods> goods;
   }
   
   @Data
   public class Goods {
     private long id;
     private String name;
     private List<Customer> customers;
   }
   ```

3. 定义Mapper接口

   ```java
   public interface CustomerMapper {
     Customer findById(long id);
   }
   
   public interface GoodsMapper {
     Goods findById(long id);
   }
   ```

4. 定义Mapper.xml

   - customerMapper.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="com.peter.mapper.CustomerMapper">
       <resultMap id="customerMap" type="com.peter.model.Customer">
         <id column="cid" property="id"></id>
         <result column="cname" property="name"></result>
         <!-- 使用collection -->
         <collection property="goods" ofType="com.peter.model.Goods">
           <id column="gid" property="id"></id>
           <result column="gname" property="name"></result>
         </collection>
       </resultMap>
       <select id="findById" parameterType="long" resultMap="customerMap">
         select c.id cid, c.name cname, g.id gid, g.name gname from customer c, goods g, customer_goods cg where g.id = #{id} and c.id = cg.cid and g.id = cg.gid;
       </select>
     </mapper>
     ```

   - goodsMapper.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="com.peter.mapper.GoodsMapper">
       <resultMap id="goodsMap" type="com.peter.model.Goods">
         <id column="gid" property="id"></id>
         <result column="gname" property="name"></result>
         <!-- 使用collection -->
         <collection property="customers" ofType="com.peter.model.Customer">
           <id column="cid" property="id"></id>
           <result column="cname" property="name"></result>
         </collection>
       </resultMap>
       <select id="findById" parameterType="long" resultMap="goodsMap">
         select c.id cid, c.name cname, g.id gid, g.name gname from customer c, goods g, customer_goods cg where g.id = #{id} and c.id = cg.cid and g.id = cg.gid;
       </select>
     </mapper>
     ```

     

5. 添加Mapper.xml到mybatis-config.xml

   ```xml
     <mappers>
       <mapper resource="mapper/accountMapper.xml"></mapper>
       <mapper resource="mapper/studentMapper.xml"></mapper>
       <mapper resource="mapper/classesMapper.xml"></mapper>
       <mapper resource="mapper/customerMapper.xml"></mapper>
       <mapper resource="mapper/goodsMapper.xml"></mapper>
     </mappers>
   ```

6. 开始查询

   ```java
   public class Test4 {
     public static void main(String[] args) throws IOException {
       InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
       SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
       SqlSessionFactory factory = builder.build(inputStream);
       SqlSession session = factory.openSession();
       CustomerMapper customerMapper = session.getMapper(CustomerMapper.class);
       System.out.println(customerMapper.findById(1));
       GoodsMapper goodsMapper = session.getMapper(GoodsMapper.class);
       System.out.println(goodsMapper.findById(2));
     }
   }
   ```



### 懒加载

如果我们将级联查询拆分为两次查询，比如先查询student数据，然后根据cid再查询classes信息，在这种情况下懒加载可以根据我们查询的字段自动决定是否要查询另一张表。如我们只需要查询student姓名，则mybatis不会去查询classes表。

这种查询相比级联查询性能要更好一点，但要求必须存在两个Mapper

1. 定义Mapper接口

   ```java
   public interface StudentLazyMapper {
     Student findById(long id);
     Student findLazyById(long id);
   }
   ```

2. 定义Mapper.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.peter.mapper.StudentLazyMapper">
     <resultMap id="studentLazyMap" type="com.peter.model.Student">
       <id column="id" property="id"></id>
       <result column="name" property="name"></result>
       <!-- 如果存在cid字段则继续查询classes表-->
       <association property="classes" javaType="com.peter.model.Classes" select="com.peter.mapper.ClassesMapper.findById" column="cid"></association>
     </resultMap>
   
     <select id="findById" parameterType="long" resultMap="studentLazyMap">
       select * from student where id = #{id};
     </select>
     <select id="findLazyById" parameterType="long" resultMap="studentLazyMap">
       select id, name from student where id = #{id};
     </select>
   
   </mapper>
   ```

   

3. 添加Mapper.xml到mybatis-config.xml同时开启懒加载

   ```xml
   <configuration>
   	<settings>
       <!-- 打开日志-->
       <setting name="logImpl" value="STDOUT_LOGGING"/>
       <!-- 开启懒加载 -->
       <setting name="lazyLoadingEnabled" value="true"/>
     </settings>  
   </configuration>
   
   <mappers>
       <mapper resource="mapper/accountMapper.xml"></mapper>
       <mapper resource="mapper/studentMapper.xml"></mapper>
       <mapper resource="mapper/classesMapper.xml"></mapper>
       <mapper resource="mapper/customerMapper.xml"></mapper>
       <mapper resource="mapper/goodsMapper.xml"></mapper>
       <mapper resource="mapper/studentLazyMapper.xml"></mapper>
     </mappers>
   ```

   

4. 开始查询

   ```java
   public class Test5 {
     public static void main(String[] args) throws IOException {
       InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
       SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
       SqlSessionFactory factory = builder.build(inputStream);
       SqlSession session = factory.openSession();
       StudentLazyMapper studentLazyMapper = session.getMapper(StudentLazyMapper.class);
       System.out.println(studentLazyMapper.findById(1));
       System.out.println(studentLazyMapper.findLazyById(1));
       session.close();
     }
   }
   ```



### 缓存

#### 什么是 MyBatis 缓存

使⽤缓存可以减少 Java 应⽤与数据库的交互次数，从⽽提升程序的运⾏效率。⽐如查询出 id = 1 的对

象，第⼀次查询出之后会⾃动将该对象保存到缓存中，当下⼀次查询时，直接从缓存中取出对象即可，

⽆需再次访问数据库。

#### MyBatis 缓存分类

1、⼀级缓存：SqlSession 级别，默认开启，并且不能关闭。

操作数据库时需要创建 SqlSession 对象，在对象中有⼀个 HashMap ⽤于存储缓存数据，不同的

SqlSession 之间缓存数据区域是互不影响的。

⼀级缓存的作⽤域是 SqlSession 范围的，当在同⼀个 SqlSession 中执⾏两次相同的 SQL 语句事，第⼀

次执⾏完毕会将结果保存到缓存中，第⼆次查询时直接从缓存中获取。

需要注意的是，如果 SqlSession 执⾏了 DML 操作（insert、update、delete），MyBatis 必须将缓存

清空以保证数据的准确性。

2、⼆级缓存：Mapper 级别，默认关闭，可以开启。

使⽤⼆级缓存时，多个 SqlSession 使⽤同⼀个 Mapper 的 SQL 语句操作数据库，得到的数据会存在⼆

级缓存区，同样是使⽤ HashMap 进⾏数据存储，相⽐较于⼀级缓存，⼆级缓存的范围更⼤，多个

SqlSession 可以共⽤⼆级缓存，⼆级缓存是跨 SqlSession 的。

⼆级缓存是多个 SqlSession 共享的，其作⽤域是 Mapper 的同⼀个 namespace，不同的 SqlSession

两次执⾏相同的 namespace 下的 SQL 语句，参数也相等，则第⼀次执⾏成功之后会将数据保存到⼆级

缓存中，第⼆次可直接从⼆级缓存中取出数据。

#### 一级缓存

```
    System.out.println(accountMapper.findAll());
    // 第二次将从缓存中读取数据
    System.out.println(accountMapper.findAll());
    session.close();
```

#### 二级缓存

1. mybatis-config.xml开启二级缓存

   ```xml
     <settings>
       <!-- 打开日志-->
       <setting name="logImpl" value="STDOUT_LOGGING"/>
       <!-- 开启懒加载 -->
       <setting name="lazyLoadingEnabled" value="true"/>
       <!-- 开启二级缓存 -->
       <setting name="cacheEnabled" value="true"/>
     </settings>
   ```

2. Mapper.xml添加缓存标签

   ```xml
    <mapper namespace="xxx"> 
     <cache></cache>
    </mapper>
   ```

3. 实体类型实现Serializable

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Account implements Serializable {
     private Long id;
     private String username;
     private String password;
     private int age;
   }
   ```

4. 执行查询

   ```java
   SqlSession session = factory.openSession();
   AccountMapper accountMapper = session.getMapper(AccountMapper.class);
   System.out.println(accountMapper.findAll());
   session.close();
   // 从二级缓存中获取数据
   SqlSession session1 = factory.openSession();
   AccountMapper accountMapper1 = session1.getMapper(AccountMapper.class);
   System.out.println(accountMapper1.findAll());
   session1.close();
   ```

   

### 动态SQL

使⽤动态 SQL 可简化代码的开发，减少开发者的⼯作量，程序可以⾃动根据业务参数来决定 SQL 的组

成。

#### if标签

```xml
  <select id="findByAccount" resultType="com.peter.model.Account">
    select * from account where
    <if test="id != null">
      id = #{id}
    </if>
    <if test="username != null">
      and username = #{username}
    </if>
    <if test="age != 0">
      and age = #{age}
    </if>
  </select>
```

#### where标签

在if标签中如果id=null，则语句会变成select * from account **where and** username = xxx"从而报错，where可以用来处理这种情况，自动去掉and 。

```xml
  <select id="findByAccount" resultType="com.peter.model.Account">
    select * from account
    <where>
      <if test="id != null">
        id = #{id}
      </if>
      <if test="username != null">
        and username = #{username}
      </if>
      <if test="age != 0">
        and age = #{age}
      </if>
    </where>
  </select>
```

#### choose和when标签

choose类似于switch语句，只要一个满足条件就不再继续向下判断

```xml
  <select id="findByAccount" resultType="com.peter.model.Account">
    select * from account
    <where>
      <choose>
        <when test="id != null">
          id = #{id}
        </when>
        <when test="username != null">
          and username = #{username}
        </when>
        <when test="age != 0">
          and age = #{age}
        </when>
      </choose>
    </where>
  </select>
```

#### trim标签

trim标签用于在语句前后增加和删除某些字符，比如我们使用where标签解决and问题，trim也可以解决。

常用属性为prefix,prefixOverrides,suffix,suffixOverrides

```xml
  <select id="findByAccount" resultType="com.peter.model.Account">
    select * from account
    <!-- 给下面SQL语句之前添加where，如果SQL语句之前出现了and则删除and -->
    <trim prefix="where" prefixOverrides="and">
      <choose>
        <when test="id != null">
          id = #{id}
        </when>
        <when test="username != null">
          and username = #{username}
        </when>
        <when test="age != 0">
          and age = #{age}
        </when>
      </choose>
    </trim>
  </select>
```

#### set标签

set标签用于update语句，其作用和where类似，set标签是为了解决SQL语句结尾出现逗号的情况。

```xml
  <update id="updateAccount" parameterType="com.peter.model.Account">
    update account
    <set>
      <if test="username != null">
        username = #{username},
      </if>
      <if test="age != 0">
        age = #{age}
      </if>
    </set>
  </update>
```

#### foreach标签

```xml
  <select id="findByIds" parameterType="long" resultType="com.peter.model.Account">
    select * from account
    <where>
      <!-- open左边开始字符串，close右边闭合字符串，separator字段分隔符 -->
      <!-- 整个一套下来就是为了拼接出 where id in (1,2) 这样的SQL语句 -->
      <foreach collection="ids" open="id in (" close=")" item="id" separator=",">
        #{id}
      </foreach>
    </where>
  </select>
```



### 总结

Mybatis作为一个半自动ORM框架，相比于JDBC而言能够减少代码量，分离出SQL到配置文件统一管理，提供ORM映射等，但使用过程中仍然能够感受到Mybatis和SQL语句之间的紧密联系，所以使用起来需要有较好的SQL功底，同时由于其对于一些简单的CRUD操作仍然需要编写SQL语句，使用起来还是不免有些繁琐。



