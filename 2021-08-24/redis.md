## redis框架使用

#### springboot整合redis

1.导入依赖

```xml
<!-- 引入 redis 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>1.5.7.RELEASE</version>
</dependency>
```

2、资源文件application.properties中增加Redis相关配置

```properties
# REDIS 配置
############################################################
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-wait=-1
spring.redis.jedis.pool.max-idle=10
spring.redis.jedis.pool.min-idle=2
spring.redis.timeout=6000
```

3、封装Json工具类

```java
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

/**
 * 
 * @Title: RedisOperator.java
 * @Package com.weiz.util
 * @Description: 使用redisTemplate的操作实现类 Copyright: Copyright (c) 2016
 * 
 * @author weiz
 * @date 2017年9月29日 下午2:25:03
 * @version V1.0
 */
@Component
public class RedisOperator {
   
// @Autowired
//    private RedisTemplate<String, Object> redisTemplate;
   
   @Autowired
   private StringRedisTemplate redisTemplate;
   
   // Key（键），简单的key-value操作

   /**
    * 实现命令：TTL key，以秒为单位，返回给定 key的剩余生存时间(TTL, time to live)。
    * 
    * @param key
    * @return
    */
   public long ttl(String key) {
      return redisTemplate.getExpire(key);
   }
   
   /**
    * 实现命令：expire 设置过期时间，单位秒
    * 
    * @param key
    * @return
    */
   public void expire(String key, long timeout) {
      redisTemplate.expire(key, timeout, TimeUnit.SECONDS);
   }
   
   /**
    * 实现命令：INCR key，增加key一次
    * 
    * @param key
    * @return
    */
   public long incr(String key, long delta) {
      return redisTemplate.opsForValue().increment(key, delta);
   }

   /**
    * 实现命令：KEYS pattern，查找所有符合给定模式 pattern的 key
    */
   public Set<String> keys(String pattern) {
      return redisTemplate.keys(pattern);
   }

   /**
    * 实现命令：DEL key，删除一个key
    * 
    * @param key
    */
   public void del(String key) {
      redisTemplate.delete(key);
   }

   // String（字符串）

   /**
    * 实现命令：SET key value，设置一个key-value（将字符串值 value关联到 key）
    * 
    * @param key
    * @param value
    */
   public void set(String key, String value) {
      redisTemplate.opsForValue().set(key, value);
   }

   /**
    * 实现命令：SET key value EX seconds，设置key-value和超时时间（秒）
    * 
    * @param key
    * @param value
    * @param timeout
    *            （以秒为单位）
    */
   public void set(String key, String value, long timeout) {
      redisTemplate.opsForValue().set(key, value, timeout, TimeUnit.SECONDS);
   }

   /**
    * 实现命令：GET key，返回 key所关联的字符串值。
    * 
    * @param key
    * @return value
    */
   public String get(String key) {
      return (String)redisTemplate.opsForValue().get(key);
   }

   // Hash（哈希表）

   /**
    * 实现命令：HSET key field value，将哈希表 key中的域 field的值设为 value
    * 
    * @param key
    * @param field
    * @param value
    */
   public void hset(String key, String field, Object value) {
      redisTemplate.opsForHash().put(key, field, value);
   }

   /**
    * 实现命令：HGET key field，返回哈希表 key中给定域 field的值
    * 
    * @param key
    * @param field
    * @return
    */
   public String hget(String key, String field) {
      return (String) redisTemplate.opsForHash().get(key, field);
   }

   /**
    * 实现命令：HDEL key field [field ...]，删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。
    * 
    * @param key
    * @param fields
    */
   public void hdel(String key, Object... fields) {
      redisTemplate.opsForHash().delete(key, fields);
   }

   /**
    * 实现命令：HGETALL key，返回哈希表 key中，所有的域和值。
    * 
    * @param key
    * @return
    */
   public Map<Object, Object> hgetall(String key) {
      return redisTemplate.opsForHash().entries(key);
   }

   // List（列表）

   /**
    * 实现命令：LPUSH key value，将一个值 value插入到列表 key的表头
    * 
    * @param key
    * @param value
    * @return 执行 LPUSH命令后，列表的长度。
    */
   public long lpush(String key, String value) {
      return redisTemplate.opsForList().leftPush(key, value);
   }

   /**
    * 实现命令：LPOP key，移除并返回列表 key的头元素。
    * 
    * @param key
    * @return 列表key的头元素。
    */
   public String lpop(String key) {
      return (String)redisTemplate.opsForList().leftPop(key);
   }

   /**
    * 实现命令：RPUSH key value，将一个值 value插入到列表 key的表尾(最右边)。
    * 
    * @param key
    * @param value
    * @return 执行 LPUSH命令后，列表的长度。
    */
   public long rpush(String key, String value) {
      return redisTemplate.opsForList().rightPush(key, value);
   }

}
```

4.测试

```java
@Resource
    public RedisUtil redisUtil;//导入工具
    @Test
    void contextLoads() {
        redisUtil.set("hello","world");
        Object hello = redisUtil.get("hello");
        System.out.println(hello);
    }
```

#### redis的事务（java实现）

1.代码实现

```java
  Jedis jedis = new Jedis("localhost",6379);
        jedis.auth("123456");
        Transaction transaction = jedis.multi();//开启事务
        transaction.lpush("key", "11");//向列表中插入数据
        transaction.lpush("key", "22");
        transaction.lpush("key", "33");
        String discard = transaction.discard();//取消事务
        System.out.println(discard);
        List<Object> key = redisUtil.lGet("key", 0, 8);
        for (Object o: key) {
            System.out.println(o.toString());
        }
```

2.jedis

**简介**

简单来说，Jedis就是Redis官方推荐的Java连接开发工具。

在Java中，Redis对应于Jedis就相当于关系数据库对应于JDBC。

**Maven依赖：**

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```

**使用Jedis连接Redis的简单示例**

```java
// 连接Redis（第一个参数是Redis的IP地址，第二个参数是Redis的端口号）
Jedis jedis = new Jedis("localhost", 6379);
jedis.auth("123456");//密码
// 尝试操作Redis
jedis.set("msg", "Hello World!");
String msg = jedis.get("msg");	
System.out.println(msg);	// 打印"Hello World!"
// 关闭Redis连接
jedis.close();
```

**注意：**redis是没有事务回滚的

**Jedis连接池**

Jedis提供了连接池机制，所以在生产环境中需要向Jedis连接池获取对Redis的连接。

Jedis的连接池类为redis.clients.jedis.JedisPool。

```java
JedisPoolConfig jpc = new JedisPoolConfig();
jpc.setMaxTotal(30);  // 设置连接池的最大连接数
jpc.setMaxIdle(8);    // 设置连接池允许的最大空闲连接数
// 初始化连接池类（使用自定义连接池参数）
JedisPool jp = new JedisPool(jpc, "localhost", 6379);
// 从连接池获取一个Jedis连接
Jedis jedis = jp.getResource();
```

**工具类代码**

```java
public class JedisUtils {
	private static JedisPool jp;
	
	static {
		// 加载Jedis连接池配置参数
		InputStream is = JedisUtils.class.getResourceAsStream("/application.properties");
		Properties prop = new Properties();
		try {
			prop.load(is);
		} catch (IOException e) {
			e.printStackTrace();
		}
		String host = prop.getProperty("jedis.host");
		int port = Integer.parseInt(prop.getProperty("jedis.port"));
		int maxTotal = Integer.parseInt(prop.getProperty("jedis.maxTotal"));
		int maxIdle = Integer.parseInt(prop.getProperty("jedis.maxIdle"));
		
		// 设置Jedis连接池参数
		JedisPoolConfig jpc = new JedisPoolConfig();
		jpc.setMaxTotal(maxTotal);
		jpc.setMaxIdle(maxIdle);
		
		// 初始化Jedis连接池
		jp = new JedisPool(jpc, host, port);
	}
	
	// 从Jedis连接池获取连接
	public static Jedis getJedis() {
		return jp.getResource();
	}
}
```

##### redis使用管道(Pipelining)提高查询速度

```java
Pipeline pipe = jedis.pipelined();
long start_pipe = System.currentTimeMillis();
for (int i = 0; i < 10000; i++) {
    pipe.set("pipe_" + i, String.valueOf(i));
}
pipe.sync(); // 获取所有的response
```

##### redis的发布与订阅

**1.发布方**

```java
// 生产环境千万不要这么使用哦，推荐使用 JedisPool 线程池的方式
    Jedis jedis = new Jedis("localhost", 6379);
    jedis.auth("123456");
    jedis.publish(channel, message);
```

**2.订阅方**

创建一个监听器

```java
import redis.clients.jedis.JedisPubSub;

public class MyListener extends JedisPubSub {
    @Override
    public void onMessage(String channel, String message) {
        System.out.println("收到订阅频道：" + channel + " 消息：" + message);
    }

    @Override
    public void onPMessage(String pattern, String channel, String message) {
        System.out.println("收到具体订阅频道：" + channel + "订阅模式：" + pattern + " 消息：" + message);
    }

}
```

把监听器放到要监听的通道

```java
// 生产环境千万不要这么使用哦，推荐使用 JedisPool 线程池的方式
        Jedis jedis = new Jedis("localhost", 6379);
        jedis.auth("123456");
        jedis.subscribe(new MyListener(), channel);
```

注意：发布的消息并没有持久性，监听之前和监听中断的消息将丢失

#### redis的持久化

**1.什么是持久化？为什么要持久化？**

redis之所以高效是因为数据存储在内存中，内存中数据会断电丢失， 所以需要持久化，持久化是将内存中的数据写到磁盘中

**2.redis持久化的方式：**

- RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照
- AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集

**3.RDB（redis database）**

redis将数据库快照保存在名为dump.rdb的二进制文件中，可以对redis进行设置，让他在“N秒内数据集至少有M个改动”时保存数据集。

例：60s内至少有1000个键被改动时，自动保存一次数据集

```
save 60 1000
```

**RDB优点：**

rdb文件紧凑，体积小，无IO阻塞（fork子进程来持久化数据），恢复速度比AOF快

**RDP缺点：**

实时性比较差（全量备份，备份频率比较低，数据丢失较多），当数据比较多时fork比较耗时

**4.AOF（append-only file）**

快照功能实时性不好，如果故障停机，服务器会丢失最近写入，且未保存到快照中的数据。

重1.1版本开始，redis增加了一种完全实时性的持久化方式：AOF持久化

通过配置文件打开AOF功能：

```
appendonly yes
```

**AOF配置选项**

- 每次有新命令追加到AOF文件就执行一次`fsync`：非常慢，也非常安全
- 每秒`fsync`一次：足够快（和使用RDB持久化差不多），并且故障只丢失1s中的数据
- 从不`fsync`：将数据交给操作系统来处理：更快，也更不安全

**AOF优点：**

实时性比RDB好（增量备份，备份频率比较高，数据丢失较少）

**AOF缺点：**

AOF文件大，恢复速度慢，对性能影响大