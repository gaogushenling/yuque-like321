# 五、SpringMVC的请求

## 1、请求参数类型


客户端请求参数的格式是：name=value&name=value……



服务器端要获得请求的参数，有时还需要进行数据的封装，SpringMVC可以接收如下类型的参数



+  基本类型参数 
+  POJO类型参数 
+  数组类型参数 
+  集合类型参数 



## 2、获得基本类型参数


Controller中的业务方法的参数名称要与请求参数的name一致，参数值会自动映射匹配。



并且能自动做类型转换；



自动的类型转换是指<font style="color:#E8323C;">从String向其他类型的转换</font>



`http://localhost:8080/itheima_springmvc1/quick9?username=zhangsan&age=12`



```java
@RequestMapping(value = "/quick11", method = RequestMethod.GET)
@ResponseBody
public void save11(String username, int age) {

    System.out.println(username);
    System.out.println(age);

}
```



## 3、获得POJO类型参数


Controller中的业务方法的POJO参数的属性名与请求参数的name一致，参数值会自动映射匹配。



```java
public class User {

    private String username;
    private int age;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", age=" + age +
                '}';
    }
}
```



[http://localhost:8080/itheima_spring_mvc_war_exploded/user/quick12?username=zhangsan&age=18](http://localhost:8080/itheima_spring_mvc_war_exploded/user/quick12?username=zhangsan&age=18)



```java
@RequestMapping(value = "/quick12", method = RequestMethod.GET)
@ResponseBody
public void save12(User user) {

    System.out.println(user);

}
```



## 4、获得数组类型参数


Controller中的业务方法数组名称与请求参数的name一致，参数值会自动映射匹配。



[http://localhost:8080/itheima_spring_mvc_war_exploded/user/quick13?strs=aaa&strs=bbb&strs=ccc](http://localhost:8080/itheima_spring_mvc_war_exploded/user/quick13?strs=aaa&strs=bbb&strs=ccc)



```java
@RequestMapping(value = "/quick13", method = RequestMethod.GET)
@ResponseBody
public void save13(String[] strs) {

    System.out.println(Arrays.asList(strs));

}
```



## 5、form获得集合类型参数


获得集合参数时，要将集合参数包装到一个POJO中才可以。



```java
package com.itheima.domain;

import java.util.List;

public class VO {

    private List<User> userList;

    public List<User> getUserList() {
        return userList;
    }

    public void setUserList(List<User> userList) {
        this.userList = userList;
    }

    @Override
    public String toString() {
        return "VO{" +
                "userList=" + userList +
                '}';
    }
}
```



```java
@RequestMapping(value = "/quick14", method = RequestMethod.POST)
@ResponseBody
public void save14(VO vo) {

    System.out.println(vo);

}
```



```plain
<form action="${pageContext.request.contextPath}/user/quick14" method="post">
<%--    表明是第几个User对象的username age--%>
    <input type="text" name="userList[0].username"><br>
    <input type="text" name="userList[0].age"><br>
    <input type="text" name="userList[1].username"><br>
    <input type="text" name="userList[1].age"><br>
    <input type="submit" value="提交">
</form>
```



## 6、ajax获得集合类型参数


当使用ajax提交时，可以指定contentType为json形式，



那么在方法参数位置使用@RequestBody可以<font style="color:#E8323C;">直接接收集合数据而无需使用POJO进行包装</font>



```plain
<script src="${pageContext.request.contextPath}/js/jquery-3.3.1.js"></script>
<script>

    var userList = new Array();
    userList.push({username: "张三", age: 18})
    userList.push({username: "李四", age: 28})

    $.ajax({
        type: "post",
        url: "${pageContext.request.contextPath}/user/quick15",
        data: JSON.stringify(userList),
        contentType: "application/json;charset=utf-8"
    })
</script>
```



```java
@RequestMapping(value = "/quick15", method = RequestMethod.POST)
@ResponseBody
public void save15(@RequestBody List<User> userList) {

    System.out.println(userList);

}
```



## 7、静态资源访问权限的开启


当有静态资源需要加载时，比如jquery文件，通过谷歌开发者工具抓包发现，没有加载到jquery文件，



原因是SpringMVC的前端控制器DispatcherServlet的url-pattern配置的是/，代表对所有的资源都进行过滤操作，



我们可以通过以下两种方式指定放行静态资源：



### 在spring-mvc.xml配置文件中指定放行的资源


```xml
<!--    开放资源的访问-->
<mvc:resources mapping="/js/**" location="/js/"/>
<mvc:resources mapping="/img/**" location="/img/"/>
```



### 使用`<mvc:default-servlet-handler/>`标签


```xml
<!--    springMvc找不到时，交由原始的容器---Tomcat去找-->
<mvc:default-servlet-handler/>
```



## 8、解决POST获取请求参数的乱码问题（web.xml配置全局乱码过滤器）


当post请求时，数据会出现乱码，

可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，但是必须在web.xml中进行注册



```xml
<!--    配置全局过滤的filter-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <!--设置响应编码-->
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



注：



> SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效
>



## 9、@RequestParam  参数绑定注解 


当请求的参数名称与Controller的业务方法参数名称不一致时，就需要通过@RequestParam注解显示的绑定



```html
<form action="${pageContext.request.contextPath}/quick16" method="post">
    <input type="text" name="name"><br>
    <input type="submit" value="提交"><br>
</form>
```



```java
@RequestMapping(value = "/quick16")
@ResponseBody
public void save16(@RequestParam(value = "name",required = false,defaultValue = "itcast") String username) {

    System.out.println(username);

}
```



## 10、@PathVariable  Restful风格的参数的获取(应用) 


Restful是一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。



主要用于客户端和服务器交互类的软件，基于这个风格设计的软件可以<font style="color:#E8323C;">更简洁，更有层次，更易于实现缓存机制等。</font>



Restful风格的请求是使用“url+请求方式”表示一次请求目的的，HTTP 协议里面四个表示操作方式的动词如下：



GET：用于获取资源



POST：用于新建资源



PUT：用于更新资源



DELETE：用于删除资源



例如：



/user/1    GET ：       得到 id = 1 的 user



/user/1   DELETE：  删除 id = 1 的 user



/user/1    PUT：       更新 id = 1 的 user



/user       POST：      新增 user



上述url地址/user/1中的1就是要获得的请求参数，在SpringMVC中可以<font style="color:#E8323C;">使用占位符进行参数绑定。</font>



地址/user/1可以写成/user/{id}，占位符{id}对应的就是1的值。



在业务方法中我们可以使用@PathVariable注解进行占位符的匹配获取工作。



`http://localhost:8080/itheima_spring_mvc_war_exploded/user/quick17/zhangsan`



```java
@RequestMapping(value = "/quick17/{username}")
@ResponseBody
public void save17(@PathVariable(value = "username") String username) {

    System.out.println(username);

}
```



## 11、自定义类型转换器


SpringMVC 默认已经提供了一些常用的类型转换器，例如客户端提交的字符串转换成int型进行参数设置。



但是不是所有的数据类型都提供了转换器，没有提供的就需要自定义转换器，



例如：<font style="color:#E8323C;">日期类型的数据就需要自定义转换器</font>



自定义类型转换器的开发步骤：

1. 定义转换器类实现Converter接口
2. 在配置文件中声明转换器
3. 在中引用转换器



### 定义转换器类实现Converter接口


```java
public class DateConverter implements Converter<String, Date> {
    
    public Date convert(String dateStr) {
        //将日期字符串转换成日期对象 返回
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        Date date = null;
        try {
            date = format.parse(dateStr);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
    
}
```



### 在配置文件中声明转换器


```xml
<!--    mvc的注解驱动-->
<mvc:annotation-driven conversion-service="conversionService2"/>

<!--    声明转换器-->
<bean id="conversionService2" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <list>
            <bean id="dateConverter2" class="com.itheima.converter.DateConverter"></bean>
        </list>
    </property>
</bean>
```



```java
@RequestMapping(value="/quick18")
@ResponseBody
public void save18(Date date) throws IOException {
    System.out.println(date);
}
```



## 12、获得Servlet相关API


SpringMVC支持使用原始ServletAPI对象作为控制器方法的参数进行注入，



常用的对象如下：



HttpServletRequest



HttpServletResponse



HttpSession



```java
@RequestMapping(value="/quick19")
@ResponseBody
public void save19(HttpServletRequest request, HttpServletResponse response, HttpSession session) {
    System.out.println(request);
    System.out.println(response);
    System.out.println(session);
}
```



## 13、@RequestHeader获得请求头信息


使用@RequestHeader可以获得请求头信息，相当于web阶段学习的request.getHeader(name)  



@RequestHeader注解的属性如下：



value：请求头的名称



required：是否必须携带此请求头



```java
@RequestMapping(value="/quick20")
@ResponseBody
public void save20(@RequestHeader(value = "User-Agent",required = false) String user_agent) {
    System.out.println(user_agent);
}
```



## 14、@CookieValue可以获得指定Cookie的值


使用@CookieValue可以获得指定Cookie的值



@CookieValue注解的属性如下：



value：指定cookie的名称



required：是否必须携带此cookie



```java
@RequestMapping(value="/quick21")
@ResponseBody
public void save21(@CookieValue(value = "JSESSIONID",required = false) String jsessionId) {
    System.out.println(jsessionId);

}
```





> 更新: 2022-08-18 23:08:14  
> 原文: <https://www.yuque.com/like321/nrum0k/fwegif>