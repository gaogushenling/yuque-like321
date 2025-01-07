# 十、整合-Mybatis

**目标**：配置Mybatis在Spring Boot工程中的整合包，设置mybatis的实体类别名，输出执行sql语句配置项



### 添加启动器依赖


添加mybatis官方对于spring boot的一个启动器



```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.17</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```



### 建表


```sql
CREATE TABLE `table_emp` (
  `emp_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `emp_name` varchar(100) NOT NULL DEFAULT '',
  `emp_age` int(10) unsigned DEFAULT NULL,
  PRIMARY KEY (`emp_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```



### 实体类


```java
package com.atguigu.spring.boot.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;


@Data //在编译阶段会根据注解自动生成对应的方法 data包含get/set/hashCode/equals/toString等方法
@NoArgsConstructor
@AllArgsConstructor
public class Emp {

    private Integer empId;

    private String empName;

    private Integer empAge;

}
```



### Mapper接口


+ 编写mapper接口



```java
package com.atguigu.spring.boot.mapper;

import com.atguigu.spring.boot.entity.Emp;

import java.util.List;

public interface EmpMapper {

    List<Emp> selectAll();

}
```



### Mapper配置文件


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.atguigu.spring.boot.mapper.EmpMapper">
    <select id="selectAll" resultType="com.atguigu.spring.boot.entity.Emp">
        select emp_id empId, emp_name empName, emp_age empAge
        from table_emp
    </select>
</mapper>
```



### yml.xml配置mybatis


实体类别名包，日志，映射文件等；



```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/test?serverTimezone=UTC
    username: root
    password: root

mybatis:
  mapper-locations: classpath*:/mybatis/mapper/*Mapper.xml

logging:
  level:
    com.atguigu.spring.boot.mapper: debug
    com.atguigu.spring.boot.test: debug
```



### 主启动类配置MapperScan


+ 设置启动器类中的mapper扫描



```java
package com.atguigu.spring.boot;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * spring boot工程都有一个启动引导类，这是工程的入口类
 * 并在引导类上添加@SpringBootApplication
 */
@SpringBootApplication
//MapperScan 扫描mybatis所有的mapper接口
@MapperScan("com.atguigu.spring.boot.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```



### 测试


```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class MybatisTest {

    @Autowired
    private EmpMapper empMapper;


    private Logger logger = LoggerFactory.getLogger(MybatisTest.class);

    @Test
    public void testSelectAll() {
        List<Emp> emps = empMapper.selectAll();
        for (Emp emp : emps) {
            logger.debug(emp.toString());
        }
    }

}
```





> 更新: 2022-08-19 14:37:06  
> 原文: <https://www.yuque.com/like321/mdsi9b/pn8992>