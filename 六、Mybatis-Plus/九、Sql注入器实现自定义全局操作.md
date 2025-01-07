# 九、Sql 注入器实现自定义全局操作

我们已经知道，在MP中，通过AbstractSqlInjector将BaseMapper中的方法注入到了Mybatis容器，这样这些方法才可以正常执行。



那么，如果我们需要<font style="color:#E8323C;">扩充BaseMapper中的方法，又该如何实现呢？</font>



下面我们以扩展findAll方法为例进行学习。



## 4.1、编写MyBaseMapper


```java
package cn.itcast.mp.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;

import java.util.List;

public interface MyBaseMapper<T> extends BaseMapper<T> {

    List<T> findAll();

    // 扩展其他的方法
}
```



其他的Mapper都可以继承该Mapper，这样实现了统一的扩展。



如：



```java
package cn.itcast.mp.mapper;

import cn.itcast.mp.pojo.User;

public interface UserMapper extends MyBaseMapper<User> {

    User findById(Long id);

}
```



## 4.2、编写MySqlInjector


如果直接继承AbstractSqlInjector的话，原有的BaseMapper中的方法将失效，所以我们选择继承DefaultSqlInjector进行扩展。



```java
package cn.itcast.mp.injectors;

import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.injector.DefaultSqlInjector;

import java.util.List;

public class MySqlInjector  extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList() {

        //父类中的方法
        List<AbstractMethod> methodList = super.getMethodList();

        // 再扩充自定义的方法
        methodList.add(new FindAll());

        return methodList;
    }
}
```



## 4.3、编写FindAll


```java
package cn.itcast.mp.injectors;

import com.baomidou.mybatisplus.core.injector.AbstractMethod;
import com.baomidou.mybatisplus.core.metadata.TableInfo;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.SqlSource;

public class FindAll extends AbstractMethod {
    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {

        String sqlMethod = "findAll";
        String sql = "select * from " + tableInfo.getTableName();
        SqlSource sqlSource = languageDriver.createSqlSource(configuration, sql, modelClass);

        return this.addSelectMappedStatement(mapperClass, sqlMethod, sqlSource, modelClass, tableInfo);
    }
}
```



## 4.4、注册到Spring容器


```java
package cn.itcast.mp;

import cn.itcast.mp.injectors.MySqlInjector;
import cn.itcast.mp.plugins.MyInterceptor;
import com.baomidou.mybatisplus.core.parser.ISqlParser;
import com.baomidou.mybatisplus.extension.parsers.BlockAttackSqlParser;
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import com.baomidou.mybatisplus.extension.plugins.SqlExplainInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.ArrayList;
import java.util.List;

@Configuration
@MapperScan("cn.itcast.mp.mapper")//设置mapper接口的扫描包
public class MybatisPlusConfig {
    
    /**
    * 自定义SQL注入器
    */
    @Bean
    public MySqlInjector mySqlInjector(){
        return new MySqlInjector();
    }
    
}
```



## 4.5、测试


```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserMapperTest {
    
    @Autowired
    private UserMapper userMapper;
    
    @Test
    public void testFindAll() {
        List<User> users = userMapper.findAll();
        for (User user : users) {
            System.out.println(user);
        }
    }
    
}
```



**输出的SQL：**



```plain
 Time：32 ms - ID：cn.itcast.mp.mapper.UserMapper.findAll
Execute SQL：
    select
        * 
    from
        tb_user
```



至此，我们实现了全局扩展SQL注入器。



> 更新: 2022-08-19 12:48:15  
> 原文: <https://www.yuque.com/like321/he07pe/oa22vd>