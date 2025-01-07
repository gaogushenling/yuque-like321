# 五、Redis的发布和订阅

## 1、什么是发布和订阅


Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。



Redis 客户端可以订阅任意数量的频道。



## 2、Redis的发布和订阅


1.  客户端可以订阅频道如下图：  
![image-20220109182217301.png](./img/VlEbULc7uhvfMqrA/1642600541420-0f98f608-63b9-4c58-aeac-276925c7977d-526707.png) 



2.  当给这个频道发布消息后，消息就会发送给订阅的客户端 



![image-20220109182255377.png](./img/VlEbULc7uhvfMqrA/1642600541426-c6e68e76-e335-4e0c-82bd-417e5c4f8b17-279416.png)



## 3、发布订阅命令行实现


1.  打开一个客户端订阅 channel1 

```plain
subscribe channel1
```

  
![image-20220109182751070.png](./img/VlEbULc7uhvfMqrA/1642600541378-19fd65dc-d41e-4e9a-94c4-c2c07eea4b57-579595.png) 



2.  打开另一个客户端，给channel1发布消息hello 

```plain
publish channel1 hello
```

  
![image-20220109183004940.png](./img/VlEbULc7uhvfMqrA/1642600541394-fe7fe5bb-4cfc-435a-9048-36eda9ec9736-410658.png)  
返回的1是订阅者数量 



3.  打开第一个客户端可以看到发送的消息  
![image-20220109183054380.png](./img/VlEbULc7uhvfMqrA/1642600541403-9f6150e1-8352-4b53-9a6c-e08dfa44394f-475916.png)  
注：发布的消息没有持久化，如果在订阅前的客户端收不到hello，只能收到订阅后发布的消息 



> 更新: 2022-08-11 16:07:18  
> 原文: <https://www.yuque.com/like321/qgn2qc/qfps8k>