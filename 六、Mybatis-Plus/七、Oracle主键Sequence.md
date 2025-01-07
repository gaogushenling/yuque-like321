# 七、Oracle 主键Sequence

在mysql中，主键往往是自增长的，这样使用起来是比较方便的，



如果使用的是Oracle数据库，那么就不能使用自增长了，就得<font style="color:#E8323C;">使用Sequence 序列生成id值了。</font>



## 2.1、部署Oracle环境


为了简化环境部署，这里使用Docker环境进行部署安装Oracle。



```plain
# 拉取镜像
docker pull sath89/oracle-12c

#创建容器
docker create --name oracle -p 1521:1521 sath89/oracle-12c

#启动
docker start oracle && docker logs -f oracle

#下面是启动过程
Database not initialized. Initializing database.
Starting tnslsnr
Copying database files
1% complete
3% complete
11% complete
18% complete
26% complete
37% complete
Creating and starting Oracle instance
40% complete
45% complete
50% complete
55% complete
56% complete
60% complete
62% complete
Completing Database Creation
66% complete
70% complete
73% complete
85% complete
96% complete
100% complete
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/xe/xe.log" for further details.
Configuring Apex console
Database initialized. Please visit http://#containeer:8080/em
http://#containeer:8080/apex for extra configuration if needed
Starting web management console
PL/SQL procedure successfully completed.
Starting import from '/docker-entrypoint-initdb.d':
ls: cannot access /docker-entrypoint-initdb.d/*: No such file or directory
Import finished
Database ready to use. Enjoy! ;)

#通过用户名密码即可登录
用户名和密码为： system/oracle
```



## 2.2、创建表以及序列


```sql
-- 创建表，表名以及字段名都要大写
CREATE TABLE "TB_USER" (
 "ID" NUMBER(20) VISIBLE NOT NULL ,
 "USER_NAME" VARCHAR2(255 BYTE) VISIBLE ,
 "PASSWORD" VARCHAR2(255 BYTE) VISIBLE ,
 "NAME" VARCHAR2(255 BYTE) VISIBLE ,
 "AGE" NUMBER(10) VISIBLE ,
 "EMAIL" VARCHAR2(255 BYTE) VISIBLE
)
```



```sql
--创建序列
CREATE SEQUENCE SEQ_USER START WITH 1 INCREMENT BY 1
```



## 2.3、jdbc驱动包


由于版权原因，我们不能直接通过maven的中央仓库下载oracle数据库的jdbc驱动包，所以我们需要将驱动包安装到本地仓库。



```plain
mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc8 -Dversion=12.1.0.1 - Dpackaging=jar -Dfile=ojdbc8.jar
```



#### 安装完成后的坐标：


```xml
< dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>12.1.0.1</version>
</dependency>
```



## 2.4、修改application.properties


对于application.properties的修改，需要修改 2 个位置，分别是：



```properties
# 数据库连接配置
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
spring.datasource.url=jdbc:oracle:thin:@192.168.31.81:1521:xe
spring.datasource.username=system
spring.datasource.password=oracle

#id生成策略
mybatis-plus.global-config.db-config.id-type=input
```



## 2.5、配置序列


使用Oracle的序列需要做 2 件事情：



第一，需要配置MP的序列生成器到Spring容器：



```java
@Configuration
@MapperScan("cn.itcast.mp.mapper") //设置mapper接口的扫描包
public class MybatisPlusConfig {
    /**
  * 分页插件
  */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
    /**
  * 序列生成器
  */
    @Bean
    public OracleKeyGenerator oracleKeyGenerator(){
        return new OracleKeyGenerator();
    }
}
```



第二，在实体对象中指定序列的名称：



```java
@KeySequence (value = "SEQ_USER", clazz = Long.class)
public class User{
    ......
}
```



## 2.6、测试


```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserMapperTest {
    
    @Autowired
    private UserMapper userMapper;
    
    @Test
    public void testInsert(){
        User user = new User();
        user.setAge(20);
        user.setEmail("test@itcast.cn");
        user.setName("曹操");
        user.setUserName("caocao");
        user.setPassword("123456");
        int result = this.userMapper.insert(user); //返回的result是受影响的行数，并不是自增后的id
        
        System.out.println("result = " + result);
        System.out.println(user.getId()); //自增后的id会回填到对象中
    }
    
    @Test
    public void testSelectById(){
        User user = this.userMapper.selectById(8L);
        System.out.println(user);
    }
    
}
```





> 更新: 2022-08-19 10:00:05  
> 原文: <https://www.yuque.com/like321/he07pe/iy0xe4>