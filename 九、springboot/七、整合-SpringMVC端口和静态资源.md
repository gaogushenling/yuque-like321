# 七、整合-SpringMVC端口和静态资源

**目标**：可以修改tomcat的端口和访问项目中的静态资源



### 修改tomcat端口


查询**Properties，设置配置项（前缀+类变量名）到application配置文件中



```plain
#tomcat端口
server:
  port: 80
```



### 访问项目中的静态资源


+  在spring boot项目中静态资源可以放置在如下目录：  
![1660890706963-43a09240-9f85-47b0-801f-d683907a5d81.png](./img/CKz0heOgqS6h8hIQ/1660890706963-43a09240-9f85-47b0-801f-d683907a5d81-354440.png) 



+ 放置静态资源并访问这些资源



![1660890707147-96745a08-7f5a-4f51-ab3c-07a305e1750f.png](./img/CKz0heOgqS6h8hIQ/1660890707147-96745a08-7f5a-4f51-ab3c-07a305e1750f-950994.png)



> 更新: 2022-08-19 14:32:29  
> 原文: <https://www.yuque.com/like321/mdsi9b/uzul0z>