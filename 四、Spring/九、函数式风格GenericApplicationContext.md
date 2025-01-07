# 九、函数式风格 GenericApplicationContext



```java
public class User {


}
```



```java
    //函数式风格创建对象，交给spring进行管理
    @Test
    public void testGenericApplicationContext() {
        
        //1、创建GenericApplicationContext
        GenericApplicationContext context = new GenericApplicationContext();
        
        //2、调用context的方法进行对象注册
        context.refresh();
//        context.registerBean(User.class, () -> new User());
//        //3、获取在spring注册的对象
//        User user = (User)context.getBean("com.atguigu.spring5.test.User");
//        System.out.println(user);


        context.registerBean("user1", User.class, () -> new User());
        //3、获取在spring注册的对象
        User user1 = (User) context.getBean("user1");
        System.out.println(user1);
    }
```



> 更新: 2023-06-13 15:04:04  
> 原文: <https://www.yuque.com/like321/kwpbuz/rtghya>