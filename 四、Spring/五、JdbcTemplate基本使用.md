# 五、JdbcTemplate基本使用

## 1、Spring配置数据源


### 1.1 数据源（连接池）的作用


数据源(连接池)是提高程序性能出现的



事先实例化数据源，初始化部分连接资源



使用连接资源时从数据源中获取



使用完毕后将连接资源归还给数据源



常见的数据源(连接池)：DBCP、C3P0、BoneCP、Druid等



**开发步骤**



①导入数据源的坐标和数据库驱动坐标



②创建数据源对象



③设置数据源的基本连接数据



④使用数据源获取连接资源和归还连接资源



### 1.2 数据源的手动创建


#### 导入c3p0/druid，mysql数据库驱动坐标


```xml
<!-- C3P0连接池 -->
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
<!-- Druid连接池 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
<!-- mysql驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.39</version>
</dependency>
```



#### 创建连接池


+ C3P0连接池



```java
@Test
public void testC3P0() throws Exception {
    //创建数据源
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    //设置数据库连接参数
    dataSource.setDriverClass("com.mysql.jdbc.Driver");
    dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
    dataSource.setUser("root");
    dataSource.setPassword("root");
    //获得连接对象
    Connection connection = dataSource.getConnection();
    System.out.println(connection);
    connection.close();
}
```



+ Druid连接池



```java
@Test
public void testDruid() throws Exception {
    //创建数据源
    DruidDataSource dataSource = new DruidDataSource();
    //设置数据库连接参数
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/test");
    dataSource.setUsername("root");
    dataSource.setPassword("root");
    //获得连接对象
    Connection connection = dataSource.getConnection();
    System.out.println(connection);
    connection.close();
}
```



#### 读取jdbc.properties配置文件创建连接池


+ 提取jdbc.properties配置文件



```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test
jdbc.username=root
jdbc.password=root
```



+ 读取jdbc.properties配置文件创建连接池



```java
@Test
public void testC3P0ByProperties() throws Exception {
    //加载类路径下的jdbc.properties
    ResourceBundle rb = ResourceBundle.getBundle("jdbc");

    String driver = rb.getString("jdbc.driver");
    String url = rb.getString("jdbc.url");
    String username = rb.getString("jdbc.username");
    String password = rb.getString("jdbc.password");

    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setDriverClass(driver);
    dataSource.setJdbcUrl(url);
    dataSource.setUser(username);
    dataSource.setPassword(password);

    Connection connection = dataSource.getConnection();
    System.out.println(connection);
    connection.close();
}
```



### 1.3 Spring配置数据源


可以将DataSource的创建权交由Spring容器去完成



DataSource有无参构造方法，而Spring默认就是通过无参构造方法实例化对象的



DataSource要想使用需要通过set方法设置数据库连接信息，而Spring可以通过set方法进行字符串注入



```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/book"></property>
    <property name="user" value="root"></property>
    <property name="password" value="root"></property>
</bean>
```



测试从容器当中获取数据源



```java
//测试spring容器产生数据源对象
@Test
public void test4() throws Exception {

    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

    DataSource dataSource = context.getBean("dataSource", DataSource.class);
    Connection connection = dataSource.getConnection();
    
    System.out.println(connection);
    connection.close();

}
```



#### 抽取jdbc配置文件


首先，需要引入context命名空间和约束路径：



```xml
命名空间：xmlns:context="http://www.springframework.org/schema/context"

约束路径：http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd
```



applicationContext.xml加载jdbc.properties配置文件获得连接信息。



```xml
<!--    加载外部的properties文件-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"></property>
    <property name="jdbcUrl" value="${jdbc.url}"></property>
    <property name="user" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
</bean>
```



## 2、JdbcTempater概述


JdbcTemplate是spring框架中提供的一个对象，是<font style="color:#E8323C;">对原始繁琐的Jdbc API对象的简单封装。</font>



spring框架为我们提供了很多的操作模板类。



## 3、快速入门


①导入spring-jdbc和spring-tx坐标



②创建数据库表和实体



③创建JdbcTemplate对象



④执行数据库操作



### 导入spring-jdbc和spring-tx坐标


```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
```



### 创建数据库表和实体


```java
public class Account {

    private String name;
    
    private double money;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getMoney() {
        return money;
    }

    public void setMoney(double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```



### 创建JdbcTemplate对象


```java
//测试JdbcTemplate开发步骤
@Test
public void test1() throws PropertyVetoException {

    //创建数据源对象
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setDriverClass("com.mysql.jdbc.Driver");
    dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/book");
    dataSource.setUser("root");
    dataSource.setPassword("root");

    JdbcTemplate jdbcTemplate = new JdbcTemplate();
    //设置数据源对象  知道数据库在哪
    jdbcTemplate.setDataSource(dataSource);

    //执行数据库操作
    int row = jdbcTemplate.update("insert into t_account values(?,?,?) ", null, "Tom", 5000);
    System.out.println(row);
}
```



## 4、spring配置JdbcTemplate


我们可以将JdbcTemplate的创建权交给Spring，



将数据源DataSource的创建权也交给Spring，



在Spring容器内部将数据源DataSource注入到JdbcTemplate模版对象中,



然后通过Spring容器获得JdbcTemplate对象来执行操作。



### 代码实现


配置如下：



```xml
<!--    配置数据库连接池-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/book"></property>
    <property name="user" value="root"></property>
    <property name="password" value="root"></property>
</bean>

<!--    JdbcTemplate对象-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <!--        注入dataSource-->
    <property name="dataSource" ref="dataSource"></property>
</bean>
```



测试代码



```java
//测试spring产生JdbcTemplate对象
@Test
public void test2() {

    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    JdbcTemplate jdbcTemplate = context.getBean("jdbcTemplate", JdbcTemplate.class);

    //执行操作
    int row = jdbcTemplate.update("insert into t_account values(?,?,?) ", null, "Lucy", 5000);
    System.out.println(row);
}
```



### 抽取jdbc.properties


将数据库的连接信息抽取到外部配置文件中，和spring的配置文件分离开，有利于后期维护



```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/book
jdbc.username=root
jdbc.password=root
```



配置文件修改为:



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
                           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--加载jdbc.properties-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--    数据源对象-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"></property>
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="user" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
    </bean>

    <!--    jdbc模板对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

</beans>
```



## 5、JdbcTemplate 操作数据库


### 更新操作


添加、修改、删除



```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class JdbcTemplateCURDTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    public void testUpdate(){
        int row = jdbcTemplate.update("update t_account set money=? where id = ?", 10000, 1);
        System.out.println(row);
    }

    @Test
    public void testDelete(){
        int row = jdbcTemplate.update("delete from t_account where id = ?", 2);
        System.out.println(row);
    }
}
```



### 查询操作


```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:ApplicationContext.xml")
public class JdbcTemplateCURDTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    public void testQueryAll() {
        List<Account> accountList = jdbcTemplate.query("select * from t_account", new BeanPropertyRowMapper<Account>(Account.class));

        System.out.println(accountList);
    }

    @Test
    public void testQueryOne() {
        Account account = jdbcTemplate.queryForObject("select * from t_account where id = ?", new BeanPropertyRowMapper<Account>(Account.class), 1);
        System.out.println(account);
    }

    @Test
    public void testQueryCount() {
        Long count = jdbcTemplate.queryForObject("select count(*) from t_account ", Long.class);
        System.out.println(count);
    }

}
```





> 更新: 2023-06-13 14:37:22  
> 原文: <https://www.yuque.com/like321/kwpbuz/zz38gp>