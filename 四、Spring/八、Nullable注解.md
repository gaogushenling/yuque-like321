# 八、Nullable注解

@Nullable  注解可以使用在方法上面，属性上面，参数上面，表示方法返回可以为空，属性值可以为空，参数值可以为空 



## 注解用在方法


注解用在方法上面，方法返回值可以为空



```java
@Nullable
String getId();
```



## 注解使用在方法参数


注解使用在方法参数里面，方法参数可以为空



```java
public <T> void registerBean(@Nullable String beanName, Class<T> beanClass, @Nullable Supplier<T> supplier, BeanDefinitionCustomizer... customizers) {
    this.reader.registerBean(beanClass, beanName, supplier, customizers);
}
```



## 注解使用在属性


注解使用在属性上面，属性值可以为空



```java
@Nullable
private String username;
```





> 更新: 2023-06-13 15:04:12  
> 原文: <https://www.yuque.com/like321/kwpbuz/kl7y4q>