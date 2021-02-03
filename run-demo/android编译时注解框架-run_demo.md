## Android编译时注解框架-Run Demo

### 概述

先讲一下编写《Android编译时注解框架》的初衷吧，APT其实并不难，可以说是简单且高效，但关于APT的资料却并不多，甚至很多人都不知道这么一个技术。国内关于APT的博客屈指可数，唯二找到的几篇初级讲解一个是用Eclipse写得，一个是用AndroidStudio加Intellij。刚开始着实踩了不少坑，但事实是，APT完全可以用AndroidStudio单独实现。光是项目搭建就如此麻烦了，更别提语法讲解了。资料匮乏无疑提高了APT的入门门槛。

正因为如此，这个系列博客就这样诞生啦~现在就教你用AndroidStudio一步步打造自己的APT框架。


以我自己的学习习惯来讲，比起前期大量枯燥的基础知识积累，我更喜欢先把项目跑起来再说，虽然会不明所以，但反而会促进学习兴趣，并且在有结果的场景下一步步深入。

所以作为《Android编译时注解框架》系列的第二篇，我们不管三七二十一，先把APT跑起来再说，看看这到底是个什么东西。跑起来，就入门啦！

**在Running的过程中，有很多语法，我们都暂时一并跳过，都放到《Android编译时注解框架-语法讲解》统一讲。**


### Running

#### 项目搭建

首先创建一个Android项目

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9e63a8d6838458da1fa306c10dc0254~tplv-k3u1fbpfcp-watermark.image)


然后给我们的项目增加一个module,一定要记得是Java Library.

因为APT需要用到jdk下的 【 *javax.~ *】包下的类，这在AndroidSdk中是没有的。


![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e04ed06bdbfd4e51b16dfa8cd7c7ef48~tplv-k3u1fbpfcp-watermark.image)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3733835c84f9490b835904a4da37ce93~tplv-k3u1fbpfcp-watermark.image)


#### 自定义注解

新建一个类，GetMsg。就是我们自定义的注解。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8463f39061946a7a7a6ab9a062bad79~tplv-k3u1fbpfcp-watermark.image)

这是一个编译时注解，用*@Retention(RetentionPolicy.CLASS)*修饰。

这个注解只能修饰方法。用*@Target(ElementType.METHOD)*修饰。

且这个注解可以设置两个值。id和name。name是有默认值的，可以不设置。 


#### 创建Processor

Processor是用来处理Annotation的类。继承自AbstractProcessor。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d5420ceebf247dfac0ce900e17c8d24~tplv-k3u1fbpfcp-watermark.image)

复写AbstractProcessor两个最重要的方法。

*process*方法是用来处理注解的，我们一会写。

*getSupportedAnnotationTypes*用来表示该Processor处理哪些注解。这里我们只有一个*GetMsg*注解需要处理。


#### 重写process方法

我们的目的呢，是获取修饰了GetMsg注解的方法所有信息，只有获得了这些信息，才有依据生成代码不是吗?

```java
@Override
public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env) {
    Messager messager = processingEnv.getMessager();
    for (Element element : env.getElementsAnnotatedWith(GetMsg.class)) {
        PackageElement packageElement = (PackageElement) element
                .getEnclosingElement();
        //获取该注解所在类的包名
        String packageName = packageElement.getQualifiedName().toString();
        TypeElement classElement = (TypeElement) element;
        //获取该注解所在类的类名
        String className = classElement.getSimpleName().toString();
        //获取该注解所在类的全类名
        String fullClassName = classElement.getQualifiedName().toString();
        VariableElement variableElement = (VariableElement) element.getEnclosingElement();
        //获取方法名
        String methodName = variableElement.getSimpleName().toString();
        //获取该注解的值
        int id = classElement.getAnnotation(GetMsg.class).id();
        String name = classElement.getAnnotation(GetMsg.class).name();
        messager.printMessage(Diagnostic.Kind.NOTE,
                "Annotation class : packageName = " + packageName);
        messager.printMessage(Diagnostic.Kind.NOTE,
                "Annotation class : className = " + className);
        messager.printMessage(Diagnostic.Kind.NOTE,
                "Annotation class : fullClassName = " + fullClassName);
        messager.printMessage(Diagnostic.Kind.NOTE,
                "Annotation class : methodName = " + methodName);
        messager.printMessage(Diagnostic.Kind.NOTE,
                "Annotation class : id = " + id + "  name = " + name);
    }
    return true;
}
```


简单介绍一下代码：

1.Messager 用来输出。就像我们平时用的System.out.pringln()和Log.d。输出位置在编译器下方的Messages窗口。这里System.out也是可以用的哦~

2.用for循环遍历所有的 GetMsg注解，然后进行处理。

3.Diagnostic.Kind.NOTE 类似于Log.d Log.e这样的等级。

4.return true;表示该Process已经处理了，其他的Process不需要再处理了。

#### 配置

一定不能忘记的文件配置。

在main文件夹下创建一个resources.META-INF.services文件夹，创建文件

javax.annotation.processing.Processor

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cb9338168294a079f193d1e6a3a20ff~tplv-k3u1fbpfcp-watermark.image)

文件内容是Process类的包名+类名

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5747a0a7d4a4a32916798044d552420~tplv-k3u1fbpfcp-watermark.image)

忘记这个配置文件的后果就是，注解无法生效。

#### 编译jar

这里有一个坑，我们的主Module是不可以直接引用这个java Module的。（直接引用，可以成功运行一次~修改代码以后就不能运行了）

而如何单独编译这个java Module呢？

在编译器Gradle视图里，找到Module apt下的build目录下的Build按钮。双击运行。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cb9338168294a079f193d1e6a3a20ff~tplv-k3u1fbpfcp-watermark.image)


代码没有问题编译通过的话，会有BUILD SUCCESS提示

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a54368528ba4f4a945e6eae24463dda~tplv-k3u1fbpfcp-watermark.image)

生成的jar包在 apt 下的build目录下的libs下

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/421b61f2f4454a76a9aea65ed45f8d11~tplv-k3u1fbpfcp-watermark.image)

将apt.jar拷贝到app下的libs目录，右键该jar，点击Add as Library，添加Library.

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c88739cc724d4340929a60e2c42fb609~tplv-k3u1fbpfcp-watermark.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/693c0073fbd141ffb3e0b64e4d6ec37a~tplv-k3u1fbpfcp-watermark.image)

在APP项目中使用该注解GetMsg。运行。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26ee19d4cf44474bb618374fce33d762~tplv-k3u1fbpfcp-watermark.image)


**当你apt这个包的代码有修改时，需要重复2.6这个步骤。这是比较烦的，但是没办法**


#### 运行结果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/782dd93436be4e0ca9f03e8f7fed0f00~tplv-k3u1fbpfcp-watermark.image)


### 总结


这个Demo只是使用了注解，并且根据注解得到一些必要的信息。没有做代码生成的操作，生成代码的具体操作我们放到后面开始写框架时再讲。

整个项目跑起来，遇到了比较多的坑，我们在下一章讲《Android编译时注解框架-爬坑》

这个Demo的代码我放到了GitHub:

[https://github.com/lizhaoxuan/Android-APT-Framework/tree/master/run-demo/CakeDao](https://github.com/lizhaoxuan/Android-APT-Framework/tree/master/run-demo/CakeDao)


### 修复者注

这个方法已经过时，现在直接引入Google Auto Service即可，gradle可以直接使用annotationProcessor或者kapt来引入注解处理器。
参考：[编译时注解 - ButterKnife源码分析和手写](https://www.jianshu.com/p/100708625605)