# 三、Spring整合Junit

## 1、原始Junit测试Spring的问题


在测试类中，每个测试方法都有以下两行代码：



```java
ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");

AccountService as = context.getBean("accountService",AccountService.class);
```



这两行代码的作用是获取容器，如果不写的话，直接会提示空指针异常。所以又不能轻易删掉。



## 2、上述问题解决思路


1. 让SpringJunit负责创建Spring容器，但是需要将配置文件的名称告诉它



2. 将需要进行测试Bean直接在测试类中进行注入



## 3、Spring集成Junit4


Spring集成Junit步骤



1.  导入spring集成Junit的坐标 
2.  使用@Runwith注解替换原来的运行期 
3.  使用@ContextConfiguration指定配置文件或配置类 
4.  使用@Autowired注入需要测试的对象 
5.  创建测试方法进行测试 



### 导入spring集成Junit的坐标


```xml
<!--此处需要注意的是，spring5 及以上版本要求 junit 的版本必须是 4.12 及以上-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```



### 使用@Runwith注解替换原来的运行期


```java
@RunWith(SpringJUnit4ClassRunner.class) //单元测试框架
public class JTest4 {

}
```



### 使用@ContextConfiguration指定配置文件或配置类


```java
@RunWith(SpringJUnit4ClassRunner.class) //单元测试框架
//加载spring核心配置文件
//@ContextConfiguration(value = {"classpath:applicationContext.xml"})
//加载spring核心配置类
@ContextConfiguration(classes = {SpringConfiguration.class})
public class JTest4 {

}
```



### 使用@Autowired注入需要测试的对象


```java
@RunWith(SpringJUnit4ClassRunner.class) //单元测试框架
//加载spring核心配置文件
//@ContextConfiguration(value = {"classpath:applicationContext.xml"})
//加载spring核心配置类
@ContextConfiguration(classes = {SpringConfiguration.class})
public class JTest4 {

    @Autowired	//注解获取service
    private UserService userService;

}
```



### 测试


```java
@RunWith(SpringJUnit4ClassRunner.class) //单元测试框架
//加载spring核心配置文件
//@ContextConfiguration(value = {"classpath:applicationContext.xml"})
//加载spring核心配置类
@ContextConfiguration(classes = {SpringConfiguration.class})
public class JTest4 {

    @Autowired
    private UserService userService;	//注解获取service

    @Test
    public void testUserService(){
        userService.save();
    }
    
}
```



## 4、spring集成Junit5


### 导入spring集成Junit的坐标


```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.5.2</version>
    <scope>test</scope>
</dependency>
```



### 测试


```java
import org.junit.jupiter.api.Test;

@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:applicationContext.xml")
class JTest5 {

    @Autowired
    private UserService userService;

    @Test
    public void test1() {
        userService.accountMoney();
    }
}
```



+ 使用一个复合注解替代上面两个注解完成整合



```java
import org.junit.jupiter.api.Test;

@SpringJUnitConfig(locations = "classpath:applicationContext.xml")
class JTest5 {

    @Autowired
    private UserService userService;

    @Test
    public void test1() {
        userService.accountMoney();
    }
}
```





> 更新: 2023-06-13 10:43:04  
> 原文: <https://www.yuque.com/like321/kwpbuz/cn8dyg>