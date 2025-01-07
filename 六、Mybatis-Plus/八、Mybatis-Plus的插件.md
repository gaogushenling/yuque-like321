# 八、Mybatis-Plus的插件

## 3.1、mybatis的插件机制


MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。



默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

+  Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed) 
+  ParameterHandler (getParameterObject, setParameters) 
+  ResultSetHandler (handleResultSets, handleOutputParameters) 
+  StatementHandler (prepare, parameterize, batch, update, query) 



我们看到了可以拦截Executor接口的部分方法，比如update，query，commit，rollback等方法，还有其他接口的一些方法等。



总体概括为：

1. 拦截执行器的方法
2. 拦截参数的处理
3. 拦截结果集的处理
4. 拦截Sql语法构建的处理



### 拦截器示例：
+ 实现Interceptor接口

```java
package cn.itcast.mp.plugins;

import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.plugin.*;

import java.util.Properties;

@Intercepts({@Signature(
        type = Executor.class,
        method = "update",
        args = {MappedStatement.class, Object.class})})
public class MyInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        //拦截方法，具体业务逻辑编写的位置
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        //创建target对象的代理对象,目的是将当前拦截器加入到该对象中
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
        //属性设置
    }
}
```



+ 注入到Spring容器：



```java
@Configuration
@MapperScan("cn.itcast.mp.mapper")//设置mapper接口的扫描包
public class MybatisPlusConfig {

    @Bean //注入自定义的拦截器（插件）
    public MyInterceptor myInterceptor(){
        return new MyInterceptor();
    }

}
```



或者通过xml配置，mybatis-config.xml：



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <plugins>
        <plugin interceptor="cn.itcast.mp.plugins.MyInterceptor"></plugin>
    </plugins>
</configuration>
```



## 3.2、执行分析插件


在MP中提供了对SQL执行的分析的插件，可用作阻断全表更新、删除的操作，



注意：该插件仅适用于开发环境，不适用于生产环境。



### SpringBoot配置：


```java
@Configuration
@MapperScan("cn.itcast.mp.mapper")//设置mapper接口的扫描包
public class MybatisPlusConfig {

    @Bean //SQL分析插件
    public SqlExplainInterceptor sqlExplainInterceptor (){
        SqlExplainInterceptor sqlExplainInterceptor = new SqlExplainInterceptor();

        List<ISqlParser> list = new ArrayList<>();
        // 攻击 SQL 阻断解析器、加入解析链
        list.add(new BlockAttackSqlParser());// 全表更新删除的阻断器
        sqlExplainInterceptor.setSqlParserList(list);

        return sqlExplainInterceptor;
    }

}
```



### 测试：


```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserMapperTest {
    
    @Autowired
    private UserMapper userMapper;
    
    /**
     * 测试全表更新，SQL分析器阻断效果
     */
    @Test
    public void testUpdateAll(){
        User user = new User();
        user.setAge(31); //更新的数据

        boolean update = user.update(null);//全表更新
        System.out.println(update);
    }
    
}
```



可以看到，当执行全表更新时，会抛出异常，这样有效防止了一些误操作。



## 3.3、性能分析插件


性能分析拦截器，用于输出每条 SQL 语句及其执行时间，可以设置最大执行时间，超过时间会抛出异常。



> 该插件只用于开发环境，不建议生产环境使用。
>



### 配置：


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <plugins>
        <!--        性能分析插件-->
        <plugin interceptor="com.baomidou.mybatisplus.extension.plugins.PerformanceInterceptor">
            <!--            最大的执行时间，单位为毫秒-->
            <property name="maxTime" value="100"/>
            <!--            对输出的SQL做格式化，默认为false-->
            <property name="format" value="true"/>
        </plugin>
    </plugins>

</configuration>
```



### 执行结果：


```plain
Time ：11 ms - ID：cn.itcast.mp.mapper.UserMapper.selectById
Execute SQL：
 SELECT
   id,
   user_name,
   password,
   name,
   age,
   email
 FROM
   tb_user
 WHERE
    id=7
```



可以看到，执行时间为11ms。



如果将maxTime设置为 1 ，那么，该操作会抛出异常。



```plain
Caused  by: com.baomidou.mybatisplus.core.exceptions.MybatisPlusException: The SQL
execution time is too large, please optimize !
at com.baomidou.mybatisplus.core.toolkit.ExceptionUtils.mpe(ExceptionUtils.java:49)
at com.baomidou.mybatisplus.core.toolkit.Assert.isTrue(Assert.java:38)
................
```



## 3.4、乐观锁插件


### 主要适用场景


意图：当要更新一条记录的时候，希望这条记录没有被别人更新



乐观锁实现方式：

+ 取出记录时，获取当前version
+ 更新时，带上这个version
+ 执行更新时， set version = newVersion where version = oldVersion
+ 如果version不对，就更新失败



### 插件配置


第一种：spring xml:



```xml
<bean class="com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor"/>
```



第二种：spring boot:



```java
@Bean
public OptimisticLockerInterceptor optimisticLockerInterceptor() {
    return new OptimisticLockerInterceptor();
}
```



第三种：mybatis



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <plugins>
        <!--        乐观锁插件-->
        <plugin interceptor="com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor"/>
    </plugins>

</configuration>
```



### 注解实体字段


需要为实体字段添加@Version注解。



> 第一步，为表添加version字段，并且设置初始值为 1 ：
>



```sql
ALTER TABLE `tb_user` ADD COLUMN `version`  int(10) NULL AFTER `email`;

UPDATE `tb_user` SET `version`='1';
```



> 第二步，为User实体对象添加version字段，并且添加@Version注解：
>



```java
@Data
@NoArgsConstructor
@AllArgsConstructor
//@TableName("tb_user")
public class User extends Model<User> {

//    @TableId(type = IdType.AUTO)
    private Long id;
    private String userName;

    @TableField(select = false) //查询时不返回该字段的值
    private String password;

    private String name;
    private Integer age;

    @TableField(value = "email") //指定数据表中的字段名
    private String email;

    @TableField(exist = false)
    private String address; //在数据表中是不存在的

    @Version //乐观锁的版本字段
    private Integer version;
}
```



### 测试


+ 测试用例



```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserMapperTest {
    
    @Autowired
    private UserMapper userMapper;
    
    /**
     * 测试乐观锁
     */
    @Test
    public void testUpdateVersion() {
        User user = new User();
        user.setId(10L);//查询条件

        user.setAge(31);//更新条件的数据

        User userVersion = user.selectById();
        user.setVersion(userVersion.getVersion()); //当前的版本信息

        boolean result = user.updateById();
        System.out.println(result);

    }
    
}
```



+ 执行日志



可以看到，更新的条件中有version条件，并且更新的version为 2 。



如果再次执行，更新则不成功。



这样就避免了多人同时更新时导致数据的不一致。



### 特别说明


+ 支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime
+ 整数类型下 newVersion = oldVersion + 1
+ newVersion 会回写到 entity 中
+ 仅支持 updateById(id) 与 update(entity, wrapper) 方法
+ 在 update(entity, wrapper) 方法下, wrapper 不能复用!!!



> 更新: 2022-08-19 12:46:28  
> 原文: <https://www.yuque.com/like321/he07pe/obvwgo>