# 十四、整合-Thymeleaf

### 简介


+ 视图模板技术



我们熟悉的JSP其实只是视图模板技术的一种，除了JSP还有很多其他技术也可以帮助我们实现服务器端渲染。例如：Freemarker、Thymeleaf、



Velocity等等。其中最优秀的是Thymeleaf。而JSP因为不容易从jar包中读取而不被SpringBoot所推荐。



+ JSP相对于Thymeleaf的巨大缺陷



前端工程师拿到JSP文件后，没法直接放在浏览器上查看布局、动态效果等等显示效果。所以，基于JSP无法和前端工程师进行协同工作。



而Thymeleaf中所有的动态效果都不影响原本HTML标签的显示。当一个前端工程师拿到Thymeleaf文件后，可以直接放在浏览器上查看效果，看代码时忽略动态部分即可。



### 在SpringBoot环境下加入使用Thymeleaf


+ 依赖



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```



+ yml配置



```yaml
spring:
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
    cache: false #开发时禁用缓存
```



+ handler



```java
@Controller
public class TestTemplateHandler {

    @RequestMapping("/test/thymeleaf")
    public String testThymeleaf(){
        return "hello";
    }

}
```



+ hello.html



```html
<!DOCTYPE html>
<!--加入名称空间-->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <p th:text="经过服务器处理后可以看到的内容">直接在浏览器上打开时可以看到的内容</p>
</body>
</html>
```



### 表达式语法


+ 名称空间



使用Thymeleaf时我们直接创建HTML文件即可，只是需要在html标签中加入Thymeleaf的名称空间



```html
<!--加入名称空间-->
<html xmlns:th="http://www.thymeleaf.org">
```



+ 修改标签文本值



```html
<h3>替换标签体</h3>
<p th:text="新值">原始值</p>
```



+ 修改指定属性值



```html
<h3>替换属性值</h3>
<input type="text" value="old-value" th:value="new-value"/>
```



+ 在表达式中访问属性域



```java
@Controller
public class TestTemplateHandler {

    @Autowired
    private ServletContext servletContext;

    @RequestMapping("/test/thymeleaf")
    public String testThymeleaf(ModelMap modelMap, HttpSession httpSession){

        //  1、将测试数据存入请求域
        modelMap.addAttribute("attrNameRequestScope","attrValueRequestScope");

        // 2、将测试数据存入会话域
        httpSession.setAttribute("attrNameSessionScope","attrValueSessionScope");

        // 3、将测试数据存入应用域
        servletContext.setAttribute("attrNameAppScope","attrValueAppScope");

        return "hello";
    }

}
```



```html
<h3>访问请求域</h3>
<p th:text="${attrNameRequestScope}">aaa</p>
或
<p th:text="${#httpServletRequest.getAttribute('attrNameRequestScope')}">这里注意给属性名加引号</p>

<h3>访问Session域</h3>
<p th:text="${session.attrNameSessionScope}">bbb</p>

<h3>访问Application域</h3>
<p th:text="${application.attrNameAppScope}">ccc</p>
```



+ 解析URL地址



```html
<h3>解析URL地址 获取contextPath值</h3>
<p th:text="@{/aaa/bbb/ccc}">@{}的作用是把contextPath的值附加到指定的地址前</p>
<a href="../aaa/bbb/ccc.html" th:href="@{/aaa/bbb/ccc.html}">使用场景举例</a>
```



+ 直接执行表达式



```html
<h3>直接执行表达式</h3>
<p>[[${attrNameRequestScope}]] 有转义效果</p>
<p>[(${attrNameRequestScope})] 无转义效果</p>
```



+ 条件判断



```html
<h3>判断字符串是否为空</h3>
<p th:if="${not #strings.isEmpty(attrNameRequestScope)}">attrNameRequestScope不为空</p>
---<p th:if="${#strings.isEmpty(attrNameRequestScope)}">attrNameRequestScope为空</p>---
```



+ 遍历集合



```java
@Controller
public class TestTemplateHandler {

    @RequestMapping("/test/thymeleaf")
    public String testThymeleaf(ModelMap modelMap, HttpSession httpSession){

        //测试集合遍历，把集合存入请求域
        modelMap.addAttribute("list", Arrays.asList("AAA","BBB","CCC"));

        return "hello";
    }

}
```



```html
<h3>测试循环</h3>
<!--使用th:each进行集合数据迭代-->
<!--th:each = "声明变量 : ${集合}"-->
<!--th:each 用在哪个标签上，哪个标签就会多次出现-->

<div>
    <p th:text="${str}" th:each="str : ${list}"></p>
</div>
```



+ 包含其他模板文件



声明代码片段：



include/part.html



```html
<div th:fragment="myFirstPart">

    <p>被包含的内容</p>

</div>
```



包含：



```html
<div th:insert="~{include/part :: myFirstPart}">

</div>
```

| 表达式 | 作用 |
| --- | --- |
| th:insert="~{xxx :: zzz}" | 以插入方式引入（使用代码片段整体插入当前标签内） |
| th:replace="~{xxx :: zzz}" | 以替换方式引入（使用代码片段整体替换当前标签） |
| th:include="~{xxx :: zzz}" | 以包含方式引入（使用代码片段内容包含到当前标签内） |




XXX部分是访问模板文件时的逻辑视图名称，要求这个xxx拼接yml里配置的前后缀之后得到访问模板文件的物理视图。



比如：想包含templates/include/aaa.html页面，那么xxx就是include/aaa，因为templates是前缀，.html是后缀。



> 更新: 2022-08-19 14:43:39  
> 原文: <https://www.yuque.com/like321/mdsi9b/kio8gv>