## Maven依赖冲突处理

​	Jar包冲突在软件开发过程中是不可避免的，因此，如何快速定位冲突源，理解冲突导致的过程及底层原理，是每个程序员的必修课。也是提升工作效率、应对面试、在团队中脱颖而出的机会。

### 1.依赖冲突

**Jar包冲突的本质：Java应用程序因某种因素，加载不到正确的类，而导致其行为跟预期不一致。**

**实践中能够直观感受到的Jar包冲突表现往往有这几种：**

- 程序抛出`java.lang.ClassNotFoundException`异常；
- 程序抛出`java.lang.NoSuchMethodError`异常；
- 程序抛出`java.lang.NoClassDefFoundError`异常；
- 程序抛出`java.lang.LinkageError`异常等；

**Jar包冲突的情况有以下几种：**

- 情况一：项目直接依赖了同一依赖Jar包的多个版本（直接冲突）
- 情况二：项目中不同的依赖Jar包中出现了相同的依赖的不同版本（间接冲突）
- 情况三：项目中依赖的Jar包里的类并不相同，但是他们互相不能共存（互斥冲突）

### 2.依赖传递

在Maven项目中，想要了解Jar冲突必须得先了解一下Maven是如何管理的Jar包的。

当在Maven项目中引入A的依赖，A的依赖通常又会引入B的jar包，B可能还会引入C的jar包。这样，当你在pom.xml文件中添加了A的依赖，Maven会自动的帮你把所有相关的依赖都添加进来。

比如，在Spring Boot项中，当引入了spring-boot-starter-web：

```text
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

此时Maven的依赖结构可能是这样的：



<img src="https://pic3.zhimg.com/80/v2-9f459bb43036ceaeb0a77f6f849e939e_1440w.webp" alt="img" style="zoom:40%;" />



上面这种关系，我们就可以理解为依赖的传递性。即当一个依赖需要另外一个依赖支撑时，Maven会帮我们把相应的依赖依次添加到项目当中。

这样的好处是，使用起来就非常方便，不用自己挨个去找依赖Jar包了。坏处是会引起Jar包冲突。举例说明：

```text
依赖链路一：A -> B -> C -> G21(guava 21.0)
依赖链路二：D -> F -> G20(guava 20.0)
```

假设项目中同时引入了A和D的依赖，按照依赖传递机制和默认依赖调节机制（第一：路径最近者优先；第二：第一声明优先），默认会引入G20版本的Jar包，而G21的Jar包不会被引用。

如果C中的方法使用了G21版本中的某个新方法（或类），由于Maven的处理，导致G21并未被引入。此时，程序在调用对应**类**时便会抛出`ClassNotFoundException`异常，调用对应**方法**时便会抛出`NoSuchMethodError`异常。

## **排查定位Jar包冲突**

**Maven Helper**

本地环境可以利用Maven Helper等插件来解决

<img src="https://pic3.zhimg.com/80/v2-31bfcfe5f14267680cf32728ea80a816_1440w.webp" alt="img" style="zoom:30%;" />



安装完插件，重启之后，打开pom.xml文件，在文件下面的Dependency Analyzer视图中便可以看到Jar包冲突的结果分析:

<img src="https://pic3.zhimg.com/80/v2-a9786d4c8ee4fa5c882b6a69f85ade32_1440w.webp" alt="img" style="zoom:33%;" />





此时，关于哪些Jar包冲突了，便一目了然。同时，可以右击冲突的Jar包，执行”Exclude“进行排除，在pom.xml中便会自动添加排除jar包的属性。

**mvn命令**

但在预生产或生成环境中就没那么方便了。此时可以通过`mvn`命令来定位突出的细节。

执行如下：

```text
mvn dependency:tree -Dverbose
```

注意不要省略`-Dverbose`，要不然不会显示被忽略的包。

执行结果如下：通过这种形式，也可以清晰的看出哪些Jar包发生了冲突。



<img src="https://pic1.zhimg.com/80/v2-1760e1b03fcd4afde84632c068e76140_1440w.webp" alt="img" style="zoom:33%;" />



## **如何统一Jar包依赖**

像上面截图所示，如果一个项目有N多个子项目构成，项目之间可能还有依赖关系，Jar包冲突不可避免，此时可采用父pom，统一对版本进行管理，一劳永逸。

通常做法，是在parent模块的pom文件中尽可能地声明所有相关依赖Jar包的版本，并在子pom中简单引用（不再指定版本）该构件即可。

比如在父pom.xml中定义Lombok的版本：

```text
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

在子module中便可定义如下：

```text
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

通过这种方式，所有的子module中都采用了统一的版本。

## **解决Jar包冲突的方法**

**这里基于Maven项目介绍几种场景下解决Jar冲突的方法：**

1、Maven默认处理：采用此种方法，要牢记Maven依赖调节机制的基本原则：Maven依赖树的层序遍历

- 最短路径原则

```text
依赖链路一：A -> X -> Y -> Z(21.0)
依赖链路二：B -> Q -> Z(20.0)
```

- 优先声明原则

```text
依赖链路一：A -> X -> Z(21.0)
依赖链路二：B -> Q -> Z(20.0)
```

> 如果层次相同，可以通过调整顺序来解决。如果层次不同，只能通过排除法来解决了。

2、排除法：上面Maven Helper的实例中已经讲到，可以将冲突的Jar包在pom.xml中通过`exclude`来进行排除；

>  POM文件，说明了Maven项目的依赖项。依赖的不是对象，也不是容器，而是Jar包里的类。大多数jar包都是client包，其中的这些类通常被我们程序作为工具调用，或以RPC调用的服务接口常驻在我们应用的Spring容器当中，除了这些我们在本项目中使用的接口、类以及特性，其余的依赖对于本项目都是没用的，通通可以排除，对构建编译和部署都不会产生任何影响。



3、版本锁定法：如果项目中依赖同一Jar包的很多版本，一个个排除非常麻烦，此时可用**版本锁定法**，即直接明确引入指定版本的依赖。根据前面介绍Maven处理Jar包基本原则，此种方式的优先级最高。这种方法一般采用上面我们讲到的**如何统一Jar包依赖**的方式。

>  父模块<denpendencyManagement>中声明的版本优先级很高，要高于子模块中<dependency>中的声明。当出现版本冲突的时候，不是很容易被注意到。处理方法：
>
> 1、在denpendencyManagement中不声明版本，只声明名称
>
> 2、同一在父pom的denpendencyManagement中声明版本，子pom中不再控制版本
>
> 3、用子pom中的denpendencyManagement覆盖父pom中的版本

