# Redis
## 如何使用
1、引入依赖包
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```
2、添加配置文件
```
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制） 默认 8
spring.redis.lettuce.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
spring.redis.lettuce.pool.max-wait=-1
# 连接池中的最大空闲连接 默认 8
spring.redis.lettuce.pool.max-idle=8
# 连接池中的最小空闲连接 默认 0
spring.redis.lettuce.pool.min-idle=0
```
3、使用测试
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisTest {

  @Autowired
  private StringRedisTemplate stringRedisTemplate;

  @Autowired
  private RedisTemplate redisTemplate;

  @Test
  public void test() {
    stringRedisTemplate.opsForValue().set("aaa", "111");
    System.out.println(stringRedisTemplate.opsForValue().get("aaa"));
  }

  @Test
  public void testObj() throws InterruptedException {
    User user = new User();
    user.setUserName("aaa");
    user.setPassword("111");
    ValueOperations<String, User> operations = redisTemplate.opsForValue();
    operations.set("test.user", user);
    operations.set("test.user.x", user, 1, TimeUnit.SECONDS);
    Thread.sleep(1000);

    System.out.println(redisTemplate.opsForValue().get("test.user"));

    System.out.println(redisTemplate.hasKey("test.user.x"));
  }
}

```
## 使用缓存
1、添加Redis的配置类，生成指定Key
```
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport{

  @Bean
  public KeyGenerator keyGenerator(){
    return new KeyGenerator() {
      @Override
      public Object generate(Object o, Method method, Object... objects) {
        StringBuilder sb=new StringBuilder();
        sb.append(o.getClass().getName());
        sb.append(".");
        sb.append(method.getName());
        for (Object object : objects) {
          sb.append(".");
          sb.append(object.toString());
        }
        System.out.println("KeyGenerator:"+sb.toString());
        return sb.toString();
      }
    };
  }

}
```
注意要用 `@EnableCaching` 来开启缓存

2、自动根据方法生成缓存
```
@RestController
public class UserController {
  @RequestMapping("/getCacheUser/{param}")
  @Cacheable(value = "cache-key" ,key = "#param")
  public User getCacheUser(@PathVariable String param){
    User user = new User();
    user.setUserName("bbb");
    user.setPassword("222");
    System.out.println("控制台没有出现这条字符串，表示没有进入这个方法啊，直接返回缓存中对象");
    return user;
  }
}
```
配置类中返回 `KeyGenerator ` 的方法只是声明了key的生成策略，
需要在方法或者类添加相关开启缓存注解才能生效  
可使用的注解如下：
>`@Cacheable`：调用方法的时候时查看是否已有缓存，
有则直接从缓存取出且不进入方法，无则调用方法并把返回值保存到缓存</br>
>`@CachePut`：每次调用该方法都会把返回值保存到缓存</br>
>`@CacheEvict`：调用这个方法后清楚缓存</br>
>`@Caching`：可以指定多个相关注解，其拥有三个属性：`cacheable`、`put`和`evict`，分别用于指定`@CachePut`、`@CachePut`和`@CacheEvict` </br>

部分参数介绍：
>`value`:指定缓存名称  
>`key`:生成指定的key，支持SpringEL。没有指定时使用默认策略生成Key  
>`keyGenerator`:指定Key的生成策略  
>`condition`:指定使用缓存处理的条件，通过SpringEL表达式了判断  
>`allEntries`: `@CacheEvict`的属性，表示是否忽略Key清除缓存中所有元素  
>`key` 和 `keyGenerator` 不能同时使用  

Spring还为我们提供了一个root对象可以用来生成Key：

属性名称|描述|示例
-|-|-
methodName|当前方法名|#root.methodName
method|当前方法|#root.method.name
target|当前被调用的对象|#root.target
targetClass|当前被调用的对象的class|#root.targetClass
args|当前方法参数组成的数组|#root.args[0]
caches|当前被调用的方法使用的Cache|#root.caches[0].name

注意：生成的**缓存到Redis的Key**为 **value::key**

## 共享Session  
分布式系统，Session共享有很多的方案，其中托管到缓存中应该是最常用的方案之一  
### Spring Session官方说明  
Spring Session 提供了一套创建和管理 Servlet HttpSession 的方案。Spring Session 提供了集群 Session（Clustered Sessions）功能，默认采用外置的 Redis 来存储 Session 数据，以此来解决 Session 共享的问题。  
### Spring Session的使用  
1、引入依赖包  
```
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```  
2、添加Session配置类
```
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 180)
public class SessionConfig {
}
```  
要添加 `@EnableRedisHttpSession`来开启共享Session  
注意：
>`maxInactiveIntervalInSeconds`: 设置 Session 失效时间，使用 Redis Session 之后，原 Spring Boot 的 `server.session.timeout` 属性不再生效。  

3、使用测试  
添加方法获取Session
```
@RestController
public class HelloWorldController {
  @RequestMapping("/uid")
  public String uid(HttpSession session){
    return session.getId();
  }
}
```  
登录Redis输入 `keys *session*`  
```
1) "spring:session:sessions:c108f103-5ecd-4ce3-9ce2-0000821bd2d2"
2) "spring:session:sessions:expires:c108f103-5ecd-4ce3-9ce2-0000821bd2d2"
3) "spring:session:expirations:1562659140000"
```  
就能看到缓存的Session信息，访问 http://localhost:8080/uid 获取sessionId跟缓存里的一致，说明Session已经缓存到Redis里面进行有效的管理了  
### 在两个或多个项目中共享Session  
按照以上步骤在另一个项目中再次配置一次，启动后自动高就进行了Session共享
