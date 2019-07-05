# Web开发综合
Spring Boot Web 开发非常的简单，其中包括常用的 json 输出、filters、property、log 等
## json接口开发
在以前使用spring开发项目，需要提供json接口时需要做哪些配置
> 1. 添加 `jackjson` 等相关 jar 包
> 2. 配置 Spring Controller 扫描
> 3. 对接的方法添加 `@ResponseBody`

现在使用Spring Boot只需在Controller类加上`@RestController`即可，这样默认类中的方法都会以json的格式返回
```
@RestController
public class UserController {
  @RequestMapping("/getUser")
  public User getUser() {
    User user = new User();
    user.setUserName("ming");
    user.setPassword("123");
    return user;
  }
}
```
## 自定义Filter
在项目中常常会使用 filters 用于录调用日志、排除有 XSS 威胁的字符、执行权限验证等等。Spring Boot 自动添加了 `OrderedCharacterEncodingFilter` 和 `HiddenHttpMethodFilter`，并且我们可以自定义 Filter。
### 方式1  
>1. 实现Filter接口，实现Filter方法
```
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
      System.out.println("===MyFilter init===");
    }
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
        FilterChain filterChain) throws IOException, ServletException {
      HttpServletRequest request = (HttpServletRequest) servletRequest;
      System.out.println("this is MyFilter,url :" + request.getRequestURI());
      filterChain.doFilter(servletRequest, servletResponse);
    }
    @Override
    public void destroy() {
      System.out.println("===MyFilter destroy===");
    }
}
```  
>2. 添加`configuration` 注解，将自定义Filter加入过滤链
```
@Configuration
public class WebConfiguration {
    @Bean
    public RemoteIpFilter remoteIpFilter() {
      return new RemoteIpFilter();
    }
    @Bean
    public FilterRegistrationBean testFilterRegistration() {
      FilterRegistrationBean registration = new FilterRegistrationBean();
      registration.setFilter(new MyFilter());
      registration.addUrlPatterns("/*");
      registration.addInitParameter("paramName", "paramValue");
      registration.setName("MyFilter");
      registration.setOrder(1);
      return registration;
    }
}
```  

### 方式2  
>1. 在入口Application类加上注解`@ServletComponentScan`
```
@SpringBootApplication
@ServletComponentScan
public class SpringbootDemoApplication {
    public static void main(String[] args) {
      SpringApplication.run(SpringbootDemoApplication.class, args);
    }
}
```  
>2. 实现Filter接口，实现Filter方法，添加`@WebFilter` 注解
```
@WebFilter(urlPatterns = "/*")
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
      System.out.println("===MyFilter init===");
    }
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
        FilterChain filterChain) throws IOException, ServletException {
      HttpServletRequest request = (HttpServletRequest) servletRequest;
      System.out.println("this is MyFilter,url :" + request.getRequestURI());
      filterChain.doFilter(servletRequest, servletResponse);
    }
    @Override
    public void destroy() {
      System.out.println("===MyFilter destroy===");
    }
}
```

在SpringBootApplication上使用 `@ServletComponentScan` 注解后，Servlet、Filter、Listener可以直接通过 `@WebServlet` 、 `@WebFilter` 、 `@WebListener` 注解自动注册，无需其他代码。
## 自定义properties
在 Web 开发的过程中，我经常需要自定义一些配置文件
配置在application.properties中
```
com.example.author=fd
com.example.number=6
```
自定义类使用配置
```
@Component
public class PropertiesUser {
  @Value("${com.example.author}")
  private String author;
  @Value("${com.example.number}")
  private int number;

  //省略setter和getter
}
```
### log配置
Spring Boot项目一般都会引用 `spring-boot-starter` 或者 `spring-boot-starter-web` ，而这两个起步依赖中都已经包含了对于 `spring-boot-starter-logging` 的依赖，所以，无需额外添加依赖。
直接在 `application.properties` 进行配置即可
## 数据库操作
一下是Mysql、spring data jpa的使用：
1、添加jar包
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```
2、添加配置
```
#数据库配置
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/spring-boot-demo?serverTimezone=CST
spring.datasource.username=root
spring.datasource.password=root123

#JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.show-sql=true
```
`hibernate.hbm2ddl.auto` 参数主要用于自动创建、更新、验证数据库表结构，有四个值:
> 1. `create`:启动时删数据库中的表，然后创建，退出时不删除数据表
> 2. `create-drop`:启动时删数据库中的表，然后创建，退出时删除数据表,如果表不存在报错
> 3. `update`:如果启动时表格式不一致则更新表，原有数据保留。**常用是这个。**
> 4. `validate`:项目启动表结构进行校验 如果不一致则报错

`database-platform` 参数用于指定生成表名的存储引擎为 InnoDBD
`show-sql` 是否打印出自动生成的 SQL，方便调试的时候查看
3、添加实体类和 Dao
```
@Entity
public class UserEntity implements Serializable {

  private static final long serialVersionUID = -8208996143845776688L;

  @Id
  @GeneratedValue
  private Long id;
  @Column(nullable = false, unique = true)
  private String userName;
  @Column(nullable = false)
  private String password;
  @Column(nullable = false, unique = true)
  private String email;
  @Column(nullable = false, unique = true)
  private String nickName;
  @Column(nullable = false)
  private String createTime;
  //省略getter和setter，构造方法。
}
```
dao 只要继承 JpaRepository 类就可以，几乎可以不用写方法。
可以根据方法名来自动的生成 SQL，比如`findByUserName` 会自动生成一个以 `userName`为参数的查询方法，比如 `findAll` 会自动会查询表里面的所有数据。
还可以通过 `@Modifying` 和 `@Query` 注解自定义SQL。
```
public interface UserDao extends JpaRepository<UserEntity, Long> {

  UserEntity findByUserName(String userName);

  List<UserEntity> findByUserNameOrEmail(String username, String email);

  Integer countByUserNameEqualsAndPasswordEquals(String username,String password);

  @Modifying
  @Query(value = "select * from user_entity ",nativeQuery = true)
  List<UserEntity> getAllUsingSql();
}
```
4、测试
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserDaoTest {

  @Autowired
  private UserDao userDao;

  @Test
  public void init(){
    Date date = new Date();
    DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG);
    String formattedDate = dateFormat.format(date);
    userDao.save(new UserEntity("aaa", "aaa123", "aaa@qq.com", "AAA", formattedDate));
    userDao.save(new UserEntity("bbb", "bbb123", "bbb@qq.com", "BBB", formattedDate));
    userDao.save(new UserEntity("ccc", "ccc123", "ccc@qq.com", "CCC", formattedDate));
  }

  @Test
  public void testDao() {
    System.out.println(userDao.findAll().size());
    System.out.println(userDao.findByUserNameOrEmail("bbb","ccc@qq.com"));
    System.out.println(userDao.countByUserNameEqualsAndPasswordEquals("aaa","aaa123"));
    System.out.println(userDao.getAllUsingSql());
  }
}
```
Spring Data Jpa 还有很多功能，比如封装好的分页，可以自己定义 SQL，主从分离等等  
