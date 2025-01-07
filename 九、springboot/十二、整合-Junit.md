# 十二、整合-Junit

**目标**：在Spring Boot项目中使用Junit进行单元测试UserService的方法



### 添加启动器依赖spring-boot-starter-test


```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```



### 编写测试类


```java
package com.itheima.service;

import com.itheima.pojo.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Date;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void queryById() {
        User user = userService.queryById(8L);
        System.out.println(user);
    }

    @Test
    public void saveUser() {
        User user = new User();

        user.setUserName("zhangsan2");
        user.setName("张三");
        user.setAge(23);
        user.setPassword("123456");
        user.setSex(1);
        user.setCreated(new Date());

        userService.saveUser(user);
    }
}
```



> 在Spring Boot项目中如果编写测试类则必须要在类上面添加@SpringBootTest  
>



> 更新: 2022-08-19 14:40:06  
> 原文: <https://www.yuque.com/like321/mdsi9b/lnkd1a>