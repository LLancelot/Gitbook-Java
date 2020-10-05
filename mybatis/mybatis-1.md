# MyBatis-1



ORM：Object Relationship Mapping 对象关系映射

面向对象的编程

```text
public class Orders{
    private Integer id;
    private Double cost;
}
​
public class User{
    private Integer id;
    private String name;
    private List<Orders> orders;
}
​
Orders orders = new Orders(1,6000);
Orders orders2 = new Orders(1,3000);
​
User user = new User(1,"小红");
user.setOrders(Arrays.asList(orders,orders2));
```

关系型数据库，通过主外键约束完成表之间的关联

Java 完成业务逻辑，数据需要持久化到 DB

Java 到数据库的数据存取需要进行转换，将面向对象方式转成面向关系的方式，这种转换就是映射，这就是 ORM。

MyBatis（MyBatis Plus）、Hibernate、Spring Data JPA 是 Java 领域 ORM 方式的主流实现框架。

Hiernate 和 Spring Data JPA （底层就是 Hibernate 实现）是全自动化的 ORM 框架，不需要做任何的操作，就可以获取到结果。

MyBatis 是半自动化的 ORM 框架。

全自动就意味着扩展性，个性化，灵活性较差，半自动化就意味灵活性更好，互联网项目需求迭代较快，MyBatis 很适合互联网项目。

#### JDBC 开发

1、加载驱动

2、配置各种参数 url、Driver class、root、password

3、获取 Connection

4、定义 SQL

5、使用 Statement 进行查询

6、ResultSet 需要手动解析

7、释放资源

#### MyBatis 使用

1、创建 Maven 工程

```text
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.5</version>
    </dependency>
​
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.21</version>
    </dependency>
    
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.12</version>
    </dependency>
</dependencies>
```

lombok 插件，可以帮助我们生成实体类中的 getter、setter、toString、构造...

IDEA 中使用 Lombok 需要安装插件

2、创建实体类

```text
package com.southwind.entity;
​
import lombok.Data;
​
@Data
public class Account {
    private Integer id;
    private String username;
    private String password;
    private Integer age;
}
```

3、创建配置文件，名字可以自定义。

```text
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 配置数据源 -->
    <environments default="development">
        <environment id="development">
            <!-- 事务管理 -->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 数据源 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybd2"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

4、开发，Mapper 代理实现自定义接口

使用 MyBatis 框架，开发者只需要定义接口即可，不需要自己实现，由 MyBatis 通过 Mapper 代理机制自动帮助我们创建接口的实现类，并且创建接口的实例化对象。

```text
package com.southwind.mapper;
​
import com.southwind.entity.Account;
​
import java.util.List;
​
public interface AccountMapper {
    public List<Account> findAll();
    public Account findById(Integer id);
    public void save(Account account);
    public void update(Account account);
    public void deleteById(Integer id);
}
```

```text
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.southwind.mapper.AccountMapper">
    <!-- statement -->
    <select id="findAll" resultType="com.southwind.entity.Account">
        select * from account
    </select>
​
    <select id="findById" parameterType="java.lang.Integer" resultType="com.southwind.entity.Account">
        select * from account where id = #{id}
    </select>
​
    <insert id="save" parameterType="com.southwind.entity.Account">
        insert into account(username,password,age) values(#{username},#{password},#{age})
    </insert>
​
    <update id="update" parameterType="com.southwind.entity.Account">
        update account set username = #{username},password = #{password},age = #{age} where id = #{id}
    </update>
​
    <delete id="deleteById" parameterType="java.lang.Integer">
        delete from account where id = #{id}
    </delete>
</mapper>
```

5、在 config.xml 中注册 Mapper.xml

```text
<mappers>
    <mapper resource="com/southwind/mapper/AccountMapper.xml"></mapper>
</mappers>
```

6、使用

```text
package com.southwind.test;
​
import com.southwind.entity.Account;
import com.southwind.mapper.AccountMapper;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
​
import java.io.InputStream;
​
public class Test {
    public static void main(String[] args) {
        //加载 MyBatis 配置文件
        InputStream inputStream = Test.class.getClassLoader().getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(inputStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //操作API
        AccountMapper mapper = sqlSession.getMapper(AccountMapper.class);
        mapper.findAll().forEach(System.out::println);
    }
}
```

#### 常见错误

* Mapper.xml 找不到，pom.xml 配置让 Maven 可以读取 java 路径下 XML 文件。

```text
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```

* Mapper.xml 中的 namespace 必须和接口的全类名保持一致。
* Mapper.xml 中的 statement 的 id 值必须和接口方法一致。
* Mapper.xml 必须在 mybatis-config.xml 中进行注册。

#### 打印 SQL

```text
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING" />
</settings>
```

