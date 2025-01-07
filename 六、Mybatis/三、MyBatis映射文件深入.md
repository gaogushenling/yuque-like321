# 三、MyBatis映射文件深入

## 1 动态sql语句


Mybatis 的映射文件中，前面我们的 SQL 都是比较简单的，有些时候业务逻辑复杂时，我们的 SQL是动态变化的，此时在前面的学习中我们的 SQL 就不能满足要求了。



### 动态 SQL  之<**if>**


我们根据实体类的不同取值，使用不同的 SQL语句来进行查询。



比如在 id如果不为空时可以根据id查询，如果username 不为空时还要加入用户名作为条件。



这种情况在我们的多条件组合查询中经常会碰到。



```java
public interface UserMapper {

    public List<User> findByCondition(User user);

}
```



```xml
<select id="findByCondition" resultType="user" parameterType="user">
    select * from user
    <where>
        <if test="id!=0">
            and id=#{id}
        </if>
        <if test="username!=null">
            and username = #{username}
        </if>
        <if test="password!=null">
            and password = #{password}
        </if>
    </where>
</select>
```



当查询条件id和username都存在时：



```java
    //加载核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    //获得session工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得sqlSession会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //获得MyBatis框架生成的UserMapper接口的实现类
  	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    User condition = new User();
    condition.setId(1);
    condition.setUsername("lucy");

    User user = userMapper.findByCondition(condition);
    

    //释放资源
    sqlSession.close();
```



当查询条件只有id存在时：



```java
    //加载核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    //获得session工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得sqlSession会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //获得MyBatis框架生成的UserMapper接口的实现类
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    User condition = new User();
    condition.setId(1);

    User user = userMapper.findByCondition(condition);

    //释放资源
    sqlSession.close();
```



### 动态 SQL  之<**foreach>**


循环执行sql的拼接操作，例如：SELECT * FROM USER WHERE id IN (1,2,5)。



```java
public interface UserMapper {

    public List<User> findByIds(List<Integer> ids);

}
```



```xml
<select id="findByIds" parameterType="list" resultType="user">
   select * from user
   <where>
       <foreach collection="list" open="id in (" close=")" item="id" separator=",">
           #{id}
       </foreach>
   </where>
</select>
```



测试代码片段如下：



```java
    //加载核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    //获得session工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得sqlSession会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

 	//获得MyBatis框架生成的UserMapper接口的实现类
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    List<Integer> ids = new ArrayList<Integer>();
    ids.add(1);
    ids.add(2);
    ids.add(3);

    List<User> userList = mapper.findByIds(ids);
    System.out.println(userList);

    //释放资源
    sqlSession.close();
```



foreach标签的属性含义如下：



标签用于遍历集合，它的属性：



+  collection：代表要遍历的集合元素，注意编写时不要写#{} 
+  open：代表语句的开始部分 
+  close：代表结束部分 
+  item：代表遍历集合的每个元素，生成的变量名 
+  sperator：代表分隔符 



### SQL片段抽取


Sql 中可将重复的 sql 提取出来，使用时用 include 引用即可，最终达到 sql 重用的目的



```xml
<mapper namespace="com.itheima.mapper.UserMapper">

    <!--    sql语句的抽取-->
     <sql id="selectUser">select * from user</sql>

    <select id="findByCondition" resultType="user" parameterType="user">
        <include refid="selectUser"></include>
        <where>
            <if test="id!=0">
                and id=#{id}
            </if>
            <if test="username!=null">
                and username = #{username}
            </if>
            <if test="password!=null">
                and password = #{password}
            </if>
        </where>
    </select>
    <select id="findByIds" parameterType="list" resultType="user">
        <include refid="selectUser"></include>
        <where>
            <foreach collection="list" open="id in (" close=")" item="id" separator=",">
                #{id}
            </foreach>
        </where>
    </select>
</mapper>
```





> 更新: 2022-08-19 08:09:23  
> 原文: <https://www.yuque.com/like321/tziuog/gp8v1m>