# 一、 Spring与Web环境集成

## 1、ApplicationContext应用上下文获取方式


1.  应用上下文对象是通过new ClasspathXmlApplicationContext(spring配置文件) 方式获取的，  
但是每次从容器中获得Bean时都要编写new ClasspathXmlApplicationContext(spring配置文件) ，这样的弊端是<font style="color:#E8323C;">配置文件加载多次，应用上下文对象创建多次。 </font>



2.  在Web项目中，可以使用ServletContextListener监听Web应用的启动，我们可以在Web应用启动时，就加载Spring的配置文件，创建应用上下文对象ApplicationContext，  
再将其存储到最大的域servletContext域中，这样就可以在任意位置从域中获得应用上下文ApplicationContext对象了。 



```xml
<!--  全局初始化参数-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>applicationContext.xml</param-value>
</context-param>


<!--  配置监听器-->
<listener>
    <listener-class>com.itheima.listener.ContextLoaderListener</listener-class>
</listener>
```



```java
public class ContextLoaderListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {

        ServletContext servletContext = sce.getServletContext();

        //读取web.xml中的全局参数
        String contextConfigLocation = servletContext.getInitParameter("contextConfigLocation");
        //创建spring的应用上下文对象
        ApplicationContext context = new ClassPathXmlApplicationContext(contextConfigLocation);

        //将spring的应用上下文对象，存储到ServletRequest域中
        servletContext.setAttribute("context", context);
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }
}
```



```java
public class WebApplicationContextUtils {

    public static ApplicationContext getWebApplicationContext(ServletContext servletContext) {
        return (ApplicationContext) servletContext.getAttribute("context");
    }

}
```



```java
public class UserServlet extends HttpServlet {
    
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
//        ServletContext servletContext = request.getServletContext();
//        ApplicationContext context = (ApplicationContext) servletContext.getAttribute("context");
        
        ServletContext servletContext = this.getServletContext();
        ApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(servletContext);

        UserService userService = context.getBean("userService", UserService.class);
        userService.save();
    }
    
}
```



## 2、Spring提供的获取应用上下文工具


上面的分析不用手动实现，Spring提供了一个监听器ContextLoaderListener就是对上述功能的封装，

该监听器内部加载Spring配置文件，创建应用上下文对象，并存储到ServletContext域中，

提供了一个客户端工具WebApplicationContextUtils供使用者获得应用上下文对象。



所以我们需要做的只有两件事：

+  配置ContextLoaderListener监听器 
+  使用WebApplicationContextUtils获得应用上下文 



### web.xml配置ContextLoaderListener监听器


+ pom.xml导入spring-web坐标



```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
```



+ web.xml配置ContextLoaderListener监听器



```xml
<!--  全局初始化参数-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>

<!--  配置spring监听器-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```



### 通过工具类获得应用上下文对象


使用WebApplicationContextUtils获得应用上下文对象ApplicationContext



```java
public class UserServlet extends HttpServlet {
    
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        //        通过工具获得应用上下文对象
        WebApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(servletContext);

        UserService userService = context.getBean("userService", UserService.class);
        userService.save();
    }
    
}
```





> 更新: 2022-08-18 22:19:57  
> 原文: <https://www.yuque.com/like321/nrum0k/ntun1l>