# JsonPath  
JSONPath - 用于JSON的XPath
## 示例表达式
`$.store.book[0].title`或者`$['store']['book'][0]['title']`,推荐使用前者。
## 可选
1. com.jayway.jsonpath
2. 阿里的fastJson自带  

## com.jayway.jsonpath  
### 依赖  
```
<dependency>
  <groupId>com.jayway.jsonpath</groupId>
  <artifactId>json-path</artifactId>
  <version>2.4.0</version>
</dependency>
```
### 语法  
JsonPath|描述
-|-
$|根节点
@|现行节点
. or []|取子节点
..|不管位置，选择所有符合条件的节点
\* |匹配所有元素节点
[]|迭代器标示（可以在里边做简单的迭代操作，如数组下标，根据内容选值等 ）
[,]|支持迭代器中做多选
[start:end :step]|数组切片运算符
?()|支持过滤操作
()|支持表达式计算  

## 阿里的fastJson自带
### 依赖
```
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.58</version>
</dependency>
```
### 语法  
JSONPath|描述
-|-
$|根对象
.key/['key']|取子对象
..key|不管位置，选择所有符合条件的节点
key.* |去所有子对象
[num]|索引访问，可为负数
[num0,num1,num2…]|多个索引访问，可为负数
[start:end]|索引范围访问，可为负数
[start:end :step]|索引范围访问，除了步长step外，start和end可为负数
[?(key)]|对象属性非空过滤
[key > 123]|数值对象属性比较过滤，支持符号=,!=,>,>=,<,<=
[key = '123']|字符串对象属性比较过滤
[key like 'aa%'']|字符串匹配过滤，通配符支持%，支持not like
[key rlike ‘regexpr’]|字符串正则匹配过滤，正则语法为jdk的正则语法，支持not like
[key in ('v0', 'v1')]|in过滤，支持数值和字符串
[key between 234 and 456]|between过滤，支持数值，支持not between
length() / size()|数组长度

## 两者差异
其实两者语法的大同小异,主要不同在`[start:end:step]`的操作上，阿里的是包括索引end的，另一个是不包括的。
因为只是工具类所以没有进一步验证其他差异，使用的时候得多留意并且测试。
## 示例参考
https://www.cnblogs.com/angle6-liu/p/10580792.html  
https://github.com/alibaba/fastjson/wiki/JSONPath
