# Java8新特性Optional
Java8中对null值提供了一个操作类。

## 静态方法  
`Optional.of(T)`  返回Optional包裹的对象，如果传入的参数为null，将抛出空指针异常。  
`Optional.ofNullable(T)`  返回Optional包裹的对象，参数如果为空返回空的Optional对象。  
`Optional.empty()`  返回空的Optional对象。  

## 实例方法  
`isPresent()` 值是否存在，如果值为空返回false,否则返回true。  
`get()` 值存在返回值,否则抛出异常。  
`map()` 值存在的情况下执行自定义方法，传入参数是值，返回值可以是任意类型的对象；最终返回值会包装成Optional类。  
`flatMap()` map方法基本一致，不同的是，返回值类型只能是Optional类型。  
`filter()`  值存在的情况下执行自定义方法，传入参数是值，返回true或false。  
`ifPrensent()`  值存在的情况下执行自定义方法，传入参数是值，方法没有返回值。    
`orElse()` 值不存在时返回参数值，参数值必须是Optional的泛型同类或子类。  
`orElseGet()` 值不存在时执行自定义方法，返回值必须与Optional的泛型同类或子类。  
`orElseThrow()` 值不存在时执行自定义方法，返回值必须是异常类。  

##   orElse()和orElseGet()方法的区别  
测试代码：
```
private String newString(String str) {
  System.out.println("String method");
  return str;
}

@Test
public void test13() {
  System.out.println(Optional.ofNullable(null).orElse(newString("newString")));
  System.out.println(Optional.ofNullable(null).orElseGet(() -> newString("newString")));\

  System.out.println("==================================================================");

  String str="oldString";
  System.out.println(Optional.ofNullable(str).orElse(newString("newString")));
  System.out.println(Optional.ofNullable(str).orElseGet(() -> newString("newString")));
}
```  
运行结果：
```
String method
newString
String method
newString
========================
String method
oldString
oldString
```  
`orElse()`若参数是通过调用方法返回的，则值不管是否存在都会调用该方法，`orElseGet()`在值不存在时不会调用参数中的方法。在对优化性能时要考虑好用哪个方法。  
## Optional的链式调用（重要）  
有时候为了避免空指针，我们需要进行多次判断，如下：
```
@Test
public void test14() {
  Father father = new Father();
  father.setSon(new Son());
  father.getSon().setName("optional");

  String name = "default";
  if (father != null) {
    Son son = father.getSon();
    if (son != null) {
      name = son.getName();
    }
  }
  System.out.println(name);
}
```  
但是如果判断次数太多，代码就会变得冗余且不好阅读，我们可以利用Optional的链式调用进行优化，代码如下：
```
@Test
public void test15() {
  Father father = new Father();
  father.setSon(new Son());
  father.getSon().setName("optional");

  String name = Optional.ofNullable(father)
      .map(f -> f.getSon())
      .map(s -> s.getName())
      .orElse("default");
}
```  
这样代码就既容易写也容易阅读了。
