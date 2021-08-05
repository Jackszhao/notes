## SpringBoot整合Ehcache

#### 1.简介

EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认CacheProvider。Ehcache是一种广泛使用的开源Java分布式缓存。主要面向通用缓存,Java EE和轻量级容器。它具有`内存`和`磁盘`存储，缓存加载器,缓存扩展,缓存异常处理程序,一个gzip缓存servlet过滤器,支持REST和SOAP api等特点

#### 2.对比redis

ehcache直接在jvm虚拟机中缓存，速度快，效率高；但是缓存共享麻烦，集群分布式应用不方便。
redis是通过socket访问到缓存服务，效率比ecache低，比数据库要快很多，
处理集群和分布式缓存方便，有成熟的方案。如果是单个应用或者对缓存访问要求很高的应用，用ehcache。如果是大型系统，存在缓存共享、分布式部署、缓存内容很大的，建议用redis。

#### 3.导入maven依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
   
    <dependency>
        <groupId>net.sf.ehcache</groupId>
        <artifactId>ehcache</artifactId>
        <version>2.10.6</version>
    </dependency>
</dependencies>

```

#### 4.添加Ehcahe配置文件

在 resources 目录下，添加 ehcache 的配置文件 ehcache.xml ，文件内容如下

```xml
<ehcache>
    <diskStore path="java.io.tmpdir/shiro-spring-sample"/>
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
    />
    <cache name="user"
            maxElementsInMemory="10000"
            eternal="true"
            overflowToDisk="true"
            diskPersistent="true"
            diskExpiryThreadIntervalSeconds="600"/>
</ehcache>
```

##### 配置含义：

name:缓存名称。
maxElementsInMemory：缓存最大个数。
eternal:对象是否永久有效，一但设置了，timeout将不起作用。
timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。
diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
maxElementsOnDisk：硬盘最大缓存个数。
diskPersistent：是否缓存虚拟机重启期数据。
diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
clearOnFlush：内存数量最大时是否清除。
diskStore 则表示临时缓存的硬盘目录。

#### 5.开启缓存

开启缓存的方式，也和 Redis 中一样，如下添加 @EnableCaching 依赖即可：

```java
@SpringBootApplication
@EnableCaching
public class EhcacheApplication {
    public static void main(String[] args) {
      SpringApplication.run(EhcacheApplication.class, args);
    }
}
```

#### 6.使用缓存

这里主要介绍缓存中几个核心的注解使用：

##### （1）@CacheConfig

这个注解在类上使用，用来描述该类中所有方法使用的缓存名称，当然也可以不使用该注解，直接在具体的缓存注解上配置名称，示例代码如下：

```java
@Service
@CacheConfig(cacheNames = "user")
public class UserService {
}
```

##### （2）@Cacheable

这个注解一般加在查询方法上，表示将一个方法的返回值缓存起来，默认情况下，缓存的 key 就是方法的参数，缓存的 value 就是方法的返回值。示例代码如下：

```java
@Cacheable(key = "#id")
public User getUserById(Integer id,String username) {
    System.out.println("getUserById");
    return getUserFromDBById(id);
}
```

当有多个参数时，默认就使用多个参数来做 key ，如果只需要其中某一个参数做 key ，则可以在 @Cacheable 注解中，通过 key 属性来指定 key ，如上代码就表示只使用 id 作为缓存的 key ，如果对 key 有复杂的要求，可以自定义 keyGenerator 。当然，Spring Cache 中提供了root对象，可以在不定义 keyGenerator 的情况下实现一些复杂的效果，root 对象有如下属性：


![](C:\Users\zhao\Pictures\aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvNzQ3ODEwLzIwMTkxMi83NDc4MTAtMjAxOTEyMTAxMDI0MjEzNzYtNjY3NjY0MzQ4LnBuZw.png)

也可以通过 keyGenerator 自定义 key ，方式如下：

```java
@Component
public class MyKeyGenerator implements KeyGenerator {
    @Override
    public Object generate(Object target, Method method, Object... params) {
        return method.getName()+Arrays.toString(params);
    }
}
```

然后在方法上使用该 keyGenerator ：

```java
@Cacheable(keyGenerator = "myKeyGenerator")
public User getUserById(Long id) {
    User user = new User();
    user.setId(id);
    user.setUsername("lisi");
    System.out.println(user);
    return user;
}
```

##### （3）@CachePut

这个注解一般加在更新方法上，当数据库中的数据更新后，缓存中的数据也要跟着更新，使用该注解，可以将方法的返回值自动更新到已经存在的 key 上，示例代码如下：

```java
@CachePut(key = "#user.id")
public User updateUserById(User user) {
    return user;
}
```

##### （4）@CacheEvict

这个注解一般加在删除方法上，当数据库中的数据删除后，相关的缓存数据也要自动清除，该注解在使用的时候也可以配置按照某种条件删除（ condition 属性）或者或者配置清除所有缓存（ allEntries 属性），示例代码如下：

```java
@CacheEvict()
public void deleteUserById(Integer id) {
    //在这里执行删除操作， 删除是去数据库中删除
}
```

