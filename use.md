# JavaPoet使用指南

-   [JavaPoet使用指南-掘金](https://juejin.cn/post/6844903456629587976)



## 0x00 概述



注解系列



-   [注解基础](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU5OTgzNzM0NA%3D%3D%26mid%3D100000012%26idx%3D1%26sn%3Dea419145d678c6a39c3981cb8512b219%26scene%3D19%26token%3D1145503795%26lang%3Dzh_CN%23wechat_redirect)
-   [JavaPoet](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU5OTgzNzM0NA%3D%3D%26mid%3D100000024%26idx%3D1%26sn%3D118f2761faee835af670d52922a69228%26scene%3D19%26token%3D1145503795%26lang%3Dzh_CN%23wechat_redirect)
-   [编译期注解处理之 APT](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU5OTgzNzM0NA%3D%3D%26mid%3D100000019%26idx%3D1%26sn%3D035fc80b9c8128ea10a11b5759f70a21%26scene%3D19%26token%3D1145503795%26lang%3Dzh_CN%23wechat_redirect)



上一篇限于篇幅只介绍了 APT，这篇来继续介绍 [javapoet](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fsquare%2Fjavapoet)，是 square 公司的开源库。正如其名，java 诗人，通过注解来生成 java 源文件，通常要使用 javapoet 这个库与 Filer 配合使用。主要和注解配合用来干掉那些重复的模板代码 (如 butterknife 和 databinding 所做的事情)，当然你也可以使用这个技术让你的代码更加的炫酷。



## 0x01 简单使用



使用之前要先引入这个库



```
compile 'com.squareup:javapoet:1.7.0'
复制代码
```

javapoet 是用来生成代码的，需要借助



常用类



使用 javapoet 前需要了解 8 个常用类



| 类名           | 作用                                         |
| -------------- | -------------------------------------------- |
| MethodSpec     | 代表一个构造函数或方法声明                   |
| TypeSpec       | 代表一个类，接口，或者枚举声明               |
| FieldSpec      | 代表一个成员变量，一个字段声明               |
| JavaFile       | 包含一个顶级类的 Java 文件                   |
| ParameterSpec  | 用来创建参数                                 |
| AnnotationSpec | 用来创建注解                                 |
| ClassName      | 用来包装一个类                               |
| TypeName       | 类型，如在添加返回值类型是使用 TypeName.VOID |



除此之外 JavaPoet 提供了一套自定义的字符串格式化规则，常用的有

![img](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202204071439605)

 S,

![img](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202204071439811)

N





| 格式化规则 | 表示含义 |
| ---------- | -------- |
| $L         | 字面量   |
| $S         | 字符串   |
| $T         | 类、接口 |
| $N         | 变量     |



## 0x02 使用进阶



下面由浅入深，循序渐进的说明用法



方法 & 控制流：



-   添加方法 `addcode` 和 `addstatement` 对与无需类引入的极简代码可以直接使用`addCode`



```
MethodSpec main = MethodSpec.methodBuilder("main")
    .addCode(""
        + "int total = 0;\n"
        + "for (int i = 0; i < 10; i++) {\n"
        + "  total += i;\n"
        + "}\n")
    .build();
复制代码
```

生成的是



```
void main() {
  int total = 0;
  for (int i = 0; i < 10; i++) {
    total += i;
  }
}
复制代码
```

要是需要 import 的方法，如上面的`.addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")` 就需要使用`.addStatement`来声明



-   更优雅的流控制



`beginControlFlow` 流开启 `addStatement` 处理语句 `endControlFlow()`流结束



如上面的用流改写就是



```
MethodSpec main = MethodSpec.methodBuilder("main")
    .addStatement("int total = 0")
    .beginControlFlow("for (int i = 0; i < 10; i++)")
    .addStatement("total += i")
    .endControlFlow()
    .build();
复制代码
```

占位符



javapoet 里面提供了占位符来帮助我们更好地生成代码



-   $L 字面常量（Literals）



```
private MethodSpec computeRange(String name, int from, int to, String op) {
  return MethodSpec.methodBuilder(name)
      .returns(int.class)
      .addStatement("int result = 0")
      .beginControlFlow("for (int i = $L; i < $L; i++)", from, to)
      .addStatement("result = result $L i", op)
      .endControlFlow()
      .addStatement("return result")
      .build();
}
复制代码
```

这个就是一个 for 循环，op 负责加减乘除等符号



-   $S 字符串常量（String）
-   $T 类型 (Types)



```
最大的特点是自动导入包
```



```
MethodSpec today = MethodSpec.methodBuilder("today")
    .returns(Date.class)
    .addStatement("return new $T()", Date.class)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(today)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);

复制代码
```

生成的代码如下，而且会自动导包



```
package com.example.helloworld;

import java.util.Date;

public final class HelloWorld {
  Date today() {
    return new Date();
  }
}
复制代码
```

-   $N 命名 (Names), 通常指我们自己生成的方法名或者变量名等等



比如这样的代码块



```
public String byteToHex(int b) {
  char[] result = new char[2];
  result[0] = hexDigit((b >>> 4) & 0xf);
  result[1] = hexDigit(b & 0xf);
  return new String(result);
}

public char hexDigit(int i) {
  return (char) (i < 10 ? i + '0' : i - 10 + 'a');
}
复制代码
```

我们可以传递`hexDigit()`来代替。



```
MethodSpec hexDigit = MethodSpec.methodBuilder("hexDigit")
    .addParameter(int.class, "i")
    .returns(char.class)
    .addStatement("return (char) (i < 10 ? i + '0' : i - 10 + 'a')")
    .build();

MethodSpec byteToHex = MethodSpec.methodBuilder("byteToHex")
    .addParameter(int.class, "b")
    .returns(String.class)
    .addStatement("char[] result = new char[2]")
    .addStatement("result[0] = $N((b >>> 4) & 0xf)", hexDigit)
    .addStatement("result[1] = $N(b & 0xf)", hexDigit)
    .addStatement("return new String(result)")
    .build();
复制代码
```

获取对应类



有两种方式:



-   ClassName.bestGuess(“类全名称”) 返回 ClassName 对象，这里的类全名称表示的类必须要存在，会自动导入相应的包

-   ClassName.get(“包名”，” 类名”) 返回 ClassName 对象，不检查该类是否存在

    因此如果使用 JavaPoet 的话后续代码重构改变类名往往需要格外注意一点



```
ClassName hoverboard = ClassName.get("com.mattel", "Hoverboard");
ClassName list = ClassName.get("java.util", "List");
ClassName arrayList = ClassName.get("java.util", "ArrayList");
TypeName listOfHoverboards = ParameterizedTypeName.get(list, hoverboard);

MethodSpec beyond = MethodSpec.methodBuilder("beyond")
    .returns(listOfHoverboards)
    .addStatement("$T result = new $T<>()", listOfHoverboards, arrayList)
    .addStatement("result.add(new $T())", hoverboard)
    .addStatement("result.add(new $T())", hoverboard)
    .addStatement("result.add(new $T())", hoverboard)
    .addStatement("return result")
    .build();
复制代码
```

然后生成



```
package com.example.helloworld;

import com.mattel.Hoverboard;
import java.util.ArrayList;
import java.util.List;

public final class HelloWorld {
  List<Hoverboard> beyond() {
    List<Hoverboard> result = new ArrayList<>();
    result.add(new Hoverboard());
    result.add(new Hoverboard());
    result.add(new Hoverboard());
    return result;
  }
}
复制代码
```

构建类的元素



-   方法



方法的修饰，如`Modifiers.ABSTRACT`



```
MethodSpec flux = MethodSpec.methodBuilder("flux")
    .addModifiers(Modifier.ABSTRACT, Modifier.PROTECTED)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addMethod(flux)
    .build();
复制代码
```

这将会生成如下代码



```
public abstract class HelloWorld {
  protected abstract void flux();
}

复制代码
```

当然 Methods 需要和`MethodSpec.Builder`配置来增加方法参数、异常、javadoc、注解等。



-   构造器



这个其实也是个函数方法而已，因此可以使用 MethodSpec 来生成构造器方法。比如：



```
MethodSpec flux = MethodSpec.constructorBuilder()
    .addModifiers(Modifier.PUBLIC)
    .addParameter(String.class, "greeting")
    .addStatement("this.$N = $N", "greeting", "greeting")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(String.class, "greeting", Modifier.PRIVATE, Modifier.FINAL)
    .addMethod(flux)
    .build();
复制代码
```

将会生成



```
public class HelloWorld {
  private final String greeting;

  public HelloWorld(String greeting) {
    this.greeting = greeting;
  }
}
复制代码
```

-   参数 (重要)



之前我们是通过`addstatement`直接设置参数，其实参数也有自己的一个专用类`ParameterSpec`，我们可以使用`ParameterSpec.builder()`来生成参数，然后 MethodSpec 的 addParameter 去使用，这样更加优雅。



```
ParameterSpec android = ParameterSpec.builder(String.class, "android")
    .addModifiers(Modifier.FINAL)
    .build();

MethodSpec welcomeOverlords = MethodSpec.methodBuilder("welcomeOverlords")
    .addParameter(android)
    .addParameter(String.class, "robot", Modifier.FINAL)
    .build();
复制代码
```

生成的代码



```
void welcomeOverlords(final String android, final String robot) {
}
复制代码
```

稍微复杂点的类型 比如泛型 、Map 之类的，需要了解下 JavaPoet 定义的几种专门描述类型的类



![img](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202204071439296.awebp)



常见的有



| 分类                                   | 生成的类型       | JavaPoet 写法                                                | 也可以这么写 （等效的 Java 写法） |
| -------------------------------------- | ---------------- | ------------------------------------------------------------ | --------------------------------- |
| 内置类型                               | int              | TypeName.INT                                                 | int.class                         |
| 数组类型                               | int[]            | ArrayTypeName.of(int.class)                                  | int[].class                       |
| 需要引入包名的类型                     | java.io.File     | ClassName.get(“[java.io](https://link.juejin.cn/?target=http%3A%2F%2Fjava.io)”, “File”) | java.io.File.class                |
| 参数化类型 （ParameterizedType         | List             | ParameterizedTypeName.get(List.class, String.class)          | -                                 |
| 类型变量 （WildcardType） 用于声明泛型 | T                | TypeVariableName.get(“T”)                                    | -                                 |
| 通配符类型                             | ? extends String | WildcardTypeName.subtypeOf(String.class)                     | -                                 |



```
/*
 *Build input type, format as :
 *Map<String, Class<? extends IRouteGroup>>
 */
    ParameterizedTypeName inputMapTypeOfRoot = ParameterizedTypeName.get(
            ClassName.get(Map.class),
            ClassName.get(String.class),
            ParameterizedTypeName.get(
                    ClassName.get(Class.class),
                    WildcardTypeName.subtypeOf(ClassName.get(type_IRouteGroup))
            )
    );

    /*
     *Map<String, RouteMeta>
     */
    ParameterizedTypeName inputMapTypeOfGroup = ParameterizedTypeName.get(
            ClassName.get(Map.class),
            ClassName.get(String.class),
            ClassName.get(RouteMeta.class)
    );        

    /*
     *Build input param name.
     */
    ParameterSpec rootParamSpec = ParameterSpec.builder(inputMapTypeOfRoot, "routes").build();
    ParameterSpec groupParamSpec = ParameterSpec.builder(inputMapTypeOfGroup, "atlas").build();
复制代码
```

生成参数类型



```
public class ARouter$$Root$$app implements IRouteRoot {
  @Override
  public void loadInto(Map<String, Class<? extends IRouteGroup>> routes) {
    routes.put("service", ARouter$$Group$$service.class);
    routes.put("test", ARouter$$Group$$test.class);
  }
}


public class ARouter$$Group$$service implements IRouteGroup {
  @Override
  public void loadInto(Map<String, RouteMeta> atlas) {
    atlas.put("/service/hello", RouteMeta.build(RouteType.PROVIDER, HelloServiceImpl.class, "/service/hello", "service", null, -1, -2147483648));
    atlas.put("/service/json", RouteMeta.build(RouteType.PROVIDER, JsonServiceImpl.class, "/service/json", "service", null, -1, -2147483648));
    atlas.put("/service/single", RouteMeta.build(RouteType.PROVIDER, SingleService.class, "/service/single", "service", null, -1, -2147483648));
  }
}

复制代码
```

-   字段



可以使用 FieldSpec 去声明字段，然后加到 Method 中处理



```
FieldSpec android = FieldSpec.builder(String.class, "android")
    .addModifiers(Modifier.PRIVATE, Modifier.FINAL)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(android)
    .addField(String.class, "robot", Modifier.PRIVATE, Modifier.FINAL)
    .build();
复制代码
```

然后生成代码



```
public class HelloWorld {
  private final String android;

  private final String robot;
}
复制代码
```

通常 Builder 可以更加详细的创建字段的内容，比如 javadoc、annotations 或者初始化字段参数等，如：



```
FieldSpec android = FieldSpec.builder(String.class, "android")
    .addModifiers(Modifier.PRIVATE, Modifier.FINAL)
    .initializer("$S + $L", "Lollipop v.", 5.0d)
    .build();
复制代码
```

对应生成的代码



```
private final String android = "Lollipop v." + 5.0;
复制代码
```

-   接口



接口方法必须是 PUBLIC ABSTRACT 并且接口字段必须是 PUBLIC STATIC FINAL ，使用`TypeSpec.interfaceBuilder`



如下



```
TypeSpec helloWorld = TypeSpec.interfaceBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(FieldSpec.builder(String.class, "ONLY_THING_THAT_IS_CONSTANT")
        .addModifiers(Modifier.PUBLIC, Modifier.STATIC, Modifier.FINAL)
        .initializer("$S", "change")
        .build())
    .addMethod(MethodSpec.methodBuilder("beep")
        .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
        .build())
    .build();
复制代码
```

生成的代码如下



```
public interface HelloWorld {
  String ONLY_THING_THAT_IS_CONSTANT = "change";

  void beep();
}
复制代码
```

-   继承父类 实现接口



接口代码



```
package com.test.javapoet;
public interface TestInterface<T> {
    void test(T testPara);
}
复制代码
```

父类代码



```
public class TestExtendesClass {

}
复制代码
```

使用 javapoet 实现接口并且继承父类



```
final ClassName  InterfaceName = ClassName.get("com.test.javapoet","TestInterface");


    ClassName superinterface = ClassName.bestGuess("com.test.javapoet.TestClass");
    //ClassName superinterface = ClassName.get("com.test.javapoet","aa");

    TypeSpec.Builder spec = TypeSpec.classBuilder("TestImpl")
            .addModifiers(Modifier.PUBLIC)
            // 添加接口，ParameterizedTypeName的参数1是接口，参数2是接口的泛型
            .addSuperinterface(ParameterizedTypeName.get(InterfaceName, superinterface)) 
            //使用ClassName.bestGuess会自动导入包
            .superclass(ClassName.bestGuess("com.zs.javapoet.test.TestExtendesClass"));


    MethodSpec.Builder methodSpec = MethodSpec.methodBuilder("test")
            .addAnnotation(Override.class)
            .returns(TypeName.VOID)
            .addParameter(superinterface, "testPara")
            .addStatement("System.out.println(hello)" );

        TypeSpec typeSpec = spec.addMethod(methodSpec.build()).build();

    JavaFile file = JavaFile.builder("com.zs.javapoet", typeSpec).build();
    file.writeTo(System.out);

复制代码
```

生成代码



```
package com.test.javapoet;

    import com.zs.javapoet.test.TestExtendesClass;
    import java.lang.Override;

    public class TestImpl extends TestExtendesClass implements TestInterface<TestClass> {
      @Override
      void test(TestClass testPara) {
        System.out.println(hello);
      }
    }
复制代码
```

-   枚举类型



使用`TypeSpec.enumBuilder`来创建，使用`addEnumConstant`来添加



```
TypeSpec helloWorld = TypeSpec.enumBuilder("Roshambo")
    .addModifiers(Modifier.PUBLIC)
    .addEnumConstant("ROCK")
    .addEnumConstant("SCISSORS")
    .addEnumConstant("PAPER")
    .build();
复制代码
```

生成的代码



```
public enum Roshambo {
  ROCK,

  SCISSORS,

  PAPER
}
复制代码
```

更复杂的类型也可以支持，如重写、注解等



```
TypeSpec helloWorld = TypeSpec.enumBuilder("Roshambo")
    .addModifiers(Modifier.PUBLIC)
    .addEnumConstant("ROCK", TypeSpec.anonymousClassBuilder("$S", "fist")
        .addMethod(MethodSpec.methodBuilder("toString")
            .addAnnotation(Override.class)
            .addModifiers(Modifier.PUBLIC)
            .addStatement("return $S", "avalanche!")
            .build())
        .build())
    .addEnumConstant("SCISSORS", TypeSpec.anonymousClassBuilder("$S", "peace")
        .build())
    .addEnumConstant("PAPER", TypeSpec.anonymousClassBuilder("$S", "flat")
        .build())
    .addField(String.class, "handsign", Modifier.PRIVATE, Modifier.FINAL)
    .addMethod(MethodSpec.constructorBuilder()
        .addParameter(String.class, "handsign")
        .addStatement("this.$N = $N", "handsign", "handsign")
        .build())
    .build();
复制代码
```

生成代码



```
public enum Roshambo {
  ROCK("fist") {
    @Override
    public void toString() {
      return "avalanche!";
    }
  },

  SCISSORS("peace"),

  PAPER("flat");

  private final String handsign;

  Roshambo(String handsign) {
    this.handsign = handsign;
  }
}
复制代码
```

-   匿名内部类



需要使用`Type.anonymousInnerClass("")`, 通常可以使用 $L 占位符来指代



```
TypeSpec comparator = TypeSpec.anonymousClassBuilder("")
    .addSuperinterface(ParameterizedTypeName.get(Comparator.class, String.class))
    .addMethod(MethodSpec.methodBuilder("compare")
        .addAnnotation(Override.class)
        .addModifiers(Modifier.PUBLIC)
        .addParameter(String.class, "a")
        .addParameter(String.class, "b")
        .returns(int.class)
        .addStatement("return $N.length() - $N.length()", "a", "b")
        .build())
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addMethod(MethodSpec.methodBuilder("sortByLength")
        .addParameter(ParameterizedTypeName.get(List.class, String.class), "strings")
        .addStatement("$T.sort($N, $L)", Collections.class, "strings", comparator)
        .build())
    .build();
复制代码
```

生成代码



```
void sortByLength(List<String> strings) {
  Collections.sort(strings, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
      return a.length() - b.length();
    }
  });
}
复制代码
```

定义匿名内部类的一个特别棘手的问题是参数的构造。在上面的代码中我们传递了不带参数的空字符串。TypeSpec.anonymousClassBuilder("")。



-   注解



注解使用起来比较简单



```
MethodSpec toString = MethodSpec.methodBuilder("toString")
    .addAnnotation(Override.class)
    .returns(String.class)
    .addModifiers(Modifier.PUBLIC)
    .addStatement("return $S", "Hoverboard")
    .build();
复制代码
```

生成代码



```
  @Override
  public String toString() {
    return "Hoverboard";
  }
复制代码
```

通过`AnnotationSpec.builder()` 可以对注解设置属性：



```
MethodSpec logRecord = MethodSpec.methodBuilder("recordEvent")
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addAnnotation(AnnotationSpec.builder(Headers.class)
        .addMember("accept", "$S", "application/json; charset=utf-8")
        .addMember("userAgent", "$S", "Square Cash")
        .build())
    .addParameter(LogRecord.class, "logRecord")
    .returns(LogReceipt.class)
    .build();
复制代码
```

代码生成如下



```
@Headers(
    accept = "application/json; charset=utf-8",
    userAgent = "Square Cash"
)
LogReceipt recordEvent(LogRecord logRecord);
复制代码
```

注解同样可以注解其他注解，通过 $L 引用如



```
MethodSpec logRecord = MethodSpec.methodBuilder("recordEvent")
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addAnnotation(AnnotationSpec.builder(HeaderList.class)
        .addMember("value", "$L", AnnotationSpec.builder(Header.class)
            .addMember("name", "$S", "Accept")
            .addMember("value", "$S", "application/json; charset=utf-8")
            .build())
        .addMember("value", "$L", AnnotationSpec.builder(Header.class)
            .addMember("name", "$S", "User-Agent")
            .addMember("value", "$S", "Square Cash")
            .build())
        .build())
    .addParameter(LogRecord.class, "logRecord")
    .returns(LogReceipt.class)
    .build();
复制代码
```

生成代码



```
@HeaderList({
    @Header(name = "Accept", value = "application/json; charset=utf-8"),
    @Header(name = "User-Agent", value = "Square Cash")
})
LogReceipt recordEvent(LogRecord logRecord);
复制代码
```

注释

-   javadoc


## 0x03 后续



在 javapoet 之前有 javawriter，但 javapoet 有着更强大的代码模型，并且对类的理解更加到位，因此推荐使用 javapoet



## 参考文章



-   [juejin.cn/post/684490…](https://juejin.cn/post/6844903475776585741)
-   [cloud.tencent.com/developer/a…](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1006210)
-   [blog.csdn.net/qq_26376637…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fqq_26376637%2Farticle%2Fdetails%2F52374063)
