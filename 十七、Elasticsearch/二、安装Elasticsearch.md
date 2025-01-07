# 二、安装Elasticsearch

## 1 安装JDK


ElasticSearch是用JAVA语言开发的，其运行需要安装JDK。



JDK (Java Development Kit)  ，是整个Java的核心，包括了Java运行环境（Java Runtime Envirnment），一堆Java工具和Java基础的类库(rt.jar)。



### 1.1 下载安装JDK


下载地址[https://www.oracle.com/technetwork/java/javase/downloads/index.html](https://www.oracle.com/technetwork/java/javase/downloads/index.html)



![1558512359460.png](./img/3w_vsg_eJtGjvsGk/1638974635236-f0732255-6080-4548-a9cc-aa0f1781ad51-817171.png)



![1558787699684.png](./img/3w_vsg_eJtGjvsGk/1638974635209-dc4eca9d-2928-43c8-8778-1d588f9ba6db-356017.png)



安装：双击 软件 打开安装界面



![1558788055103.png](./img/3w_vsg_eJtGjvsGk/1638974635185-63907378-ca4f-409f-8d19-69b6d01078bd-224906.png)



点击下一步 进入选择安装目录界面



![1558788142922.png](./img/3w_vsg_eJtGjvsGk/1638974635145-5d199e20-7be5-4c4f-83d3-89b6df84a455-987240.png)



点击更改 自定义安装目录



![1558788294354.png](./img/3w_vsg_eJtGjvsGk/1638974635156-92d8e35c-6b7b-406d-9fe1-6e4d1b5a13d0-752559.png)



点击 下一步  进行安装



![1558788384212.png](./img/3w_vsg_eJtGjvsGk/1638974635183-4af808df-866a-4475-a3f4-6f1a96cb50ce-489819.png)



等待，出现以下界面，则安装完成，点击关闭即可。



![1558788443016.png](./img/3w_vsg_eJtGjvsGk/1638974635216-c35c269a-9e07-4320-a7cf-471cfac7c8a5-203276.png)



### 1.2 配置环境变量


配置 JAVA_HOME环境变量



![1558789277385.png](./img/3w_vsg_eJtGjvsGk/1638974635272-0201d8ba-5b7c-463f-a7be-8c5a36cb3695-922504.png)



配置Path环境变量



![1558513120626.png](./img/3w_vsg_eJtGjvsGk/1638974635224-30e71fa4-303b-4a3c-8c36-a46133ad3c0f-426530.png)



### 1.3 测试-查看JDK版本


打开命令行窗口，输入`java -version`查看JDK版本



![1558789368309.png](./img/3w_vsg_eJtGjvsGk/1638974635188-0b3fe445-e880-4acb-9cf4-903bd860d9bc-391307.png)



出现以上界面，说明安装成功。



## 2 安装Elasticsearch


权威指南[https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)



### 2.1 下载安装


下载地址[https://www.elastic.co/downloads](https://www.elastic.co/downloads)



![image-20201015230159226.png](./img/3w_vsg_eJtGjvsGk/1638974635285-e958f7a5-9c8a-41b3-bccf-65576e0f00ca-733872.png)



![image-20201015230630412.png](./img/3w_vsg_eJtGjvsGk/1638974635281-aa285193-1387-42fe-91bb-c66dcf3a6c4a-361975.png)



Windows 版的 Elasticsearch 的安装很简单，解压即安装完毕，解压后的 Elasticsearch 的目录结构如下



![image-20211207234540692.png](./img/3w_vsg_eJtGjvsGk/1639379655700-a1b7c5f8-f1da-42fd-a3a6-e618901dac74-624710.png)

| 目录 | 含义 |
| --- | --- |
| bin | 可执行脚本目录 |
| config | 配置目录 |
| jdk | 内置 JDK 目录 |
| lib | 类库 |
| logs | 日志目录 |
| modules | 模块目录 |
| plugins | 插件目录 |




> bin：启动文件
>
>  
>
> config：配置文件
>
>  
>
>  
>
> data：索引数据目录
>
>  
>
> lib：相关类库Jar包
>
>  
>
> logs：日志目录
>
>  
>
> modules：功能模块
>
>  
>
> plugins：插件
>

> log4j2.properties：日志配置文件
>
>  
>
> jvm.options：java虚拟机的配置
>
>  
>
> elasticsearch.yml：es的配置文件
>



**解压后，进入 bin 文件目录，点击 elasticsearch.bat 文件启动 ES 服务**



![image-20211207235032937.png](./img/3w_vsg_eJtGjvsGk/1639379750533-9bd0d142-2b24-48d4-8d1c-58ede47edd67-183148.png)



注意：**9300**  端口为 Elasticsearch  集群间组件的通信端口，**9200**  端口为浏览器访问的 http 协议 RESTful  端口。



打开浏览器，输入地址：http://localhost:9200，测试结果



![image-20211207235204709.png](./img/3w_vsg_eJtGjvsGk/1639379750489-95d1225b-8700-4966-a1f1-ac61ff7f5228-211424.png)



### 2.2  问题解决


+ Elasticsearch 是使用 java 开发的，且 7.8 版本的 ES 需要 JDK 版本 1.8 以上，默认安装包带有 jdk 环境，如果系统配置 JAVA_HOME，那么使用系统默认的 JDK，如果没有配置使用自带的 JDK，一般建议使用系统配置的 JDK。
+ 双击启动窗口闪退，通过路径访问追踪错误，如果是“空间不足”，请修改config/jvm.options 配置文件



```plain
# 设置 JVM 初始内存为 1G。此值可以设置与-Xmx 相同，以避免每次垃圾回收完成后 JVM 重新分配内存
# Xms represents the initial size of total heap space
# 设置 JVM 最大可用内存为 1G
# Xmx represents the maximum size of total heap space

-Xms1g
-Xmx1g
```



### 2.3 配置Path环境变量


(bin目录)



![image-20201016211107078.png](./img/3w_vsg_eJtGjvsGk/1638974639949-9f37646c-1662-4c55-ab91-66d2347b6414-738030.png)



### 2.4 中文分词插件


GitHub：[https://github.com/medcl/elasticsearch-analysis-ik/releases](https://github.com/medcl/elasticsearch-analysis-ik/releases)



![image-20210117173749207.png](./img/3w_vsg_eJtGjvsGk/1638974639951-2507f3cc-aa76-4e5d-98c2-430f2c9849bb-955282.png)



解压到elasticsearch插件目录



![image-20210117173912738.png](./img/3w_vsg_eJtGjvsGk/1638974639904-15c7b7cd-4e66-4d02-b2c4-686f2c4f15cc-413616.png)



### 2.5 启动elasticsearch


打开命令行窗口  执行命令 elasticsearch -d  启动elasticsearch



![image-20201016211500118.png](./img/3w_vsg_eJtGjvsGk/1638974639913-88903e76-4aae-4ac7-aa5b-3f5611d3b768-392256.png)



注：该命令行窗口 不要关闭。



浏览器打开 [http://localhost:9200](http://localhost:9200)



![image-20201016211805650.png](./img/3w_vsg_eJtGjvsGk/1638974639987-1b310fca-b80b-4045-a9f9-c701d5e5edca-394224.png)



出现以上界面，则启动成功。



### 2.6 Elasticsearch-Head（可视化工具）


elasticsearch-head是一个用于浏览ElasticSearch集群并与其进行交互的Web项目



GitHub托管地址：[https://github.com/mobz/elasticsearch-head](https://github.com/mobz/elasticsearch-head)



下载并解压：



![image-20201016212512835.png](./img/3w_vsg_eJtGjvsGk/1638974639952-7ec483dd-f092-42f5-94d9-23caabe9276f-952597.png)



安装：打开命令行，切换到Elasticsearch-Head目录，执行以下命令



```plain
npm install
```



启动：打开命令行，切换到Elasticsearch-Head目录，执行以下命令



```plain
npm run start
```



![image-20201016213749817.png](./img/3w_vsg_eJtGjvsGk/1638974639916-e1288f4e-b7ee-44a1-9a3e-92063dc3c619-807095.png)



启动成功后，可通过http://localhost:9100进行访问



由于跨域（Elasticsearch位于9200端口），需要添加配置： D:\Java_Web\elasticsearch\elasticsearch-7.8.0\config\elasticsearch.yml中



```yaml
#新添加的配置行
http.cors.enabled: true
http.cors.allow-origin: "*"
```



重新启动elasticsearch



访问效果：



![image-20201016214140361.png](./img/3w_vsg_eJtGjvsGk/1638974639937-d3defedf-535c-4d27-b3e1-4b118166f7a4-237637.png)



> 更新: 2022-08-17 23:51:26  
> 原文: <https://www.yuque.com/like321/fk7s34/ytg9du>