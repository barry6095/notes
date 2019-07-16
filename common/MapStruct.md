# MapStruct
## 介绍  
## 使用
1、引入依赖  
```
<properties>
    <mapstruct.version>1.2.0.Final</mapstruct.version>
</properties>

<dependencies>
    <dependency>
      <groupId>org.mapstruct</groupId>
      <artifactId>mapstruct-jdk8</artifactId>
      <version>${mapstruct.version}</version>
    </dependency>
    <dependency>
      <groupId>org.mapstruct</groupId>
      <artifactId>mapstruct-processor</artifactId>
      <version>${mapstruct.version}</version>
    </dependency>
</dependencies>
```
2、定义对象DO和DTO,或者VO和PO  
3、编写Map转换器接口  
以下是两个实例
### 基础映射
`PersonDO`转换为`PersonDTO`，其中`UserDO`是`PersonDO`的一个属性。  

#### `PersonDO`和`UserDO`:
```
@NoArgsConstructor
@AllArgsConstructor
@Data
public class PersonDO {
  private Long id;
  private String name;
  private String email;
  private Date birthday;
  private UserDO user;
}
```
```
@NoArgsConstructor
@AllArgsConstructor
@Data
public class UserDO {
  private Integer age;
}

```
#### `PersonDTO`:
```
@NoArgsConstructor
@AllArgsConstructor
@Data
public class PersonDTO {
  private Long id;
  private String name;
  private Integer age;//对应PersonDO.UserDO.age;
  private String email;
  private Date birth;//与DO里面的字段（birthDay）不一样
  private String birthDateFormat;//对DO里面的字段（birthDay）进行拓展，dateFormat的形式
  private String birthExpressionFormat;//对 DO 里面的字段(birthDay)进行拓展,expression 的形式
}
```
#### 转换器接口`PersonConverter`:
```
@Mapper
public interface PersonConverter {

  PersonConverter INSTANCE = Mappers.getMapper(PersonConverter.class);

  @Mappings({
      @Mapping(source = "birthday", target = "birth"),
      @Mapping(source = "birthday", target = "birthDateFormat", dateFormat = "yyyy-MM-dd HH:mm:ss"),
      @Mapping(target = "birthExpressionFormat", expression = "java(org.apache.commons.lang3.time.DateFormatUtils.format(personDO.getBirthday(),\"yyyy-MM-dd HH:mm:ss\"))"),
      @Mapping(source = "user.age", target = "age"),
      @Mapping(target = "email", ignore = true)
  })
  PersonDTO personDO2DTO(PersonDO personDO);

  List<PersonDTO> personDO2DTO(List<PersonDO> personDOs);
}
```
#### 测试方法  
```
@Test
public void test1(){
  PersonDO personDO = new PersonDO(1L,"bee","bee@agree.com.cn",new Date(),new UserDO(1));
  PersonDTO personDTO = PersonConverter.INSTANCE.personDO2DTO(personDO);
  System.out.println(personDO);
  System.out.println(personDTO);
  System.out.println("getId:"+personDO.getId().equals(personDTO.getId()));
  System.out.println("getName:"+personDO.getName().equals(personDTO.getName()));
  System.out.println("getBirth:"+personDO.getBirthday().equals(personDTO.getBirth()));
  String format = DateFormatUtils.format(personDTO.getBirth(), "yyyy-MM-dd HH:mm:ss");
  System.out.println("getBirthDateFormat:"+format.equals(personDTO.getBirthDateFormat()));
  System.out.println("getBirthExpressionFormat:"+format.equals(personDTO.getBirthExpressionFormat()));


  List<PersonDO> personDOs = new ArrayList<>();
  personDOs.add(personDO);
  List<PersonDTO> personDTOs = PersonConverter.INSTANCE.personDO2DTO(personDOs);
  System.out.println(personDTOs);
}
```
### 多转一  
`BoyDO`和`GirlDO`转换为`CoupleDTO`。  

#### `BoyDO`和`GirlDO`:
```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class BoyDO {
  private String name;
  private int age;
}
```
```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class GirlDO {
  private String name;
  private int age;
}
```
#### `CoupleDTO`:  
```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class CoupleDTO {
  String boyName;
  String girlName;
  String coupleName;
  Date date;
}
```
#### 转换器`CoupleConverter`:
```
@Mapper
public interface CoupleConverter {

  CoupleConverter INSTANCE = Mappers.getMapper(CoupleConverter.class);

  @Mappings({
      @Mapping(source = "boyDO.name",target = "boyName"),
      @Mapping(source ="girlDO.name",target ="girlName"),
      @Mapping(target = "coupleName",expression = "java(boyDO.getName()+\"-\"+girlDO.getName())"),
      @Mapping(target = "date",expression = "java(new java.util.Date())")
  })
  CoupleDTO toCouple(BoyDO boyDO,GirlDO girlDO);
}
```  
#### 测试方法  
```
@Test
public void test2(){
  BoyDO boyDO=new BoyDO("小明",22);
  GirlDO girlDO=new GirlDO("小红",20);
  CoupleDTO coupleDTO= CoupleConverter.INSTANCE.toCouple(boyDO,girlDO);
  System.out.println(boyDO);
  System.out.println(girlDO);
  System.out.println(coupleDTO);
}
```
### 直接映射/更新
在某些情况下，您需要映射，它不创建目标类型的新实例，而是更新该类型的现有实例。可以通过为目标对象添加一个参数并使用`@MappingTarget`标记该参数来实现这种映射（相同名称属性进行映射）。
```
void boy2Girl(BoyDO boyDO,@MappingTarget GirlDO girlDO);
```  
### 更多用法
官方文档：http://mapstruct.org/documentation/stable/reference/html/
## Spring注入方式  
以上使用的示例都是默认方式:
```
PersonConverter INSTANCE = Mappers.getMapper(PersonConverter.class);
CoupleConverter INSTANCE = Mappers.getMapper(CoupleConverter.class);
```  
还有一种常用方式，就是注入到Spring：  
1、给`@Mapper`添加属性：`componentModel = "spring" `  
```
@Mapper(componentModel = "spring")
public interface CoupleConverter {

  CoupleConverter INSTANCE = Mappers.getMapper(CoupleConverter.class);

  @Mappings({
      @Mapping(source = "boyDO.name",target = "boyName"),
      @Mapping(source ="girlDO.name",target ="girlName"),
      @Mapping(target = "coupleName",expression = "java(boyDO.getName()+\"-\"+girlDO.getName())"),
      @Mapping(target = "date",expression = "java(new java.util.Date())")
  })
  CoupleDTO toCouple(BoyDO boyDO,GirlDO girlDO);
}
```
2、使用@Autowired装配转换器  
3、即可直接调用装配器  
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class MapStructTest {

  @Autowired
  CoupleConverter coupleConverter;

  @Test
  public void testSpring(){
    BoyDO boyDO=new BoyDO("小明",22);
    GirlDO girlDO=new GirlDO("小红",20);
    CoupleDTO coupleDTO=coupleConverter.toCouple(boyDO,girlDO);
    System.out.println(boyDO);
    System.out.println(girlDO);
    System.out.println(coupleDTO);
  }

}
```  
## 常用注解和属性  
