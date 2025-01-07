# 七、jedis操作Redis

<font style="color:rgb(77, 77, 77);">Jedis是redis的java版本的客户端实现，使用Jedis提供的Java </font>API<font style="color:rgb(77, 77, 77);">对Redis进行操作，是Redis官方推崇的方式；</font>jedis使用起来比较简单，它的操作方法与redis命令相类似

## 1、Jedis所需要的jar包


```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>compile</scope>
</dependency>
```



## 2、连接Redis注意事项


redis.conf中注释掉bind 127.0.0.1，然后 protected-mode no



禁用Linux的防火墙：Linux(CentOS7)里执行命令



```plain
systemctl stop/disable firewalld.service
```



## 3、创建测试程序


```java
public static void main(String[] args) {
    //创建Jedis对象
    Jedis jedis = new Jedis("192.168.199.155", 6379);
    jedis.auth("******");
    //测试
    String ping = jedis.ping();
    System.out.println("连接成功：" + ping);
    jedis.close();
}
```



## 4、测试相关数据类型


### 4.1、Jedis-Api：key


```java
//操作key
@Test
public void demo1() {
    //创建Jedis对象
    Jedis jedis = new Jedis("192.168.199.155", 6379);
    jedis.auth("321612");
    Set<String> keys = jedis.keys("*");

    System.out.println(keys.size());

    for (String key : keys) {
        System.out.println(key);
    }

    System.out.println(jedis.exists("k1"));
    System.out.println(jedis.ttl("k1"));
    jedis.close();
}
```



### 4.2、Jedis-API:  String


```java
@Test
public void demo2() {
    //创建Jedis对象
    Jedis jedis = new Jedis("192.168.199.155", 6379);
    jedis.auth("321612");
    //添加
    jedis.set("name", "lucy");

    //获取
    String name = jedis.get("name");
    System.out.println(name);

    //设置多个key-value
    jedis.mset("k1", "v1", "k2", "v2");
    List<String> mget = jedis.mget("k1", "k2");
    System.out.println(mget);
    jedis.close();
}
```



### 4.3、Jedis-API:  List  linkedlist格式。支持重复元素


```java
public void demo3() {
    //创建Jedis对象
    Jedis jedis = new Jedis("192.168.199.155", 6379);
    jedis.auth("321612");
    //添加
    jedis.lpush("key1", "lucy", "marry", "jack");

    //获取
    List<String> key1 = jedis.lrange("key1", 0, -1);
    for (String s : key1) {
        System.out.println(s);
    }
    jedis.close();
}
```



### 4.4、Jedis-API:  set  不允许重复元素


```java
@Test
public void demo4() {
    //创建Jedis对象
    Jedis jedis = new Jedis("192.168.199.155", 6379);
    jedis.auth("321612");
    //添加
    jedis.sadd("orders", "order01");
    jedis.sadd("orders", "order02", "order03", "order04");

    //获取
    Set<String> smembers = jedis.smembers("orders");

    for (String s : smembers) {
        System.out.println(s);
    }

    jedis.srem("orders", "order02");
    jedis.close();
}
```



### 4.5、Jedis-API:  hash   map格式


```java
@Test
public void demo5() {
    //创建Jedis对象
    Jedis jedis = new Jedis("192.168.199.155", 6379);
    jedis.auth("321612");

    //添加
    jedis.hset("users", "age", "20");

    //获取
    String hget = jedis.hget("users", "age");
    System.out.println(hget);


    Map<String,String> map = new HashMap<>();
    map.put("telphone","13810169999");
    map.put("address","atguigu");
    map.put("email","abc@163.com");
    jedis.hmset("hash2",map);
    List<String> result = jedis.hmget("hash2", "telphone","email");
    for (String element : result) {
        System.out.println(element);
    }
    jedis.close();
}
```



### 4.6、Jedis-API:  zset  不允许重复元素，且元素有顺序


```java
@Test
public void demo6() {
    //创建Jedis对象
    Jedis jedis = new Jedis("192.168.199.155", 6379);
    jedis.auth("321612");

    //添加
    jedis.zadd("china", 100d, "shanghai");

    //获取
    Set<String> china = jedis.zrange("china", 0, -1);
    for (String s : china) {
        System.out.println(s);
    }
    jedis.close();
}
```



## 5、案例


**完成一个手机验证码功能**



要求：



1、输入手机号，点击发送后随机生成6位数字码，2分钟有效



2、输入验证码，点击验证，返回成功或失败



3、每个手机号每天只能输入3次



```java
public class PhoneCode {

    public static void main(String[] args) {

        //模拟验证码发送
        verifyCode("15137234301");

        getRedisCode("15137234301","417895");
    }

    //3、验证码校验
    public static void getRedisCode(String phone, String code) {
        //创建Jedis对象
        Jedis jedis = new Jedis("192.168.199.155", 6379);
        jedis.auth("321612");

        //验证码key
        String codeKey = "verifyCode" + phone + ":code";
        String redisCode = jedis.get(codeKey);

        //判断
        if (redisCode.equals(code)) {
            System.out.println("成功");
        } else {
            System.out.println("失败");
        }

        jedis.close();
    }

    //2、每个手机每天只能发送三次，验证码发到redis中，设置过期时间
    public static void verifyCode(String phone) {
        //创建Jedis对象
        Jedis jedis = new Jedis("192.168.199.155", 6379);
        jedis.auth("321612");

        //拼接key
        //手机发送次数key
        String countKey = "verifyCode" + phone + ":count";
        //验证码key
        String codeKey = "verifyCode" + phone + ":code";

        //每个手机每天只能发送三次
        String count = jedis.get(countKey);
        if (count == null) {
            jedis.setex(countKey, 24 * 60 * 60, "1");
        } else if (Integer.parseInt(count) <= 2) {
            //发送次数+1
            jedis.incr(countKey);
        } else if (Integer.parseInt(count) > 2) {
            //发送三次，不能再发送
            System.out.println("今天发送次数已经超过3次");
            jedis.close();
            return;
        }

        //发送验证码放到redis里面
        String vcode = getCode();
        jedis.setex(codeKey, 120, vcode);
        jedis.close();
    }

    //1、生成6位数字验证码
    public static String getCode() {
        Random random = new Random();
        String code = "";
        for (int i = 0; i < 6; i++) {
            int rand = random.nextInt(10);
            code += rand;
        }

        return code;
    }

}
```



## 6、JedisPool


+  创建JedisPool连接池对象 
+ 调用方法 getResource()方法获取Jedis连接 



```java
@Test
public void test7() {

    //0 创建一个配置对象
    JedisPoolConfig config = new JedisPoolConfig();
    config.setMaxTotal(50);  //最大允许连接数
    config.setMaxIdle(10); //最大空闲连接

    //1、初始化JedisPool对象
    JedisPool jedisPool = new JedisPool(config, "192.168.199.155", 6379);

    //2、获取连接
    Jedis jedis = jedisPool.getResource();
    jedis.auth("321612");

    //3、使用
    jedis.set("username", "张三");
    System.out.println(jedis.get("username"));

    //4、关闭连接 归还到连接池中
    jedis.close();
}
```



### 连接池工具类


```properties
host=192.168.199.155
port=6379
maxTotal=50
maxIdle=10
```



```java
public class JedisPoolUtils {


    private static JedisPool jedisPool;


    static {
        //读取配置文件
        InputStream resourceAsStream = JedisPoolUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
        //创建Properties对象
        Properties properties = new Properties();
        //关联文件
        try {
            properties.load(resourceAsStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        //读取数据，设置到JedisPoolConfig中
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(Integer.parseInt(properties.getProperty("maxTotal")));
        config.setMaxIdle(Integer.parseInt(properties.getProperty("maxIdle")));

        //初始化JedisPool对象
        jedisPool = new JedisPool(config, properties.getProperty("host"), Integer.parseInt(properties.getProperty("port")));

    }


    /**
     * 获取连接
     *
     * @return
     */
    public static Jedis getJedis() {
        //获取连接
        Jedis jedis = jedisPool.getResource();
        jedis.auth("******");
        return jedis;
    }


}
```



注意：



+  使用redis缓存一些不经常发生变化的数据。 
+  数据库的数据一旦发生改变，则需要更新缓存。 
+  数据库的表执行 增删改的相关操作，需要将redis缓存数据清空，再次存入 
+  在service对应的增删改方法中，将redis数据删除。 



> 更新: 2022-08-11 16:20:16  
> 原文: <https://www.yuque.com/like321/qgn2qc/cg61ti>