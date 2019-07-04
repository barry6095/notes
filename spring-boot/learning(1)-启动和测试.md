# 快速入门
1、访问 http://start.spring.io/  
2、选择构建工具Maven Project、Java、Spring Boot以及填写一些工程基本信息，可参考下图：![](assets/learning(1)-启动和测试-c9ef57de.png)
3、点击 Generate Project 下载项目压缩包  
4、解压后导入项目：  
- **IDEA:**File -> New -> Model from Existing Source.. -> 选择解压后的文件夹 -> OK，选择 Maven 然后一路 Next和OK即可。
- **Eclipse:**Import -> Existing Maven Projects -> Next -> 选择解压后的文件夹 -> Finsh
- **IDEA集成的工具:**  
1、选择 File -> New -> Project… 弹出新建项目的框  
2、选择 Spring Initializr -> Next 也会出现上述类似的配置界面  
3、填写相关内容，Next -> 选择需要依赖的包，Next -> 确认信息无误，Finish  
## 项目结构介绍
![](assets/learning(1)-启动和测试-c36997d5.png)  
如上图所示，Spring Boot的基础结构共三个文件：
- `src/main/java` 程序开发以及主程序入口
- `src/main/resources` 配置文件
- `src/test/java` 测试程序  
Application.java建议放到根目录下面，主要用于做一些框架配置  
最后Application的main方法，至此一个Java项目搭建好了
## 默认依赖  
pom.xml中默认有两个模块:  
- `spring-boot-starter`:核心模块，包括自动配置支持、日志和YAML，web项目默认有 `spring-boot-starter-web` web模块， `spring-boot-starter-web` 自动依赖了 `spring-boot-starter`。
- `spring-boot-starter-test`:测试模块，包括JUnit、Hamcrest、Mockito。  
- web项目还会多一个 `spring-boot-starter-tomcat` 模块支持全栈式Web开发。
## 快速启动和测试  
1、编写Controller内容：
```
@RestController
public class HelloWorldController {

    @RequestMapping("/hello")
    public String sayHello() {
      return "Hello World!";
    }

    @RequestMapping("/hi")
    public String sayHi() {
      return "hi";
    }
}
```  
`@RestController` 的意思就是 Controller 里面的方法都以 json 格式输出，不用再写什么 jackjson 配置的了！  
2、启动主程序，打开浏览器访问 http://localhost:8080/hello ，就可以看到效果了。若引入了`spring-boot-starter-tomcat` 模块，需要把该模块的 `<scope>provided</scope>` 删除。  
3、单元测试  
打开`src/test/` 下的测试入口，编写简单的http请求来测试，使用MockMvc来测试。  
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloTest {

    private MockMvc mvc;

    @Before
    public void setUp() {
      mvc = MockMvcBuilders
        .standaloneSetup(new HelloWorldController())
        .build();
    }

    @Test
    public void getHello() throws Exception {
      mvc.perform(MockMvcRequestBuilders.get("hello")
          .accept(MediaType.APPLICATION_JSON))
          .andExpect(MockMvcResultMatchers.status()
          .isOk()).andDo(MockMvcResultHandlers.print())
          .andReturn();
    }
}
```  
## 开发环境调试
热启动在正常开发项目中已经很常见了，Spring Boot对调试支持很好，修改可以实时生效，需要修改一下配置：  
```
  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-devtools</artifactId>
          <optional>true</optional>
      </dependency>
  </dependencies>  

  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
              <configuration>
                  <fork>true</fork>
              </configuration>
          </plugin>
      </plugins>
  </build>
```  
该模块在完整的打包环境下运行的时候会被禁用。如果你使用 `java -jar`启动应用或者用一个特定的 classloader 启动，它会认为这是一个“生产环境”。  
 **IDEA设置**  
 当我们修改了Java类后，IDEA默认是不自动编译的，而`spring-boot-devtools`又是监测classpath下的文件发生变化才会重启应用，所以需要设置IDEA的自动编译：  
 1、File -> Settings... -> Build,Exception,Deployment -> Compiler -> 勾选Build project automatically  -> OK
 ![](assets/learning(1)-启动和测试-0549ecac.png)  
 2、ctrl + shift + alt + / -> Registry... -> 勾选 compiler.automake.allow.when.app.running -> Close
 ![](assets/learning(1)-启动和测试-e89bf12a.png  
 3、重启IDEA即可
