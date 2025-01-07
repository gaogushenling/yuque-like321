# 六、ActiveRecord

ActiveRecord（简称AR）一直广受动态语言（ PHP 、 Ruby 等）的喜爱，而 Java 作为准静态语言，对于ActiveRecord 往往只能感叹其优雅，所以我们也在 AR 道路上进行了一定的探索，喜欢大家能够喜欢。





什么是ActiveRecord？

ActiveRecord也属于ORM（对象关系映射）层，由Rails最早提出，遵循标准的ORM模型：表映射到记录，记录映射到对象，字段映射到对象属性。配合遵循的命名和配置惯例，能够很大程度的快速实现模型的操作，而且简洁易懂。



ActiveRecord的主要思想是：

+ 每一个数据库表对应创建一个类，类的每一个对象实例对应于数据库中表的一行记录；通常表的每个字段在类中都有相应的Field； 
+  ActiveRecord同时负责把自己持久化，在ActiveRecord中封装了对数据库的访问，即CURD;； 
+  ActiveRecord是一种领域模型(Domain Model)，封装了部分业务逻辑； 



## 1.1、开启AR之旅


在MP中，开启AR非常简单，只需要将实体对象继承Model即可。



```java
package cn.itcast.mp.pojo;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.extension.activerecord.Model;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

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
}
```



## 1.2、根据主键查询


```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestUserMapper2 {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelectById(){
        User user = new User();

        user.setId(2L);
        User user1 = user.selectById();
        System.out.println(user1);
    }

}
```



## 1.3、新增数据


```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestUserMapper2 {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testInsert(){
        User user = new User();
        user.setUserName("liubei");
        user.setPassword("123456");
        user.setAge(30);
        user.setName("刘备");
        user.setEmail("liubei@itcast.cn");

        // 调用AR的insert方法进行插入数据
        boolean insert = user.insert();
        System.out.println(insert);
    }

}
```



## 1.5、更新操作


```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestUserMapper2 {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testUpdate() {
        User user = new User();
        user.setId(11L);//查询条件

        user.setAge(31);//更新条件的数据

        boolean result = user.updateById();
        System.out.println(result);

    }

}
```



## 1.6、删除操作


```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestUserMapper2 {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testDelete(){
        User user = new User();

        user.setId(11L);

        boolean delete = user.deleteById();
        System.out.println(delete);
    }

}
```



## 1.7、根据条件查询


```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestUserMapper2 {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect(){
        User user = new User();

        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.ge("age",30); //大于等于30岁的用户查询

        List<User> users = user.selectList(wrapper);
        for (User user1 : users) {
            System.out.println(user1);
        }
    }

}
```





> 更新: 2022-08-19 09:58:51  
> 原文: <https://www.yuque.com/like321/he07pe/kq9pyg>