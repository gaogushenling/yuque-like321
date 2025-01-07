# 八、整合-SpringMVC拦截器



**目标**：可以在Spring Boot项目中配置自定义SpringMVC拦截器



### 编写拦截器（实现HandlerInterceptor）


```java
package com.itheima.interceptor;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Slf4j
public class MyInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.debug("这是MyInterceptor的preHandle方法");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.debug("这是MyInterceptor的postHandle方法");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.debug("这是MyInterceptor的afterCompletion方法");
    }
}
```



```plain
#日志记录级别
logging:
  level:
    com.itheima: debug
    org.springframework: info
```



### 编写配置类实现 WebMvcConfigurer


可以在spring boot项目中通过配置类添加各种组件；



+ 如果要添加拦截器的话：



```java
package com.itheima.config;

import com.itheima.interceptor.MyInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MvcConfig implements WebMvcConfigurer {

    //注册拦截器
    @Bean
    public MyInterceptor myInterceptor() {
        return new MyInterceptor();
    }

    //添加拦截器到spring MVC拦截器链
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(myInterceptor()).addPathPatterns("/*");
    }

}
```





> 更新: 2022-08-19 14:33:57  
> 原文: <https://www.yuque.com/like321/mdsi9b/iopmgg>