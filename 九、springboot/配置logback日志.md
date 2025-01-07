# 配置logback日志

Logback是由log4j创始人设计的又一个开源日志组件。



logback当前分成三个模块：logback-core，logback- classic 和 logback-access。



+  logback-core是其它两个模块的基础模块。 
+  logback-classic是log4j的一个改良版本。此外logback-classic完整实现slf4j API，使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging。 
+  logback-access访问模块与Servlet容器集成提供通过Http来访问日志的功能。 



## logback,slf4j,log4j之间的关系


Slf4j是The Simple Logging Facade for Java的简称，是一个简单日志门面抽象框架，它本身只提供了日志Facade API和一个简单的日志类实现，一般常配合Log4j，LogBack，java.util.logging使用。



Slf4j作为应用层的Log接入时，程序可以根据实际应用场景动态调整底层的日志实现框架(Log4j/LogBack/JdkLog...)；



LogBack和Log4j都是开源日记工具库，LogBack是Log4j的改良版本，比Log4j拥有更多的特性，同时也带来很大性能提升。



## 添加引用


```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```



## Logback.xml配置


### Logback默认配置


如果配置文件 logback-test.xml 和 logback.xml 都不存在，那么 logback 默认地会调用BasicConfigurator ，创建一个最小化配置。



最小化配置由一个关联到根 logger 的ConsoleAppender 组成。



输出模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n 的 PatternLayoutEncoder 进行格式化。



root logger 默认级别是 DEBUG。



**步骤：**



1.  尝试在 classpath 下查找文件 logback-test.xml； 
2.  如果文件不存在，则查找文件 logback.xml； 
3.  如果两个文件都不存在，logback 用 BasicConfigurator 自动对自己进行配置，这会导致记录输出到控制台。 



### 添加Logback.xml


```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="E:\work\log" />
    
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- 按照每天生成日志文件 -->
    <appender name="FILE"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/log.%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>
    
    <!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder"  level="TRACE" />
    <logger name="org.hibernate.type.descriptor.sql.BasicExtractor"  level="DEBUG" />
    <logger name="org.hibernate.SQL" level="DEBUG" />
    <logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
    <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />

    <!--myibatis log configure-->
    <logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>

    <!-- 日志输出级别 -->
    <root level="ERROR">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>
    
    <!--日志异步到数据库 -->
    <!--<appender name="DB" class="ch.qos.logback.classic.db.DBAppender">-->
    <!--<!–日志异步到数据库 –>-->
    <!--<connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">-->
    <!--<!–连接池 –>-->
    <!--<dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">-->
    <!--<driverClass>com.mysql.jdbc.Driver</driverClass>-->
    <!--<url>jdbc:mysql://127.0.0.1:3306/databaseName</url>-->
    <!--<user>root</user>-->
    <!--<password>root</password>-->
    <!--</dataSource>-->
    <!--</connectionSource>-->
    <!--</appender>-->
    
</configuration>
```



> 更新: 2022-08-19 16:26:50  
> 原文: <https://www.yuque.com/like321/mdsi9b/fmabgm>