![image-20221008182958877](https://s2.loli.net/2022/10/08/ZsKlOvz5xmXSutw.png)

# Spring核心技术

**前置课程要求：**请各位小伙伴先完成《JavaWeb》篇、《Java 9-17新特性》篇视频教程之后，再来观看此教程。

**建议：**对Java开发还不是很熟悉的同学，最好先花费半个月到一个月时间大量地去编写小项目，不推荐一口气学完，后面的内容相比前面的内容几乎是降维打击，一口气学完很容易忘记之前所学的基础知识，尤其是JavaSE阶段的内容。

**不同于2021版本SSM教程，本期视频为重制版本，视频学习的Spring框架版本为：6.0**

恭喜各位顺利进入到SSM（Spring+SpringMVC+Mybatis）阶段的学习，也算是成功出了Java新手村，由于前面我们已经学习过Mybatis了，因此，本期教程的时间安排相比之前会更短一些。从这里开始，很多的概念理解起来就稍微有一点难度了，因为你们没有接触过企业开发场景，很难体会到那种思想带来的好处，甚至到后期接触到的几乎都是基于云计算和大数据理论实现的框架（当下最热门最前沿的技术）逐渐不再是和计算机基础相关联，而是和怎么高效干活相关了。

在JavaWeb阶段，我们已经学习了如何使用Java进行Web应用程序开发，我们现在已经具有搭建Web网站的能力，但是，我们在开发的过程中，发现存在诸多的不便，在最后的图书管理系统编程实战中，我们发现虽然我们思路很清晰，知道如何编写对应的接口，但是这样的开发效率，实在是太慢了，并且对于对象创建的管理，存在诸多的不妥之处，因此，我们要去继续学习更多的框架技术，来简化和规范我们的Java开发。

Spring就是这样的一个框架（文档：https://docs.spring.io/spring-framework/docs/6.0.10/reference/html/core.html#spring-core），它就是为了简化开发而生，它是轻量级的**IoC**和**AOP**的容器框架，主要是针对**Bean**的生命周期进行管理的轻量级容器，并且它的生态已经发展得极为庞大。那么，首先一问，什么是**IoC**和**AOP**，什么又是**Bean**呢？不要害怕，这些概念只是听起来满满的高级感，实际上没有多高级（很多东西都是这样，名字听起来很牛，实际上只是一个很容易理解的东西）

## IoC容器基础

Spring框架最核心的其实它的IoC容器，这是我们开启Spring学习的第一站。

### IoC理论介绍

在我们之前的图书管理系统Web应用程序中，我们发现，整个程序其实是依靠各个部分相互协作，共同完成一个操作，比如要展示借阅信息列表，那么首先需要使用Servlet进行请求和响应的数据处理，然后请求的数据全部交给对应的Service（业务层）来处理，当Service发现要从数据库中获取数据时，再向对应的Mapper发起请求。

它们之间就像连接在一起的齿轮，谁也离不开谁：

![img](https://s2.loli.net/2022/10/08/YQRP2idIS5skHJ4.png)

就像一个团队，每个人的分工都很明确，流水线上的一套操作必须环环相扣，这是一种高度耦合的体系。

虽然这样的体系逻辑非常清晰，整个流程也能够让人快速了解，但是这样存在一个很严重的问题，我们现在的时代实际上是一个软件项目高速迭代的时代，我们发现很多App三天两头隔三差五地就更新，而且是什么功能当下最火，就马不停蹄地进行跟进开发，因此，就很容易出现，之前写好的代码，实现的功能，需要全部推翻，改成新的功能，那么我们就不得不去修改某些流水线上的模块，但是这样一修改，会直接导致整个流水线的引用关系大面积更新。

比如下面的情况：

```java
class A{
    private List<B> list;
    public B test(B b){
        return null;
    }
}

class C{
    public C(B b){}
}

class B{ }
```

可以看到，A和C在大量地直接使用B，但是某一天，这个B的实现已经过时了，此时来了个把功能实现的更好的D，我们需要用这个新的类来完成业务了：

![image-20221122135859871](https://s2.loli.net/2022/11/22/FRQn6vEpTklsJKe.png)

可以看到，因为类之间的关联性太强了，会开始大面积报错，所有之前用了B的类，得挨个进行修改，全都改成D，这简直是灾难啊！

包括我们之前JavaWeb阶段编写的实战项目，如果我们不想用某个Service实现类了，我想使用其他的实现类用不同的逻辑做这些功能，那么这个时候，我们只能每个类都去挨个进行修改，当项目特别庞大时，光是改个类名导致的连带修改就够你改一天了。

因此，高耦合度带来的缺点是很明显的，也是现代软件开发中很致命的问题。如果要改善这种情况，我们只能将各个模块进行解耦，让各个模块之间的依赖性不再那么地强。也就是说，Service的实现类，不再由我们决定，而是让程序自己决定，所有的实现类对象，全部交给程序来管理，所有对象之间的关系，也由程序来动态决定，这样就引入了IoC理论。

IOC是Inversion of Control的缩写，翻译为：“控制反转”，把复杂系统分解成相互合作的对象，这些对象类通过封装以后，内部实现对外部是透明的，从而降低了解决问题的复杂度，而且可以灵活地被重用和扩展。

![img](https://s2.loli.net/2022/10/08/XsYQRk93CHewISB.png)

我们可以将对象交给IoC容器进行管理，比如当我们需要一个接口的实现时，由它根据配置文件来决定到底给我们哪一个实现类，这样，我们就可以不用再关心我们要去使用哪一个实现类了，我们只需要关心，给到我的一定是一个可以正常使用的实现类，能用就完事了，反正接口定义了啥，我只管调，这样，我们就可以放心地让一个人去写视图层的代码，一个人去写业务层的代码，开发效率那是高的一匹啊。

还是之前的代码，但是有了IoC容器加持之后：

```java
public static void main(String[] args) {
		A a = new A();
  	a.test(IoC.getBean(Service.class));   //瞎编的一个容器类，但是是那个意思
  	//比如现在在IoC容器中管理的Service的实现是B，那么我们从里面拿到的Service实现就是B
}

class A{
    private List<Service> list;   //一律使用Service，具体实现由IoC容器提供
    public Service test(Service b){
        return null;
    }
}

interface Service{ }   //使用Service做一个顶层抽象

class B implements Service{}  //B依然是具体实现类，并交给IoC容器管理
```

当具体实现类发生修改时，我们同样只需要将新的实现类交给IoC容器管理，这样我们无需修改之前的任何代码：

```java
interface Service{ }

class D implements Service{}   //现在实现类变成了D，但是之前的代码并不会报错
```

这样，即使我们的底层实现类发生了修改，也不会导致与其相关联的类出现错误，而进行大面积修改，通过定义抽象+容器管理的形式，我们就可以将原有的强关联解除。

高内聚，低耦合，是现代软件的开发的设计目标，而Spring框架就给我们提供了这样的一个IoC容器进行对象的的管理，一个由Spring IoC容器实例化、组装和管理的对象，我们称其为Bean。

### 第一个Spring项目

首先一定要明确，使用Spring首要目的是为了使得软件项目进行解耦，而不是为了去简化代码！通过它，就可以更好的对我们的Bean进行管理，这一部分我们来体验一下Spring的基本使用。

Spring并不是一个独立的框架，它实际上包含了很多的模块：

![image-20221121233807593](https://s2.loli.net/2022/11/21/KT2XhuCNVmcSvi5.png)

而我们首先要去学习的就是Core Container，也就是核心容器模块，只有了解了Spring的核心技术，我们才能真正认识这个框架为我们带来的便捷之处。

Spring是一个非入侵式的框架，就像一个工具库一样，它可以很简单地加入到我们已有的项目中，因此，我们只需要直接导入其依赖就可以使用了，Spring核心框架的Maven依赖坐标：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.0.10</version>
</dependency>
```

**注意：**与旧版教程不同的是，Spring 6要求你使用的Java版本为`17`及以上，包括后面我们在学习SpringMvc时，要求Tomcat版本必须为10以上。这个依赖中包含了如下依赖：

![image-20221122133820198](https://s2.loli.net/2022/11/22/HszTflPavUdQKGJ.png)

这里出现的都是Spring核心相关的内容，如Beans、Core、Context、SpEL以及非常关键的AOP框架，在本章中，我们都会进行讲解。

> 如果在使用Spring框架的过程中出现如下警告：
>
> ```
> 12月 17, 2022 3:26:26 下午 org.springframework.core.LocalVariableTableParameterNameDiscoverer inspectClass
> 警告: Using deprecated '-debug' fallback for parameter name resolution. Compile the affected code with '-parameters' instead or avoid its introspection: XXXX
> ```
>
> 这是因为LocalVariableTableParameterNameDiscoverer在Spring 6.0.1版本已经被标记为过时，并且即将移除，请在Maven配置文件中为编译插件添加`-parameters`编译参数：
>
> ```xml
> <build>
>     <pluginManagement>
>         <plugins>
>             <plugin>
>                 <artifactId>maven-compiler-plugin</artifactId>
>                 <version>3.10.1</version>
>                 <configuration>
>                     <compilerArgs>
>                         <arg>-parameters</arg>
>                     </compilerArgs>
>                 </configuration>
>             </plugin>
>         </plugins>
>     </pluginManagement>
> </build>
> ```
>
> 没有此问题请无视这部分。

这里我们就来尝试编写一个最简的Spring项目，我们在前面已经讲过了，Spring会给我们提供IoC容器用于管理Bean，但是我们得先为这个容器编写一个配置文件，我们可以通过配置文件告诉容器需要管理哪些Bean以及Bean的属性、依赖关系等等。

首先我们需要在resource中创建一个Spring配置文件（在resource中创建的文件，会在编译时被一起放到类路径下），命名为test.xml，直接右键点击即可创建：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

此时IDEA会提示我们没有为此文件配置应用程序上下文，这里我们只需要指定成当前项目就行了，当然配置这个只是为了代码提示和依赖关系快速查看，如果不进行配置也不会影响什么，程序依然可以正常运行：

![image-20221121234427739](https://s2.loli.net/2022/11/21/bBtrSWlVz9oD2JF.png)

这里我们直接按照默认配置点确定就行了：

![image-20221121234447213](https://s2.loli.net/2022/11/21/xoatfu4nX6iRr3v.png)

Spring为我们提供了一个IoC容器，用于去存放我们需要使用的对象，我们可以将对象交给IoC容器进行管理，当我们需要使用对象时，就可以向IoC容器去索要，并由它来决定给我们哪一个对象。而我们如果需要使用Spring为我们提供的IoC容器，那么就需要创建一个应用程序上下文，它代表的就是IoC容器，它会负责实例化、配置和组装Bean：

```java
public static void main(String[] args) {
  	//ApplicationContext是应用程序上下文的顶层接口，它有很多种实现，这里我们先介绍第一种
  	//因为这里使用的是XML配置文件，所以说我们就使用 ClassPathXmlApplicationContext 这个实现类
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");  //这里写上刚刚的名字
}
```

比如现在我们要让IoC容器帮助我们管理一个Student对象（Bean），当我们需要这个对象时再申请，那么就需要这样，首先先将Student类定义出来：

```java
package com.test.bean;

public class Student {
    public void hello(){
        System.out.println("Hello World!");
    }
}
```

既然现在要让别人帮忙管理对象，那么就不能再由我们自己去new这个对象了，而是编写对应的配置，我们打开刚刚创建的`test.xml`文件进行编辑，添加：

```java
<bean name="student" class="com.test.bean.Student"/>
```

这里我们就在配置文件中编写好了对应Bean的信息，之后容器就会根据这里的配置进行处理了。

现在，这个对象不需要我们再去创建了，而是由IoC容器自动进行创建并提供，我们可以直接从上下文中获取到它为我们创建的对象：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");
    Student student = (Student) context.getBean("student");   //使用getBean方法来获取对应的对象（Bean）
    student.hello();
}
```

实际上，这里得到的Student对象是由Spring通过反射机制帮助我们创建的，初学者会非常疑惑，为什么要这样来创建对象，我们直接new一个它不香吗？为什么要交给IoC容器管理呢？在后面的学习中，我们再慢慢进行体会。

![image-20221122153946251](https://s2.loli.net/2022/11/22/sjLiFokU1f3CvH5.png)

### Bean注册与配置

前面我们通过一个简单例子体验了一下如何使用Spring来管理我们的对象，并向IoC容器索要被管理的对象。这节课我们就来详细了解一下如何向Spring注册Bean以及Bean的相关配置。

实际上我们的配置文件可以有很多个，并且这些配置文件是可以相互导入的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>
    <import resource="test.xml"/>
</beans>
```

但是为了简单起见，我们还是从单配置文件开始讲起，首先我们需要知道如何配置Bean并注册。

要配置一个Bean，只需要添加：

```xml
<bean/>
```

但是这样写的话，Spring无法得知我们要配置的Bean到底是哪一个类，所以说我们还得指定对应的类才可以：

```xml
<bean class="com.test.bean.Student"/>
```

![image-20221122164149924](https://s2.loli.net/2022/11/22/SRV3znQJH4A7vDl.png)

可以看到类的旁边出现了Bean的图标，表示我们的Bean已经注册成功了，这样，我们就可以根据类型向容器索要Bean实例对象了：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");
  	//getBean有多种形式，其中第一种就是根据类型获取对应的Bean
  	//容器中只要注册了对应类的Bean或是对应类型子类的Bean，都可以获取到
    Student student = context.getBean(Student.class);
    student.hello();
}
```

不过在有些时候，Bean的获取可能会出现歧义，我们可以来分别注册两个子类的Bean：

```java
public class ArtStudent extends Student{
  	public void art(){
        System.out.println("我爱画画");
    }
}
```

```java
public class SportStudent extends Student{
		public void sport(){
        System.out.println("我爱运动");
    }
}
```

```xml
<bean class="com.test.bean.ArtStudent"/>
<bean class="com.test.bean.SportStudent"/>
```

但是此时我们在获取Bean时却是索要的它们的父类：

```java
Student student = context.getBean(Student.class);
student.hello();
```

运行时得到如下报错：

![image-20221122164100561](https://s2.loli.net/2022/11/22/vliWJ4SZdx5E6yX.png)

这里出现了一个Bean定义不唯一异常，很明显，因为我们需要的类型是Student，但是此时有两个Bean定义都满足这个类型，它们都是Student的子类，此时IoC容器不知道给我们返回哪一个Bean，所以就只能抛出异常了。

因此，如果我们需要一个Bean并且使用类型进行获取，那么必须要指明类型并且不能出现歧义：

```java
ArtStudent student = context.getBean(ArtStudent.class);
student.art();
```

那要是两个Bean的类型都是一样的呢？

```java
<bean class="com.test.bean.Student"/>
<bean class="com.test.bean.Student"/>
```

这种情况下，就无法使用Class来进行区分了，除了为Bean指定对应类型之外，我们也可以为Bean指定一个名称用于区分：

```xml
<bean name="art" class="com.test.bean.ArtStudent"/>
<bean name="sport" class="com.test.bean.SportStudent"/>
```

name属性就是为这个Bean设定一个独一无二的名称（id属性也可以，跟name功能相同，但是会检查命名是否规范，否则会显示黄标），不同的Bean名字不能相同，否则报错：

```xml
<bean name="a" class="com.test.bean.Student"/>
<bean name="b" class="com.test.bean.Student"/>
```

这样，这两个Bean我们就可以区分出来了：

```java
Student student = (Student) context.getBean("a");
student.hello();
```

虽然目前这两Bean定义都是一模一样的，也没什么区别，但是这确实是两个不同的Bean，只是类型一样而已，之后我们还可以为这两个Bean分别设置不同的其他属性。

我们可以给Bean起名字，也可以起别名，就行我们除了有一个名字之外，可能在家里还有自己的小名：

```xml
<bean name="a" class="com.test.bean.Student"/>
<alias name="a" alias="test"/>
```

这样，我们使用别名也是可以拿到对应的Bean的：

```java
Student student = (Student) context.getBean("test");
student.hello();
```

那么现在又有新的问题了，IoC容器创建的Bean是只有一个还是每次索要的时候都会给我们一个新的对象？我们现在在主方法中连续获取两次Bean对象：

```java
Student student1 = context.getBean(Student.class);
Student student2 = context.getBean(Student.class);
System.out.println(student1 == student2);   //默认为单例模式，对象始终为同一个
```

我们发现，最后得到的结果为true，那么说明每次从IoC容器获取到的对象，始终都是同一个，默认情况下，通过IoC容器进行管理的Bean都是单例模式的，这个对象只会被创建一次。

如果我们希望每次拿到的对象都是一个新的，我们也可以将其作用域进行修改：

![image-20221122175719997](https://s2.loli.net/2022/11/22/hDGo7m9uBlgVn5A.png)

这里一共有两种作用域，第一种是`singleton`，默认情况下就是这一种，当然还有`prototype`，表示为原型模式（为了方便叫多例模式也行）这种模式每次得到的对象都是一个新的：

```java
Student student1 = context.getBean(Student.class);  //原型模式下，对象不再始终是同一个了
Student student2 = context.getBean(Student.class);
System.out.println(student1 == student2);
```

实际上，当Bean的作用域为单例模式时，那么它会在一开始（容器加载配置时）就被创建，我们之后拿到的都是这个对象。而处于原型模式下，只有在获取时才会被创建，也就是说，单例模式下，Bean会被IoC容器存储，只要容器没有被销毁，那么此对象将一直存在，而原型模式才是相当于在要用的时候直接new了一个对象，并不会被保存。

当然，如果我们希望单例模式下的Bean不用再一开始就加载，而是一样等到需要时再加载（加载后依然会被容器存储，之后一直使用这个对象了，不会再创建新的）我们也可以开启懒加载：

```xml
<bean class="com.test.bean.Student" lazy-init="true"/>
```

开启懒加载后，只有在真正第一次使用时才会创建对象。

因为单例模式下Bean是由IoC容器加载，但是加载顺序我们并不清楚，如果我们需要维护Bean的加载顺序（比如某个Bean必须要在另一个Bean之前创建）那么我们可以使用`depends-on`来设定前置加载Bean，这样被依赖的Bean一定会在之前加载，比如Teacher应该在Student之前加载：

```xml
<bean name="teacher" class="com.test.bean.Teacher"/>
<bean name="student" class="com.test.bean.Student" depends-on="teacher"/>
```

这样就可以保证Bean的加载顺序了。

### 依赖注入

依赖注入(Dependency Injection, DI)是一种设计模式，也是Spring框架的核心概念之一。现在我们已经了解了如何注册和使用一个Bean，但是这样还远远不够，还记得我们一开始说的，消除类之间的强关联吗？比如现在有一个教师接口：

```java
public interface Teacher {
    void teach();
}
```

具体的实现有两个：

```java
public class ArtTeacher implements Teacher{
    @Override
    public void teach() {
        System.out.println("我是美术老师，我教你画画！");
    }
}
```

```java
public class ProgramTeacher implements Teacher{
    @Override
    public void teach() {
        System.out.println("我是编程老师，我教你学Golang！");
    }
}
```

我们的学生一开始有一个老师教他，比如美术老师：

```java
public class Student {
    private Teacher teacher = new ArtTeacher();   
  	//在以前，如果我们需要制定哪个老师教我们，直接new创建对应的对象就可以了
    public void study(){
        teacher.teach();
    }
}
```

但是我们发现，如果美术老师不教了，现在来了一个其他的老师教学生，那么就需要去修改Student类的定义：

```java
public class Student {
    private Teacher teacher = new ProgramTeacher();
  	...
```

可以想象一下，如果现在冒出来各种各样的类都需要这样去用Teacher，那么一旦Teacher的实现发生变化，会导致我们挨个对之前用到Teacher的类进行修改，这就很难受了。

而有了依赖注入之后，Student中的Teacher成员变量，可以由IoC容器来选择一个合适的Teacher对象进行赋值，也就是说，IoC容器在创建对象时，需要将我们预先给定的属性注入到对象中，非常简单，我们可以使用`property`标签来实现，我们将bean标签展开：

```xml
<bean name="teacher" class="com.test.bean.ProgramTeacher"/>
<bean name="student" class="com.test.bean.Student">
    <property name="teacher" ref="teacher"/>
</bean>
```

同时我们还需要修改一下Student类，依赖注入要求对应的属性必须有一个set方法：

```java
public class Student {
    private Teacher teacher;
  	//要使用依赖注入，我们必须提供一个set方法（无论成员变量的访问权限是什么）命名规则依然是驼峰命名法
    public void setTeacher(Teacher teacher) {
        this.teacher = teacher;
    }
    ...
```

![image-20221122191025279](https://s2.loli.net/2022/11/22/wu2EYC8HToJbsva.png)

使用`property`来指定需要注入的值或是一个Bean，这里我们选择ProgramTeacher，那么在使用时，Student类中的得到的就是这个Bean的对象了：

```java
Student student = context.getBean(Student.class);
student.study();
```

![image-20221122191109690](https://s2.loli.net/2022/11/22/n3zYWvIoE8CrRDT.png)

可以看到，现在我们的Java代码中，没有出现任何的具体实现类信息（ArtTeacher、ProgramTeacher都没出现）取而代之的是那一堆xml配置，这样，就算我们切换老师的实现为另一个类，也不用去调整代码，只需要变动一下Bean的类型就可以：

```xml
<!--  只需要修改这里的class即可，现在改为ArtTeacher  -->
<bean name="teacher" class="com.test.bean.ArtTeacher"/>
<bean name="student" class="com.test.bean.Student">
    <property name="teacher" ref="teacher"/>
</bean>
```

这样，这个Bean的class就变成了新的类型，并且我们不需要再去调整其他位置的代码，再次启动程序：

![image-20221122191427776](https://s2.loli.net/2022/11/22/evKArqDYcIQPCXT.png)

通过依赖注入，是不是开始逐渐感受到Spring为我们带来的便利了？当然，依赖注入并不一定要注入其他的Bean，也可以是一个简单的值：

```java
<bean name="student" class="com.test.bean.Student">
    <property name="name" value="卢本伟"/>
</bean>
```

直接使用`value`可以直接传入一个具体值。

实际上，在很多情况下，类中的某些参数是在构造方法中就已经完成的初始化，而不是创建之后，比如：

```java
public class Student {
    private final Teacher teacher;   //构造方法中完成，所以说是一个final变量

    public Student(Teacher teacher){   //Teacher属性是在构造方法中完成的初始化
        this.teacher = teacher;
    }
  	...
```

我们前面说了，Bean实际上是由IoC容器进行创建的，但是现在我们修改了默认的无参构造，可以看到配置文件里面报错了：

![image-20221122174328107](https://s2.loli.net/2022/11/22/5HN8GKQywWaYvrF.png)

很明显，是因为我们修改了构造方法，IoC容器默认只会调用无参构造，所以，我们需要指明一个可以用的构造方法，我们展开bean标签，添加一个`constructor-arg`标签：

```xml
<bean name="teacher" class="com.test.bean.ArtTeacher"/>
<bean name="student" class="com.test.bean.Student">
    <constructor-arg name="teacher" ref="teacher"/>
</bean>
```

这里的`constructor-arg`就是构造方法的一个参数，这个参数可以写很多个，会自动匹配符合里面参数数量的构造方法，这里匹配的就是我们刚刚编写的需要一个参数的构造方法。

![image-20221122191427776](https://s2.loli.net/2022/11/22/evKArqDYcIQPCXT.png)

通过这种方式，我们也能实现依赖注入，只不过现在我们将依赖注入的时机提前到了对象构造时。

那要是出现这种情况呢？现在我们的Student类中是这样定义的：

```java
public class Student {
    private final String name;
    public Student(String name){
        System.out.println("我是一号构造方法");
        this.name = name;
    }

    public Student(int age){
        System.out.println("我是二号构造方法");
        this.name = String.valueOf(age);
    }
}
```

此时我们希望使用的是二号构造方法，那么怎么才能指定呢？有2种方式，我们可以给标签添加类型：

```xml
<constructor-arg value="1" type="int"/>
```

也可以指定为对应的参数名称：

```xml
<constructor-arg value="1" name="age"/>
```

反正只要能够保证我们指定的参数匹配到目标构造方法即可。

现在我们的类中出现了一个比较特殊的类型，它是一个集合类型：

```java
public class Student {
    private List<String> list;

    public void setList(List<String> list) {
        this.list = list;
    }
}
```

对于这种集合类型，有着特殊的支持：

```xml
<bean name="student" class="com.test.bean.Student">
  	<!--  对于集合类型，我们可以直接使用标签编辑集合的默认值  -->
    <property name="list">
        <list>
            <value>AAA</value>
            <value>BBB</value>
            <value>CCC</value>
        </list>
    </property>
</bean>
```

不仅仅是List，Map、Set这类常用集合类包括数组在内，都是支持这样编写的，比如Map类型，我们也可以使用`entry`来注入：

```xml
<bean name="student" class="com.test.bean.Student">
    <property name="map">
        <map>
            <entry key="语文" value="100.0"/>
            <entry key="数学" value="80.0"/>
            <entry key="英语" value="92.5"/>
        </map>
    </property>
</bean>
```

至此，我们就已经完成了两种依赖注入的学习：

* Setter依赖注入：通过成员属性对应的set方法完成注入。
* 构造方法依赖注入：通过构造方法完成注入。

### 自动装配

在之前，如果我们需要使用依赖注入的话，我们需要对`property`参数进行配置：

```xml
<bean name="student" class="com.test.bean.Student">
    <property name="teacher" ref="teacher"/>
</bean>
```

但是有些时候为了方便，我们也可以开启自动装配。自动装配就是让IoC容器自己去寻找需要填入的值，我们只需要将set方法提供好就可以了，这里需要添加autowire属性：

```xml
<bean name="student" class="com.test.bean.Student" autowire="byType"/>
```

`autowire`属性有两个值普通，一个是byName，还有一个是byType，顾名思义，一个是根据类型去寻找合适的Bean自动装配，还有一个是根据名字去找，这样我们就不需要显式指定`property`了。

![image-20221122221936559](https://s2.loli.net/2022/11/22/QIBRwScq6fu4XDm.png)

此时set方法旁边会出现一个自动装配图标，效果和上面是一样的。

对于使用构造方法完成的依赖注入，也支持自动装配，我们只需要将autowire修改为：

```xml
<bean name="student" class="com.test.bean.Student" autowire="constructor"/>
```

这样，我们只需要提供一个对应参数的构造方法就可以了（这种情况默认也是byType寻找的）：

![image-20221122230320004](https://s2.loli.net/2022/11/22/rgl7fXJ2ZKAU8Rd.png)

这样同样可以完成自动注入：

![image-20221122191427776](https://s2.loli.net/2022/11/22/evKArqDYcIQPCXT.png)

自动化的东西虽然省事，但是太过机械，有些时候，自动装配可能会遇到一些问题，比如出现了下面的情况：

![image-20221122223048820](https://s2.loli.net/2022/11/22/SQTchJBq4G8NWyC.png)

此时，由于`autowire`的规则为byType，存在两个候选Bean，但是我们其实希望ProgramTeacher这个Bean在任何情况下都不参与到自动装配中，此时我们就可以将它的自动装配候选关闭：

```xml
<bean name="teacher" class="com.test.bean.ArtTeacher"/>
<bean name="teacher2" class="com.test.bean.ProgramTeacher" autowire-candidate="false"/>
<bean name="student" class="com.test.bean.Student" autowire="byType"/>
```

当`autowire-candidate`设定false时，这个Bean将不再作为自动装配的候选Bean，此时自动装配候选就只剩下一个唯一的Bean了，报错消失，程序可以正常运行。

除了这种方式，我们也可以设定primary属性，表示这个Bean作为主要的Bean，当出现歧义时，也会优先选择：

```xml
<bean name="teacher" class="com.test.bean.ArtTeacher" primary="true"/>
<bean name="teacher2" class="com.test.bean.ProgramTeacher"/>
<bean name="student" class="com.test.bean.Student" autowire="byType"/>
```

这样写程序依然可以正常运行，并且选择的也是ArtTeacher（就是不知道为什么IDEA会上红标，BUG？）

### 生命周期与继承

除了修改构造方法，我们也可以为Bean指定初始化方法和销毁方法，以便在对象创建和被销毁时执行一些其他的任务：

```java
public void init(){
    System.out.println("我是对象初始化时要做的事情！");    
}

public void destroy(){
    System.out.println("我是对象销毁时要做的事情！");
}
```

我们可以通过`init-method`和`destroy-method`来指定：

```xml
<bean name="student" class="com.test.bean.Student" init-method="init" destroy-method="destroy"/>
```

那么什么时候是初始化，什么时候又是销毁呢？

```java
//当容器创建时，默认情况下Bean都是单例的，那么都会在一开始就加载好，对象构造完成后，会执行init-method
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("test.xml");
//我们可以调用close方法关闭容器，此时容器内存放的Bean也会被一起销毁，会执行destroy-method
context.close();
```

所以说，最后的结果为：

![image-20221123132604262](https://s2.loli.net/2022/11/23/GWIyPDOaK4TAM1N.png)

注意，如果Bean不是单例模式，而是采用的原型模式，那么就只会在获取时才创建，并调用init-method，而对应的销毁方法不会被调用（因此，对于原型模式下的Bean，Spring无法顾及其完整生命周期，而在单例模式下，Spring能够从Bean对象的创建一直管理到对象的销毁）官方文档原文如下：

> In contrast to the other scopes, Spring does not manage the complete lifecycle of a prototype bean. The container instantiates, configures, and otherwise assembles a prototype object and hands it to the client, with no further record of that prototype instance. Thus, although initialization lifecycle callback methods are called on all objects regardless of scope, in the case of prototypes, configured destruction lifecycle callbacks are not called. The client code must clean up prototype-scoped objects and release expensive resources that the prototype beans hold. To get the Spring container to release resources held by prototype-scoped beans, try using a custom [bean post-processor](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp), which holds a reference to beans that need to be cleaned up.

Bean之间也是具备继承关系的，只不过这里的继承并不是类的继承，而是属性的继承，比如：

```java
public class SportStudent {
    private String name;

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
public class ArtStudent {
    private String name;
   
    public void setName(String name) {
        this.name = name;
    }
}
```

此时，我们先将ArtStudent注册一个Bean：

```xml
<bean name="artStudent" class="com.test.bean.ArtStudent">
    <property name="name" value="小明"/>
</bean>
```

这里我们会注入一个name的初始值，此时我们创建了一个SportStudent的Bean，我们希望这个Bean的属性跟刚刚创建的Bean属性是一样的，那么我们可以写一个一模一样的：

```xml
<bean class="com.test.bean.SportStudent">
    <property name="name" value="小明"/>
</bean>
```

但是如果属性太多的话，是不是写起来有点麻烦？这种情况，我们就可以配置Bean之间的继承关系了，我们可以让SportStudent这个Bean直接继承ArtStudent这个Bean配置的属性：

```xml
<bean class="com.test.bean.SportStudent" parent="artStudent"/>
```

这样，在ArtStudent Bean中配置的属性，会直接继承给SportStudent Bean（注意，所有配置的属性，在子Bean中必须也要存在，并且可以进行注入，否则会出现错误）当然，如果子类中某些属性比较特殊，也可以在继承的基础上单独配置：

```xml
<bean name="artStudent" class="com.test.bean.ArtStudent" abstract="true">
    <property name="name" value="小明"/>
    <property name="id" value="1"/>
</bean>
<bean class="com.test.bean.SportStudent" parent="artStudent">
    <property name="id" value="2"/>
</bean>
```

如果我们只是希望某一个Bean仅作为一个配置模版供其他Bean继承使用，那么我们可以将其配置为abstract，这样，容器就不会创建这个Bean的对象了：

```xml
<bean name="artStudent" class="com.test.bean.ArtStudent" abstract="true">
    <property name="name" value="小明"/>
</bean>
<bean class="com.test.bean.SportStudent" parent="artStudent"/>
```

注意，一旦声明为抽象Bean，那么就无法通过容器获取到其实例化对象了。

![image-20221123140409416](https://s2.loli.net/2022/11/23/SyDkvOldB7ETW4z.png)

不过Bean的继承使用频率不是很高，了解就行。

这里最后再提一下，我们前面已经学习了各种各样的Bean配置属性，如果我们希望整个上下文中所有的Bean都采用某种配置，我们可以在最外层的beans标签中进行默认配置：

![image-20221123141221259](https://s2.loli.net/2022/11/23/KzSUJXa4jBfO9rd.png)

这样，即使Bean没有配置某项属性，但是只要在最外层编写了默认配置，那么同样会生效，除非Bean自己进行配置覆盖掉默认配置。

### 工厂模式和工厂Bean

前面我们介绍了IoC容器的Bean创建机制，默认情况下，容器会调用Bean对应类型的构造方法进行对象创建，但是在某些时候，我们可能不希望外界使用类的构造方法完成对象创建，比如在工厂方法设计模式中（详情请观看《Java设计模式》篇 视频教程）我们更希望 Spring不要直接利用反射机制通过构造方法创建Bean对象， 而是利用反射机制先找到对应的工厂类，然后利用工厂类去生成需要的Bean对象：

```java
public class Student {
    Student() {
        System.out.println("我被构造了");
    }
}
```

```java
public class StudentFactory {
    public static Student getStudent(){
      	System.out.println("欢迎光临电子厂");
        return new Student();
    }
}
```

此时Student有一个工厂，我们正常情况下需要使用工厂才可以得到Student对象，现在我们希望Spring也这样做，不要直接去反射搞构造方法创建，我们可以通过factory-method进行指定：

```xml
<bean class="com.test.bean.StudentFactory" factory-method="getStudent"/>
```

注意，这里的Bean类型需要填写为Student类的工厂类，并且添加factory-method指定对应的工厂方法，但是最后注册的是工厂方法的返回类型，所以说依然是Student的Bean：

![image-20221123143302785](https://s2.loli.net/2022/11/23/5Id43xPneJiWfZs.png)

此时我们再去进行获取，拿到的也是通过工厂方法得到的对象：

![image-20221123143347376](https://s2.loli.net/2022/11/23/l8HzN7Rwthqrim5.png)

这里有一个误区，千万不要认为是我们注册了StudentFactory这个Bean，class填写为这个类这个只是为了告诉Spring我们的工厂方法在哪个位置，真正注册的是工厂方法提供的东西。

可以发现，当我们采用工厂模式后，我们就无法再通过配置文件对Bean进行依赖注入等操作了，而是只能在工厂方法中完成，这似乎与Spring的设计理念背道而驰？

当然，可能某些工厂类需要构造出对象之后才能使用，我们也可以将某个工厂类直接注册为工厂Bean：

```java
public class StudentFactory {
    public Student getStudent(){
        System.out.println("欢迎光临电子厂");
        return new Student();
    }
}
```

现在需要StudentFactory对象才可以获取到Student，此时我们就只能先将其注册为Bean了：

```xml
<bean name="studentFactory" class="com.test.bean.StudentFactory"/>
```

像这样将工厂类注册为Bean，我们称其为工厂Bean，然后再使用`factory-bean`来指定Bean的工厂Bean：

```xml
<bean factory-bean="studentFactory" factory-method="getStudent"/>
```

注意，使用factory-bean之后，不再要求指定class，我们可以直接使用了：

![image-20221123164134470](https://s2.loli.net/2022/11/23/ih1Af7xBdX3ebaG.png)

此时可以看到，工厂方法上同样有了图标，这种方式，由于工厂类被注册为Bean，此时我们就可以在配置文件中为工厂Bean配置依赖注入等内容了。

这里还有一个很细节的操作，如果我们想获取工厂Bean为我们提供的Bean，可以直接输入工厂Bean的名称，这样不会得到工厂Bean的实例，而是工厂Bean生产的Bean的实例：

```java
Student bean = (Student) context.getBean("studentFactory");
```

当然，如果我们需要获取工厂类的实例，可以在名称前面添加`&`符号：

```java
StudentFactory bean = (StudentFactory) context.getBean("&studentFactory");
```

又是一个小细节。

### 使用注解开发

前面我们已经完成了大部分的配置文件学习，但是我们发现，使用配置文件进行配置，貌似有点太累了吧？可以想象一下，如果我们的项目非常庞大，整个配置文件将会充满Bean配置，并且会继续庞大下去，能否有一种更加高效的方法能够省去配置呢？还记得我们在JavaWeb阶段用到的非常方便东西吗？没错，就是注解。

既然现在要使用注解来进行开发，那么我们就删掉之前的xml配置文件吧，我们来看看使用注解能有多方便。

```java
ApplicationContext context = new AnnotationConfigApplicationContext();
```

现在我们使用AnnotationConfigApplicationContext作为上下文实现，它是注解配置的。

既然现在采用注解，我们就需要使用类来编写配置文件，在之前，我们如果要编写一个配置的话，需要：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

现在我们只需要创建一个配置类就可以了：

```java
@Configuration
public class MainConfiguration {
}
```

这两者是等价的，同样的，在一开始会提示我们没有配置上下文：

![image-20221123175555433](https://s2.loli.net/2022/11/23/YNl5nVkfJriogdD.png)

这里按照要求配置一下就可以，同上，这个只是会影响IDEA的代码提示，不会影响程序运行。

我们可以为AnnotationConfigApplicationContext指定一个默认的配置类：

```java
ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
//这个构造方法可以接收多个配置类（更准确的说是多个组件）
```

那么现在我们该如何配置Bean呢？

```java
@Configuration
public class MainConfiguration {

    @Bean("student")
    public Student student(){
        return new Student();
    }
}
```

这样写相对于配置文件中的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean name = "student" class="com.test.bean.Student"/>
</beans>
```

通过@Import还可以引入其他配置类：

```java
@Import(LBWConfiguration.class)  //在讲解到Spring原理时，我们还会遇到它，目前只做了解即可。
@Configuration
public class MainConfiguration {
```

只不过现在变成了由Java代码为我们提供Bean配置，这样会更加的灵活，也更加便于控制Bean对象的创建。

```java
ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
Student student = context.getBean(Student.class);
System.out.println(student);
```

使用方法是相同的，这跟使用XML配置是一样的。

那么肯定就有小伙伴好奇了，我们之前使用的那么多特性在哪里配置呢？首先，初始化方法和摧毁方法、自动装配可以直接在@Bean注解中进行配置：

```java
@Bean(name = "", initMethod = "", destroyMethod = "", autowireCandidate = false)
public Student student(){
    return new Student();
}
```

其次，我们可以使用一些其他的注解来配置其他属性，比如：

```java
@Bean
@Lazy(true)     //对应lazy-init属性
@Scope("prototype")    //对应scope属性
@DependsOn("teacher")    //对应depends-on属性
public Student student(){
    return new Student();
}
```

对于那些我们需要通过构造方法或是Setter完成依赖注入的Bean，比如：

```xml
<bean name="teacher" class="com.test.bean.ProgramTeacher"/>
<bean name="student" class="com.test.bean.Student">
    <property name="teacher" ref="teacher"/>
</bean>
```

像这种需要引入其他Bean进行的注入，我们可以直接将其作为形式参数放到方法中：

```java
@Configuration
public class MainConfiguration {
    @Bean
    public Teacher teacher(){
        return new Teacher();
    }

    @Bean
    public Student student(Teacher teacher){
        return new Student(teacher);
    }
}
```

此时我们可以看到，旁边已经出现图标了：

![image-20221123213527325](https://s2.loli.net/2022/11/23/wy5JtiVp8zK1bmG.png)

运行程序之后，我们发现，这样确实可以直接得到对应的Bean并使用。

只不过，除了这种基于构造器或是Setter的依赖注入之外，我们也可以直接到Bean对应的类中使用自动装配：

```java
public class Student {
    @Autowired   //使用此注解来进行自动装配，由IoC容器自动为其赋值
    private Teacher teacher;
}
```

现在，我们甚至连构造方法和Setter都不需要去编写了，就能直接完成自动装配，是不是感觉比那堆配置方便多了？

当然，@Autowired并不是只能用于字段，对于构造方法或是Setter，它同样可以：

```java
public class Student {
    private Teacher teacher;

    @Autowired
    public void setTeacher(Teacher teacher) {
        this.teacher = teacher;
    }
}
```

@Autowired默认采用byType的方式进行自动装配，也就是说会使用类型进行配，那么要是出现了多个相同类型的Bean，如果我们想要指定使用其中的某一个该怎么办呢？

```java
@Bean("a")
public Teacher teacherA(){
    return new Teacher();
}

@Bean("b")
public Teacher teacherB(){
    return new Teacher();
}
```

此时，我们可以配合@Qualifier进行名称匹配：

```java
public class Student {
    @Autowired
    @Qualifier("a")   //匹配名称为a的Teacher类型的Bean
    private Teacher teacher;
}
```

这里需要提一下，在我们旧版本的SSM教程中讲解了@Resource这个注解，但是现在它没有了。

随着Java版本的更新迭代，某些javax包下的包，会被逐渐弃用并移除。在JDK11版本以后，javax.annotation这个包被移除并且更名为jakarta.annotation（我们在JavaWeb篇已经介绍过为什么要改名字了）其中有一个非常重要的注解，叫做@Resource，它的作用与@Autowired时相同的，也可以实现自动装配，但是在IDEA中并不推荐使用@Autowired注解对成员字段进行自动装配，而是推荐使用@Resource，如果需要使用这个注解，还需要额外导入包：

```xml
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
```

使用方法一样，直接替换掉就可以了：

```java
public class Student {
    @Resource
    private Teacher teacher;
}
```

只不过，他们两有些机制上的不同：

- @Resource默认**ByName**如果找不到则**ByType**，可以添加到set方法、字段上。
- @Autowired默认是**byType**，只会根据类型寻找，可以添加在构造方法、set方法、字段、方法参数上。

因为@Resource的匹配机制更加合理高效，因此官方并不推荐使用@Autowired字段注入，当然，实际上Spring官方更推荐我们使用基于构造方法或是Setter的@Autowired注入，比如Setter 注入的一个好处是，Setter 方法使该类的对象能够在以后重新配置或重新注入。其实，最后使用哪个注解，还是看你自己，要是有强迫症不能忍受黄标但是又实在想用字段注入，那就用@Resource注解。

除了这个注解之外，还有@PostConstruct和@PreDestroy，它们效果和init-method和destroy-method是一样的：

```java
@PostConstruct
public void init(){
    System.out.println("我是初始化方法");
}

@PreDestroy
public void destroy(){
    System.out.println("我是销毁方法");
}
```

我们只需要将其添加到对应的方法上即可：

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
Student student = context.getBean(Student.class);
context.close();
```

![image-20221123225232979](https://s2.loli.net/2022/11/23/wXRuAjVF2ykCOJ4.png)

可以看到效果是完全一样的，这些注解都是jakarta.annotation提供的，有关Spring和JakartaEE的渊源，还请各位小伙伴自行了解。

前面我们介绍了使用@Bean来注册Bean，但是实际上我们发现，如果只是简单将一个类作为Bean的话，这样写还是不太方便，因为都是固定模式，就是单纯的new一个对象出来，能不能像之前一样，让容器自己反射获取构造方法去生成这个对象呢？

肯定是可以的，我们可以在需要注册为Bean的类上添加`@Component`注解来将一个类进行注册**（现在最常用的方式）**，不过要实现这样的方式，我们需要添加一个自动扫描来告诉Spring，它需要在哪些包中查找我们提供的`@Component`声明的Bean。

```java
@Component("lbwnb")   //同样可以自己起名字
public class Student {

}
```

要注册这个类的Bean，只需要添加@Component即可，然后配置一下包扫描：

```java
@Configuration
@ComponentScan("com.test.bean")   //包扫描，这样Spring就会去扫描对应包下所有的类
public class MainConfiguration {

}
```

Spring在扫描对应包下所有的类时，会自动将那些添加了@Component的类注册为Bean，是不是感觉很方便？只不过这种方式只适用于我们自己编写类的情况，如果是第三方包提供的类，只能使用前者完成注册，并且这种方式并不是那么的灵活。

不过，无论是通过@Bean还是@Component形式注册的Bean，Spring都会为其添加一个默认的name属性，比如：

```java
@Component
public class Student {
}
```

它的默认名称生产规则依然是类名并按照首字母小写的驼峰命名法来的，所以说对应的就是student：

```java
Student student = (Student) context.getBean("student");   //这样同样可以获取到
System.out.println(student);
```

同样的，如果是通过@Bean注册的，默认名称是对应的方法名称：

```java
@Bean
public Student artStudent(){
    return new Student();
}
```

```java
Student student = (Student) context.getBean("artStudent");
System.out.println(student);
```

相比传统的XML配置方式，注解形式的配置确实能够减少我们很多工作量。并且，对于这种使用`@Component`注册的Bean，如果其构造方法不是默认无参构造，那么默认会对其每一个参数都进行自动注入：

```java
@Component
public class Student {
    Teacher teacher;
    public Student(Teacher teacher){   //如果有Teacher类型的Bean，那么这里的参数会被自动注入
        this.teacher = teacher;
    }
}
```

最后，对于我们之前使用的工厂模式，Spring也提供了接口，我们可以直接实现接口表示这个Bean是一个工厂Bean：

```java
@Component
public class StudentFactory implements FactoryBean<Student> {
    @Override
    public Student getObject() {   //生产的Bean对象
        return new Student();
    }

    @Override
    public Class<?> getObjectType() {   //生产的Bean类型
        return Student.class;
    }

    @Override
    public boolean isSingleton() {   //生产的Bean是否采用单例模式
        return false;
    }
}
```

实际上跟我们之前在配置文件中编写是一样的，这里就不多说了。

请注意，使用注解虽然可以省事很多，代码也能变得更简洁，但是这并不代表XML配置文件就是没有意义的，它们有着各自的优点，在不同的场景下合理使用，能够起到事半功倍的效果，官方原文：

> Are annotations better than XML for configuring Spring?
>
> The introduction of annotation-based configuration raised the question of whether this approach is “better” than XML. The short answer is “it depends.” The long answer is that each approach has its pros and cons, and, usually, it is up to the developer to decide which strategy suits them better. Due to the way they are defined, annotations provide a lot of context in their declaration, leading to shorter and more concise configuration. However, XML excels at wiring up components without touching their source code or recompiling them. Some developers prefer having the wiring close to the source while others argue that annotated classes are no longer POJOs and, furthermore, that the configuration becomes decentralized and harder to control.
>
> No matter the choice, Spring can accommodate both styles and even mix them together. It is worth pointing out that through its [JavaConfig](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) option, Spring lets annotations be used in a non-invasive way, without touching the target components source code and that, in terms of tooling, all configuration styles are supported by the [Spring Tools for Eclipse](https://spring.io/tools).

至此，关于Spring的IoC基础部分，我们就全部介绍完了。在最后，留给各位小伙伴一个问题，现在有两个类：

```java
@Component
public class Student {
    @Resource
    private Teacher teacher;
}
```

```java
@Component
public class Teacher {
    @Resource
    private Student student;
}
```

这两个类互相需要注入对方的实例对象，这个时候Spring会怎么进行处理呢？如果Bean变成原型模式，Spring又会怎么处理呢？

这个问题我们会在实现原理探究部分进行详细介绍。

***

## Spring高级特性（选学）

**注意：**本部分为选学内容，如果Spring基础部分学的不是很明白，不建议看这一部分，在理解清楚之后再来看也可以，但也不是说以后就不用学了，这些东西在某些项目中可能会用到，你迟早还是要回来补的。

前面我们介绍了Spring的基础部分，我们接着来介绍Spring的一些其他高级特性。

### Bean Aware

在Spring中提供了一些以Aware结尾的接口，实现了Aware接口的bean在被初始化之后，可以获取相应资源。Aware的中文意思为**感知**。简单来说，他就是一个标识，实现此接口的类会获得某些感知能力，Spring容器会在Bean被加载时，根据类实现的感知接口，会调用类中实现的对应感知方法。

比如BeanNameAware之类的以Aware结尾的接口，这个接口获取的资源就是BeanName：

```java
@Component
public class Student implements BeanNameAware {   //我们只需要实现这个接口就可以了

    @Override
    public void setBeanName(String name) {   //Bean在加载的时候，容器就会自动调用此方法，将Bean的名称给到我们
        System.out.println("我在加载阶段获得了Bean名字："+name);
    }
}
```

又比如BeanClassLoaderAware，那么它能够使得我们可以在Bean加载阶段就获取到当前Bean的类加载器：

```java
@Component
public class Student implements BeanClassLoaderAware {

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println(classLoader);
    }
}
```

有关其他的Aware这里就不一一列举了，我们会在后面的实现原理探究部分逐步认识的。

### 任务调度

为了执行某些任务，我们可能需要一些非常规的操作，比如我们希望使用多线程来处理我们的结果或是执行一些定时任务，到达指定时间再去执行。这时我们首先想到的就是创建一个新的线程来处理，或是使用TimerTask来完成定时任务，但是我们有了Spring框架之后，就不用这样了，因为Spring框架为我们提供了更加便捷的方式进行任务调度。

首先我们来看异步任务执行，需要使用Spring异步任务支持，我们需要在配置类上添加`@EnableAsync`注解。

```java
@EnableAsync
@Configuration
@ComponentScan("com.test.bean")
public class MainConfiguration {
}
```

接着我们只需要在需要异步执行的方法上，添加`@Async`注解即可将此方法标记为异步，当此方法被调用时，会异步执行，也就是新开一个线程执行，而不是在当前线程执行。我们来测试一下：

```java
@Component
public class Student {
    public void syncTest() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"我是同步执行的方法，开始...");
        Thread.sleep(3000);
        System.out.println("我是同步执行的方法，结束！");
    }

    @Async
    public void asyncTest() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"我是异步执行的方法，开始...");
        Thread.sleep(3000);
        System.out.println("我是异步执行的方法，结束！");
    }
}
```

现在我们在主方法中分别调用一下试试看：

```java
public static void main(String[] args) throws InterruptedException {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
    Student student = context.getBean(Student.class);
    student.asyncTest();   //异步执行
    student.syncTest();    //同步执行
}
```

可以看到，我们的任务执行结果为：

![image-20221125153110860](https://s2.loli.net/2022/11/25/7VKh3dreROJUTcN.png)

很明显，异步执行的任务并不是在当前线程启动的，而是在其他线程启动的，所以说并不会在当前线程阻塞，可以看到马上就开始执行下一行代码，调用同步执行的任务了。

因此，当我们要将Bean的某个方法设计为异步执行时，就可以直接添加这个注解。但是需要注意，添加此注解要求方法的返回值只能是void或是Future类型才可以（Future类型我们在JUC篇视频教程中有详细介绍）

还有，在使用时，可能还会出现这样的信息：

![image-20221125153426124](https://s2.loli.net/2022/11/25/7RfCIvYAZhMDPJe.png)

虽然出现了这样的信息，但是我们的程序依然可以正常运行，这是因为Spring默认会从容器中选择一个`Executor`类型（这同样是在JUC篇视频教程中介绍的类型）的实例，并使用它来创建线程执行任务，这是Spring推荐的方式，当然，如果没有找到，那么会使用自带的 SimpleAsyncTaskExecutor 处理异步方法调用。

肯定会有小伙伴疑惑，什么情况？！这个方法很明显我们并没有去编写异步执行的逻辑，那么为什么会异步执行呢？这里很明显是同步调用的方法啊。的确，如果这个Bean只是一个简简单单的Student类型的对象，确实做不到。但是它真的只是一个简简单单的Student类型对象吗？

```java
Student student = context.getBean(Student.class);
System.out.println(student.getClass());   //这里我们通过getClass来获取一下类型，你会发现惊喜
```

我们来看看结果：

![image-20221125154133618](https://s2.loli.net/2022/11/25/qUlV5hj96YEupQM.png)

？？？这是什么东西？这实际上Spring帮助我们动态生成的一个代理类，我们原本的类代码已经被修改了，当然，这只是冰山一角，更多的内容，我们还会再AOP面向切片部分中继续为大家进行介绍，能做到这样的操作，这其实都是AOP的功劳。

看完了异步任务，我们接着来看定时任务，定时任务其实就是指定在哪个时候再去执行，在JavaSE阶段我们使用过TimerTask来执行定时任务。Spring中的定时任务是全局性质的，当我们的Spring程序启动后，那么定时任务也就跟着启动了，我们可以在配置类上添加`@EnableScheduling`注解：

```java
@EnableScheduling
@Configuration
@ComponentScan("com.test.bean")
public class MainConfiguration {
}
```

接着我们可以直接在配置类里面编写定时任务，把我们要做的任务写成方法，并添加`@Scheduled`注解：

```java
@Scheduled(fixedRate = 2000)   //单位依然是毫秒，这里是每两秒钟打印一次
public void task(){
    System.out.println("我是定时任务！"+new Date());
}
```

![image-20221125155352390](https://s2.loli.net/2022/11/25/aGv7f3eBXPsFdYr.png)

我们注意到`@Scheduled`中有很多参数，我们需要指定'cron', 'fixedDelay(String)', or 'fixedRate(String)'的其中一个，否则无法创建定时任务，他们的区别如下：

- fixedDelay：在上一次定时任务执行完之后，间隔多久继续执行。
- fixedRate：无论上一次定时任务有没有执行完成，两次任务之间的时间间隔。
- cron：如果嫌上面两个不够灵活，你还可以使用cron表达式来指定任务计划。

这里简单讲解一下cron表达式：https://blog.csdn.net/sunnyzyq/article/details/98597252

### 监听器

监听器对我们来说也是一个比较陌生的概念，那么何谓监听呢？

监听实际上就是等待某个事件的触发，当事件触发时，对应事件的监听器就会被通知，如果你学习过Java Swing篇视频教程，应该会深有体会，监听器可是很关键的，只不过在Spring中用的不是很频繁罢了。但是这里还是要简单介绍一下：

```java
@Component
public class TestListener implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println(event.getApplicationContext());   //可以直接通过事件获取到事件相关的东西
    }
}
```

要编写监听器，我们只需要让Bean继承ApplicationListener就可以了，并且将类型指定为对应的Event事件，这样，当发生某个事件时就会通知我们，比如ContextRefreshedEvent，这个事件会在Spring容器初始化完成会触发一次：

![image-20221125155804255](https://s2.loli.net/2022/11/25/MZN3eohGmV17vUJ.png)

是不是感觉挺智能的？Spring内部有各种各样的事件，当然我们也可以自己编写事件，然后在某个时刻发布这个事件到所有的监听器：

```java
public class TestEvent extends ApplicationEvent {   //自定义事件需要继承ApplicationEvent
    public TestEvent(Object source) {
        super(source);
    }
}
```

```java
@Component
public class TestListener implements ApplicationListener<TestEvent> {
    @Override
    public void onApplicationEvent(TestEvent event) {
        System.out.println("发生了一次自定义事件，成功监听到！");
    }
}
```

比如现在我们希望在定时任务中每秒钟发生一次这个事件：

```java
@Component
public class TaskComponent  implements ApplicationEventPublisherAware {  
  	//要发布事件，需要拿到ApplicationEventPublisher，这里我们通过Aware在初始化的时候拿到
  	//实际上我们的ApplicationContext就是ApplicationEventPublisher的实现类，这里拿到的就是
  	//我们创建的ApplicationContext对象
    ApplicationEventPublisher publisher;

    @Scheduled(fixedRate = 1000)   //一秒一次
    public void task(){
      	//直接通过ApplicationEventPublisher的publishEvent方法发布事件
      	//这样，所有这个事件的监听器，都会监听到这个事件发生了
        publisher.publishEvent(new TestEvent(this));
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }
}
```

此时，发布事件旁边出现了图标，说明就可以了：

![image-20221125161224037](https://s2.loli.net/2022/11/25/fDxYEGPWdyMt7XF.png)

我们可以点击这个图标快速跳转到哪里监听了这个事件，IDEA这些细节做的还是挺好的。我们来看看运行结果吧：

![image-20221125161125557](https://s2.loli.net/2022/11/25/FKy1jBx3MJvVdDN.png)

是不是感觉好像也没那么难，这套机制其实还挺简单的，这样，我们就实现了自定义事件发布和监听。

***

## SpringEL表达式

SpEL 是一种强大，简洁的装配 Bean 的方式，它可以通过运行期间执行的表达式将值装配到我们的属性或构造函数当中，更可以调用 JDK 中提供的静态常量，获取外部 Properties 文件中的的配置。

### 外部属性注入

有些时候，我们甚至可以将一些外部配置文件中的配置进行读取，并完成注入。

我们需要创建以`.properties`结尾的配置文件，这种配置文件格式很简单，类似于Map，需要一个Key和一个Value，中间使用等号进行连接，这里我们在resource目录下创建一个`test.properties`文件：

```properties
test.name=只因
```

这样，Key就是`test.name`，Value就是`只因`，我们可以通过一个注解直接读取到外部配置文件中对应的属性值，首先我们需要引入这个配置文件，我们可以在配置类上添加`@PropertySource`注解：

```java
@Configuration
@ComponentScan("com.test.bean")
@PropertySource("classpath:test.properties")   //注意，类路径下的文件名称需要在前面加上classpath:
public class MainConfiguration{
    
}
```

接着，我们就可以开始快乐的使用了，我们可以使用 @Value 注解将外部配置文件中的值注入到任何我们想要的位置，就像我们之前使用@Resource自动注入一样：

```java
@Component
public class Student {
    @Value("${test.name}")   //这里需要在外层套上 ${ }
    private String name;   //String会被自动赋值为配置文件中对应属性的值

    public void hello(){
        System.out.println("我的名字是："+name);
    }
}
```

`@Value`中的`${...}`表示占位符，它会读取外部配置文件的属性值装配到属性中，如果配置正确没问题的话，这里甚至还会直接显示对应配置项的值：

![image-20221125164854022](https://s2.loli.net/2022/11/25/HDZ4l3tcreoOGh8.png)

我们来测试一下吧：

![image-20221125165145332](https://s2.loli.net/2022/11/25/g5tBKW4Sm9lXnrR.png)

如果遇到乱码的情况，请将配置文件的编码格式切换成UTF-8（可以在IDEA设置中进行配置）然后在@PropertySource注解中添加属性 encoding = "UTF-8" 这样就正常了，当然，其实一般情况下也很少会在配置文件中用到中文。

除了在字段上进行注入之外，我们也可以在需要注入的方法中使用：

```java
@Component
public class Student {
    private final String name;

  	//构造方法中的参数除了被自动注入外，我们也可以选择使用@Value进行注入
    public Student(@Value("${test.name}") String name){
        this.name = name;
    }

    public void hello(){
        System.out.println("我的名字是："+name);
    }
}
```

当然，如果我们只是想简单的注入一个常量值，也可以直接填入固定值：

```java
private final String name;
public Student(@Value("10") String name){   //只不过，这里都是常量值了，我干嘛不直接写到代码里呢
    this.name = name;
}
```

当然，@Value 的功能还远不止这些，配合SpringEL表达式，能够实现更加强大的功能。

### SpEL简单使用

Spring官方为我们提供了一套非常高级SpEL表达式，通过使用表达式，我们可以更加灵活地使用Spring框架。

首先我们来看看如何创建一个SpEL表达式：

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");  //使用parseExpression方法来创建一个表达式
System.out.println(exp.getValue());   //表达式最终的运算结果可以通过getValue()获取
```

这里得到的就是一个很简单的 Hello World 字符串，字符串使用单引号囊括，SpEL是具有运算能力的。

我们可以像写Java一样，对这个字符串进行各种操作，比如调用方法之类的：

```java
Expression exp = parser.parseExpression("'Hello World'.toUpperCase()");   //调用String的toUpperCase方法
System.out.println(exp.getValue());
```

![image-20221125173157008](https://s2.loli.net/2022/11/25/PZmheYn5EVTvURN.png)

不仅能调用方法、还可以访问属性、使用构造方法等，是不是感觉挺牛的，居然还能这样玩。

对于Getter方法，我们可以像访问属性一样去使用：

```java
//比如 String.getBytes() 方法，就是一个Getter，那么可以写成 bytes
Expression exp = parser.parseExpression("'Hello World'.bytes");
System.out.println(exp.getValue());
```

表达式可以不止一级，我们可以多级调用：

```java
Expression exp = parser.parseExpression("'Hello World'.bytes.length");   //继续访问数组的length属性
System.out.println(exp.getValue());
```

是不是感觉挺好玩的？我们继续来试试看构造方法，其实就是写Java代码，只是可以写成这种表达式而已：

```java
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()");
System.out.println(exp.getValue());
```

![image-20221125173157008](https://s2.loli.net/2022/11/25/PZmheYn5EVTvURN.png)

它甚至还支持根据特定表达式，从给定对象中获取属性出来：

```java
@Component
public class Student {
    private final String name;
    public Student(@Value("${test.name}") String name){
        this.name = name;
    }

    public String getName() {    //比如下面要访问name属性，那么这个属性得可以访问才行，访问权限不够是不行的
        return name;
    }
}
```

```java
Student student = context.getBean(Student.class);
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name");
System.out.println(exp.getValue(student));    //直接读取对象的name属性
```

拿到对象属性之后，甚至还可以继续去处理：

```java
Expression exp = parser.parseExpression("name.bytes.length");   //拿到name之后继续getBytes然后length
```

除了获取，我们也可以调用表达式的setValue方法来设定属性的值：

```java
Expression exp = parser.parseExpression("name");
exp.setValue(student, "刻师傅");   //同样的，这个属性得有访问权限且能set才可以，否则会报错
```

除了属性调用，我们也可以使用运算符进行各种高级运算：

```java
Expression exp = parser.parseExpression("66 > 77");   //比较运算
System.out.println(exp.getValue());
```

```java
Expression exp = parser.parseExpression("99 + 99 * 3");   //算数运算
System.out.println(exp.getValue());
```

对于那些需要导入才能使用的类，我们需要使用一个特殊的语法：

```java
Expression exp = parser.parseExpression("T(java.lang.Math).random()");   //由T()囊括，包含完整包名+类名
//Expression exp = parser.parseExpression("T(System).nanoTime()");   //默认导入的类可以不加包名
System.out.println(exp.getValue());
```

### 集合操作相关语法

现在我们的类中存在一些集合类：

```java
@Component
public class Student {
    public Map<String, String> map = Map.of("test", "你干嘛");
    public List<String> list = List.of("AAA", "BBB", "CCC");
}
```

我们可以使用SpEL快速取出集合中的元素：

```java
Expression exp = parser.parseExpression("map['test']");  //对于Map这里映射型，可以直接使用map[key]来取出value
System.out.println(exp.getValue(student));
```

```java
Expression exp = parser.parseExpression("list[2]");   //对于List、数组这类，可以直接使用[index]
System.out.println(exp.getValue(student));
```

我们也可以快速创建集合：

```java
Expression exp = parser.parseExpression("{5, 2, 1, 4, 6, 7, 0, 3, 9, 8}"); //使用{}来快速创建List集合
List value = (List) exp.getValue();
value.forEach(System.out::println);
```

```java
Expression exp = parser.parseExpression("{{1, 2}, {3, 4}}");   //它是支持嵌套使用的
```

```java
//创建Map也很简单，只需要key:value就可以了，怎么有股JSON味
Expression exp = parser.parseExpression("{name: '小明', info: {address: '北京市朝阳区', tel: 10086}}");
System.out.println(exp.getValue());
```

你以为就这么简单吗，我们还可以直接根据条件获取集合中的元素：

```java
@Component
public class Student {
    public List<Clazz> list = List.of(new Clazz("高等数学", 4));

    public record Clazz(String name, int score){ }
}
```

```java
//现在我们希望从list中获取那些满足我们条件的元素，并组成一个新的集合，我们可以使用.?运算符
Expression exp = parser.parseExpression("list.?[name == '高等数学']");
System.out.println(exp.getValue(student));
```

```java
Expression exp = parser.parseExpression("list.?[score > 3]");   //选择学分大于3分的科目
System.out.println(exp.getValue(student));
```

我们还可以针对某个属性创建对应的投影集合：

```java
Expression exp = parser.parseExpression("list.![name]");   //使用.!创建投影集合，这里创建的时课程名称组成的新集合
System.out.println(exp.getValue(student));
```

![image-20221130153142677](https://s2.loli.net/2022/11/30/yLNHPJnWkoR3Cb2.png)

我们接着来介绍安全导航运算符，安全导航运算符用于避免NullPointerException，它来自Groovy语言。通常，当您有对对象的引用时，您可能需要在访问对象的方法或属性之前验证它是否为空。为了避免这种情况，安全导航运算符返回null而不是抛出异常。以下示例显示了如何使用安全导航运算符：

```java
Expression exp = parser.parseExpression("name.toUpperCase()");   //如果Student对象中的name属性为null
System.out.println(exp.getValue(student));
```

![image-20221130150723018](https://s2.loli.net/2022/11/30/dojeP5kYcM7KHiv.png)

当遇到null时很不方便，我们还得写判断：

```java
if(student.name != null)
    System.out.println(student.name.toUpperCase());
```

Java 8之后能这样写：

```java
Optional.ofNullable(student.name).ifPresent(System.out::println);
```

但是你如果写过Kotlin：
```kotlin
println(student.name?.toUpperCase());
```

类似于这种判空问题，我们就可以直接使用安全导航运算符，SpEL也支持这种写法：

```java
Expression exp = parser.parseExpression("name?.toUpperCase()");
System.out.println(exp.getValue(student));
```

当遇到空时，只会得到一个null，而不是直接抛出一个异常：

![image-20221130150654287](https://s2.loli.net/2022/11/30/tOf3LFsWE4H8BVc.png)

我们可以将SpEL配合 @Value 注解或是xml配置文件中的value属性使用，比如XML中可以这样写：

```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>
</bean>
```

或是使用注解开发：

```java
public class FieldValueTestBean {
    @Value("#{ systemProperties['user.region'] }")
    private String defaultLocale;
}
```

这样，我们有时候在使用配置文件中的值时，就能进行一些简单的处理了。

有关更多详细语法教程，请前往：https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions-language-ref

***

## AOP面向切片

又是一个听起来很高大上的名词，AOP（Aspect Oriented Programming）思想实际上就是：在运行时，动态地将代码切入到类的指定方法、指定位置上。也就是说，我们可以使用AOP来帮助我们在方法执行前或执行之后，做一些额外的操作，实际上，它就是代理！

通过AOP我们可以在保证原有业务不变的情况下，添加额外的动作，比如我们的某些方法执行完成之后，需要打印日志，那么这个时候，我们就可以使用AOP来帮助我们完成，它可以批量地为这些方法添加动作。可以说，它相当于将我们原有的方法，在不改变源代码的基础上进行了增强处理。

![image-20221130155358974](https://s2.loli.net/2022/11/30/sJbSrgiAxa8Vhcv.png)

相当于我们的整个业务流程，被直接斩断，并在断掉的位置添加了一个额外的操作，再连接起来，也就是在一个切点位置插入内容。它的原理实际上就是通过动态代理机制实现的，我们在JavaWeb阶段已经给大家讲解过动态代理了。不过Spring底层并不是使用的JDK提供的动态代理，而是使用的第三方库实现，它能够以父类的形式代理，而不仅仅是接口。

### 使用配置实现AOP

在开始之前，我们先换回之前的XML配置模式，之后也会给大家讲解如何使用注解完成AOP操作，注意这里我们还加入了一些新的AOP相关的约束进来，建议直接CV下面的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
</beans>
```

Spring是支持AOP编程的框架之一（实际上它整合了AspectJ框架的一部分），要使用AOP我们需要先导入一个依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>6.0.10</version>
</dependency>
```

那么，如何使用AOP呢？首先我们要明确，要实现AOP操作，我们需要知道这些内容：

1. 需要切入的类，类的哪个方法需要被切入
2. 切入之后需要执行什么动作
3. 是在方法执行前切入还是在方法执行后切入
4. 如何告诉Spring需要进行切入

比如现在我们希望对这个学生对象的`study`方法进行增强，在不修改源代码的情况下，增加一些额外的操作：

```java
public class Student {
    public void study(){
        System.out.println("室友还在打游戏，我狠狠的学Java，太爽了"); 
      	//现在我们希望在这个方法执行完之后，打印一些其他的内容，在不修改原有代码的情况下，该怎么做呢？
    }
}
```

```java
<bean class="org.example.entity.Student"/>
```

那么我们按照上面的流程，依次来看，首先需要解决的问题是，找到需要切入的类，很明显，就是这个Student类，我们要切入的是这个`study`方法。

第二步，我们切入之后要做什么呢？这里我们直接创建一个新的类，并将要执行的操作写成一个方法：

```java
public class StudentAOP {
  	//这个方法就是我们打算对其进行的增强操作
    public void afterStudy() {
        System.out.println("为什么毕业了他们都继承家产，我还倒给他们打工，我努力的意义在哪里...");
    }
}
```

注意这个类也得注册为Bean才可以：

```xml
<bean id="studentAOP" class="org.example.entity.StudentAOP"/>
```

第三步，我们要明确这是在方法执行之前切入还是执行之后切入，很明显，按照上面的要求，我们需要执行之后进行切入。

第四步，最关键的来了，我们怎么才能告诉Spring我们要进行切入操作呢？这里我们需要在配置文件中进行AOP配置：

```xml
<aop:config>

</aop:config>
```

接着我们需要添加一个新的切点，首先填写ID，这个随便起都可以：

```xml
<aop:pointcut id="test" expression=""/>
```

然后就是通过后面的`expression`表达式来选择到我们需要切入的方法，这个表达式支持很多种方式进行选择，Spring AOP支持以下AspectJ切点指示器（PCD）用于表达式：

- `execution`：用于匹配方法执行连接点。这是使用Spring AOP时使用的主要点切割指示器。
- `within`：限制匹配到某些类型的连接点（使用Spring AOP时在匹配类型中声明的方法的执行）。
- `this`：限制与连接点匹配（使用Spring AOP时方法的执行），其中bean引用（Spring AOP代理）是给定类型的实例。
- `target`：限制匹配连接点（使用Spring AOP时方法的执行），其中目标对象（正在代理的应用程序对象）是给定类型的实例。
- `args`：限制与连接点匹配（使用Spring AOP时方法的执行），其中参数是给定类型的实例。
- `@target`：限制匹配连接点（使用Spring AOP时方法的执行），其中执行对象的类具有给定类型的注释。
- `@args`：限制匹配到连接点（使用Spring AOP时方法的执行），其中传递的实际参数的运行时类型具有给定类型的注释。
- `@within`：限制与具有给定注释的类型中的连接点匹配（使用Spring AOP时在带有给定注释的类型中声明的方法的执行）。
- `@annotation`：与连接点主体（在Spring AOP中运行的方法）具有给定注释的连接点匹配的限制。

更多详细内容请查阅：https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-pointcuts-designators

其中，我们主要学习的`execution`填写格式如下：

```xml
修饰符 包名.类名.方法名称(方法参数)
```

- 修饰符：public、protected、private、包括返回值类型、static等等（使用*代表任意修饰符）
- 包名：如com.test（* 代表全部，比如com.*代表com包下的全部包）
- 类名：使用*也可以代表包下的所有类
- 方法名称：可以使用*代表全部方法
- 方法参数：填写对应的参数即可，比如(String, String)，也可以使用*来代表任意一个参数，使用..代表所有参数。

也可以使用其他属性来进行匹配，比如`@annotation`可以用于表示标记了哪些注解的方法被切入，这里我们就只是简单的执行，所以说只需要这样写就可以了：

```xml
<aop:pointcut id="test" expression="execution(* org.example.entity.Student.study())"/>
```

这样，我们就指明了需要切入的方法，然后就是将我们的增强方法，我们在里面继续添加`aop:aspect`标签，并使用`ref`属性将其指向我们刚刚注册的AOP类Bean：

```xml
<aop:config>
    <aop:pointcut id="test" expression="execution(* org.example.entity.Student.study())"/>
    <aop:aspect ref="studentAOP">
				
    </aop:aspect>
</aop:config>
```

接着就是添加后续动作了，当然，官方支持的有多种多样的，比如执行前、执行后、抛出异常后、方法返回后等等：

![image-20221216155201849](https://s2.loli.net/2022/12/16/uopJ9KyqMvQSwi4.png)

其中around方法为环绕方法，自定义度会更高，我们会在稍后介绍。这里我们按照上面的要求，直接添加后续动作，注意需要指明生效的切点：

```xml
<aop:aspect ref="studentAOP">
  	<!--     method就是我们的增强方法，pointcut-ref指向我们刚刚创建的切点     -->
    <aop:after method="afterStudy" pointcut-ref="test"/>
</aop:aspect>
```

这样，我们就成功配置好了，配置正确会在旁边出现图标：

![image-20221216155551779](https://s2.loli.net/2022/12/16/hBaSmuovMzp5iIn.png)

我们来试试看吧：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    Student bean = context.getBean(Student.class);
    bean.study();
}
```

结果如下：

![image-20221216155616952](https://s2.loli.net/2022/12/16/JlvLe9rgQw2pbXo.png)

可以看到在我们原本的方法执行完成之后，它还继续执行了我们的增强方法，这实际上就是动态代理做到的，实现在不修改原有代码的基础上，对方法的调用进行各种增强，在之后的SpringMVC学习中，我们甚至可以使用它来快速配置访问日志打印。

前面我们说了，AOP是基于动态代理实现的，所以说我们如果直接获取Bean的类型，会发现不是原本的类型了：

```java
Student bean = context.getBean(Student.class);
System.out.println(bean.getClass());
```

![image-20221216172042146](https://s2.loli.net/2022/12/16/8lsiRj6Q9eTLhSI.png)

这里其实是Spring通过CGLib为我们生成的动态代理类，也就不难理解为什么调用方法会直接得到增强之后的结果了。包括我们前面讲解Spring的异步任务调度时，为什么能够直接实现异步，其实就是利用了AOP机制实现的方法增强。

虽然这些功能已经非常强大了，但是仅仅只能简单的切入还是不能满足一些需求，在某些情况下，我们可以需求方法执行的一些参数，比如方法执行之后返回了什么，或是方法开始之前传入了什么参数等等，现在我们修改一下Student中`study`方法的参数：

```java
public class Student {
    public void study(String str){  //现在方法有一个String类型的参数
        System.out.println("都别学Java了，根本找不到工作，快去卷"+str);
    }
}
```

我们希望在增强的方法中也能拿到这个参数，然后进行处理：

```java
public class StudentAOP {
    public void afterStudy() {
      	//这个str参数我们该从哪里拿呢？
        System.out.println("学什么"+str+"，Rust天下第一！");
    }
}
```

这个时候，我们可以为我们切入的方法添加一个JoinPoint参数，通过此参数就可以快速获取切点位置的一些信息：

```java
public void afterStudy(JoinPoint point) {   //JoinPoint实例会被自动传入
    //这里我们直接通过getArgs()返回的参数数组获取第1个参数
    System.out.println("学什么"+point.getArgs()[0]+"，Rust天下第一！");
}
```

接着我们修改一下刚刚的AOP配置（因为方法参数有变动）看看结果吧：

```xml
<aop:pointcut id="test" expression="execution(* org.example.entity.Student.study(String))"/>
```

现在我们来测试一下：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    Student bean = context.getBean(Student.class);
    bean.study("PHP");
}
```

![image-20221216160501469](https://s2.loli.net/2022/12/16/NrZA49JvpgEyL2O.png)

是不是感觉大部分功能都可以通过AOP来完成了？

我们接着来看自定义度更高的环绕方法，现在我们希望在方法执行前和执行后都加入各种各样的动作，如果还是一个一个切点写，有点太慢了，能不能直接写一起呢，此时我们就可以使用环绕方法。

环绕方法相当于完全代理了此方法，它完全将此方法包含在中间，需要我们手动调用才可以执行此方法，并且我们可以直接获取更多的参数：

```java
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("方法开始之前");
    Object value = joinPoint.proceed();   //调用process方法来执行被代理的原方法，如果有返回值，可以使用value接收
    System.out.println("方法执行完成，结果为："+value);
  	return value;
}
```

注意，如果代理方法存在返回值，那么环绕方法也需要有一个返回值，通过`proceed`方法来执行代理的方法，也可以修改参数之后调用`proceed(Object[])`，使用我们给定的参数再去执行：

```java
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("方法开始之前");
    String arg = joinPoint.getArgs()[0] + "伞兵一号";
    Object value = joinPoint.proceed(new Object[]{arg});
    System.out.println("方法执行完成，结果为："+value);
    return value;
}
```

这里我们还是以`study`方法为例，现在我们希望在调用前修改这个方法传入的参数值，改成我们自己的，然后在调用之后对返回值结果也进行处理：

```java
public String study(String str){
    if(str.equals("Java"))
        System.out.println("我的梦想是学Java");
    else {
        System.out.println("我就要学Java，不要修改我的梦想！");
        str = "Java";
    }
    return str;
}
```

现在我们编写一个环绕方法，对其进行全方面处理：

```java
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("我是她的家长，他不能学Java，必须学Rust，这是为他好");
    Object value = joinPoint.proceed(new Object[]{"Rust"});
    if(value.equals("Java")) {
        System.out.println("听话，学Rust以后进大厂！");
        value = "Rust";
    }
    return value;
}
```

同样的，因为方法变动了，现在我们去修改一下我们的AOP配置：

```xml
<aop:pointcut id="test" expression="execution(* org.example.entity.Student.study(String))"/>
<aop:aspect ref="studentAOP">
    <aop:around method="around" pointcut-ref="test"/>
</aop:aspect>
```

![image-20221216162003675](https://s2.loli.net/2022/12/16/FPwQjRvsDgTnoWx.png)

细心的小伙伴可能会发现，环绕方法的图标是全包的，跟我们之前的图标不太一样。

现在我们来试试看吧：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    Student bean = context.getBean(Student.class);
    System.out.println("已报名："+bean.study("Java"));
}
```

![image-20221216161855358](https://s2.loli.net/2022/12/16/pGl7n8qboe4tuJf.png)

这样，我们就实现了环绕方法，通过合理利用AOP带来的便捷，可以使得我们的代码更加清爽和优美。这里介绍一下 AOP 领域中的特性术语，防止自己下来看不懂文章：

- 通知（Advice）: AOP 框架中的增强处理，通知描述了切面何时执行以及如何执行增强处理，也就是我们上面编写的方法实现。
- 连接点（join point）: 连接点表示应用执行过程中能够插入切面的一个点，这个点可以是方法的调用、异常的抛出，实际上就是我们在方法执行前或是执行后需要做的内容。
- 切点（PointCut）: 可以插入增强处理的连接点，可以是方法执行之前也可以方法执行之后，还可以是抛出异常之类的。
- 切面（Aspect）: 切面是通知和切点的结合，我们之前在xml中定义的就是切面，包括很多信息。
- 引入（Introduction）：引入允许我们向现有的类添加新的方法或者属性。
- 织入（Weaving）: 将增强处理添加到目标对象中，并创建一个被增强的对象，我们之前都是在将我们的增强处理添加到目标对象，也就是织入（这名字挺有文艺范的）

### 使用接口实现AOP

前面我们介绍了如何使用xml配置一个AOP操作，这节课我们来看看如何使用Advice实现AOP。

它与我们之前学习的动态代理更接近一些，比如在方法开始执行之前或是执行之后会去调用我们实现的接口，首先我们需要将一个类实现Advice接口，只有实现此接口，才可以被通知，比如我们这里使用`MethodBeforeAdvice`表示是一个在方法执行之前的动作：

```java
public class StudentAOP implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("通过Advice实现AOP");
    }
}
```

我们发现，方法中包括了很多的参数，其中args代表的是方法执行前得到的实参列表，还有target表示执行此方法的实例对象。运行之后，效果和之前是一样的，但是在这里我们就可以快速获取到更多信息。还是以简单的study方法为例：

```java
public class Student {
    public void study(){
        System.out.println("我是学习方法！");
    }
}
```

```xml
<bean id="student" class="org.example.entity.Student"/>
<bean id="studentAOP" class="org.example.entity.StudentAOP"/>
<aop:config>
    <aop:pointcut id="test" expression="execution(* org.example.entity.Student.study())"/>
  	<!--  这里只需要添加我们刚刚写好的advisor就可以了，注意是Bean的名字  -->
    <aop:advisor advice-ref="studentAOP" pointcut-ref="test"/>
</aop:config>
```

我们来测试一下吧：

![image-20221216164110367](https://s2.loli.net/2022/12/16/ofducpb2mLh9XHi.png)

除了此接口以外，还有其他的接口，比如`AfterReturningAdvice`就需要实现一个方法执行之后的操作：

```java
public class StudentAOP implements MethodBeforeAdvice, AfterReturningAdvice {
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("通过Advice实现AOP");
    }

    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println("我是方法执行之后的结果，方法返回值为："+returnValue);
    }
}
```

因为使用的是接口，就非常方便，直接写一起，配置文件都不需要改了：

![image-20221216164242506](https://s2.loli.net/2022/12/16/DUZzqaBSiJKNv8j.png)

我们也可以使用MethodInterceptor（同样也是Advice的子接口）进行更加环绕那样的自定义的增强，它用起来就真的像代理一样，例子如下：

```java
public class Student {
    public String study(){
        System.out.println("我是学习方法！");
        return "lbwnb";
    }
}
```

```java
public class StudentAOP implements MethodInterceptor {   //实现MethodInterceptor接口
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {  //invoke方法就是代理方法
        Object value = invocation.proceed();   //跟之前一样，需要手动proceed()才能调用原方法
        return value+"增强";
    }
}
```

我们来看看结果吧：

![image-20221216173211310](https://s2.loli.net/2022/12/16/ARcUW2mJrn7Y6f9.png)

使用起来还是挺简单的。

### 使用注解实现AOP

接着我们来看看如何使用注解实现AOP操作，现在变回我们之前的注解开发，首先我们需要在主类添加`@EnableAspectJAutoProxy`注解，开启AOP注解支持：

```java
@EnableAspectJAutoProxy
@ComponentScan("org.example.entity")
@Configuration
public class MainConfiguration {
}
```

还是熟悉的玩法，类上直接添加`@Component`快速注册Bean：

```java
@Component
public class Student {
    public void study(){
        System.out.println("我是学习方法！");
    }
}
```

接着我们需要在定义AOP增强操作的类上添加`@Aspect`注解和`@Component`将其注册为Bean即可，就像我们之前在配置文件中也要将其注册为Bean那样：

```java
@Aspect
@Component
public class StudentAOP {

}
```

接着，我们可以在里面编写增强方法，并将此方法添加到一个切点中，比如我们希望在Student的study方法执行之前执行我们的`before`方法：

```java
public void before(){
    System.out.println("我是之前执行的内容！");
}
```

那么只需要添加@Before注解即可：

```java
@Before("execution(* org.example.entity.Student.study())")  //execution写法跟之前一样
public void before(){
    System.out.println("我是之前执行的内容！");
}
```

这样，这个方法就会在指定方法执行之前执行了，是不是感觉比XML配置方便多了。我们来测试一下：

```java
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
    Student bean = context.getBean(Student.class);
    bean.study();
}
```

![image-20221216165625372](https://s2.loli.net/2022/12/16/KpiXcdNt7BglYQh.png)

同样的，我们可以为其添加`JoinPoint`参数来获取切入点信息，使用方法跟之前一样：

```java
@Before("execution(* org.example.entity.Student.study())")
public void before(JoinPoint point){
    System.out.println("参数列表："+ Arrays.toString(point.getArgs()));
    System.out.println("我是之前执行的内容！");
}
```

为了更方便，我们还可以直接将参数放入，比如：

```java
public void study(String str){
    System.out.println("我是学习方法！");
}
```

使用命名绑定模式，可以快速得到原方法的参数：

```java
@Before(value = "execution(* org.example.entity.Student.study(..)) && args(str)", argNames = "str")
//命名绑定模式就是根据下面的方法参数列表进行匹配
//这里args指明参数，注意需要跟原方法保持一致，然后在argNames中指明
public void before(String str){
    System.out.println(str);   //可以快速得到传入的参数
    System.out.println("我是之前执行的内容！");
}
```

除了@Before，还有很多可以直接使用的注解，比如@AfterReturning、@AfterThrowing等，比如@AfterReturning：

```java
public String study(){
    System.out.println("我是学习方法！");
    return "lbwnb";
}
```

```java
@AfterReturning(value = "execution(* org.example.entity.Student.study())", argNames = "returnVal", returning = "returnVal")   //使用returning指定接收方法返回值的参数returnVal
public void afterReturn(Object returnVal){
    System.out.println("返回值是："+returnVal);
}
```

同样的，环绕也可以直接通过注解声明：

```java
@Around("execution(* com.test.bean.Student.test(..))")
public Object around(ProceedingJoinPoint point) throws Throwable {
    System.out.println("方法执行之前！");
    Object val = point.proceed();
    System.out.println("方法执行之后！");
    return val;
}
```

实际上，无论是使用注解或是XML配置，我们要做的流程都是一样的，在之后的学习中，我们还会遇到更多需要使用AOP的地方。

***

## 数据库框架整合

学习了Spring之后，我们已经了解如何将一个类作为Bean交由IoC容器管理，这样，我们就可以通过更方便的方式来使用Mybatis框架，我们可以直接把SqlSessionFactory、Mapper交给Spring进行管理，并且可以通过注入的方式快速地使用它们。

因此，我们要学习一下如何将Mybatis与Spring进行整合，那么首先，我们需要在之前知识的基础上继续深化学习。

### 了解数据源

在之前，我们如果需要创建一个JDBC的连接，那么必须使用`DriverManager.getConnection()`来创建连接，连接建立后，我们才可以进行数据库操作。而学习了Mybatis之后，我们就不用再去使用`DriverManager`为我们提供连接对象，而是直接使用Mybatis为我们提供的`SqlSessionFactory`工具类来获取对应的`SqlSession`通过会话对象去操作数据库。

那么，它到底是如何封装JDBC的呢？我们可以试着来猜想一下，会不会是Mybatis每次都是帮助我们调用`DriverManager`来实现的数据库连接创建？我们可以看看Mybatis的源码：

```java
public SqlSession openSession(boolean autoCommit) {
    return this.openSessionFromDataSource(this.configuration.getDefaultExecutorType(), (TransactionIsolationLevel)null, autoCommit);
}
```

在通过`SqlSessionFactory`调用`openSession`方法之后，它调用了内部的一个私有的方法`openSessionFromDataSource`，我们接着来看，这个方法里面定义了什么内容：

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;

    DefaultSqlSession var8;
    try {
        //获取当前环境（由配置文件映射的对象实体）
        Environment environment = this.configuration.getEnvironment();
      	//事务工厂（暂时不提，下一板块讲解）
        TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
      	//配置文件中：<transactionManager type="JDBC"/>
      	//生成事务（根据我们的配置，会默认生成JdbcTransaction），这里是关键，我们看到这里用到了environment.getDataSource()方法
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      	//执行器，包括全部的数据库操作方法定义，本质上是在使用执行器操作数据库，需要传入事务对象
        Executor executor = this.configuration.newExecutor(tx, execType);
      	//封装为SqlSession对象
        var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
    } catch (Exception var12) {
        this.closeTransaction(tx);
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
    } finally {
        ErrorContext.instance().reset();
    }

    return var8;
}
```

也就是说，我们的数据源配置信息，存放在了`Transaction`对象中，那么现在我们只需要知道执行器到底是如何执行SQL语句的，我们就知道到底如何创建`Connection`对象了，这时就需要获取数据库的链接信息了，那么我们来看看，这个`DataSource`到底是个什么：

```java
public interface DataSource  extends CommonDataSource, Wrapper {

  Connection getConnection() throws SQLException;

  Connection getConnection(String username, String password)
    throws SQLException;
}
```

我们发现，它是在`javax.sql`定义的一个接口，它包括了两个方法，都是用于获取连接的。因此，现在我们可以断定，并不是通过之前`DriverManager`的方法去获取连接了，而是使用`DataSource`的实现类来获取的，因此，也就正式引入到我们这一节的话题了：

> 数据库链接的建立和关闭是极其耗费系统资源的操作，通过DriverManager获取的数据库连接，一个数据库连接对象均对应一个物理数据库连接，每次操作都打开一个物理连接，使用完后立即关闭连接，频繁的打开、关闭连接会持续消耗网络资源，造成整个系统性能的低下。

因此，JDBC为我们定义了一个数据源的标准，也就是`DataSource`接口，告诉数据源数据库的连接信息，并将所有的连接全部交给数据源进行集中管理，当需要一个`Connection`对象时，可以向数据源申请，数据源会根据内部机制，合理地分配连接对象给我们。

一般比较常用的`DataSource`实现，都是采用池化技术，就是在一开始就创建好N个连接，这样之后使用就无需再次进行连接，而是直接使用现成的`Connection`对象进行数据库操作。

![image-20221217134119558](https://s2.loli.net/2022/12/17/rk4mcdvYn6osOLW.png)

当然，也可以使用传统的即用即连的方式获取`Connection`对象，Mybatis为我们提供了几个默认的数据源实现，我们之前一直在使用的是官方的默认配置，也就是池化数据源：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${驱动类（含包名）}"/>
                <property name="url" value="${数据库连接URL}"/>
                <property name="username" value="${用户名}"/>
                <property name="password" value="${密码}"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

这里的`type`属性一共三个选项：

- UNPOOLED   不使用连接池的数据源
- POOLED    使用连接池的数据源
- JNDI     使用JNDI实现的数据源

### 解读Mybatis数据源实现（选学）

前面我们介绍了DataSource数据源，那么我们就来看看，Mybatis到底是怎么实现的。我们先来看看，不使用池化的数据源实现，它叫做`UnpooledDataSource`，我们来看看源码：

```java
public class UnpooledDataSource implements DataSource {
    private ClassLoader driverClassLoader;
    private Properties driverProperties;
    private static Map<String, Driver> registeredDrivers = new ConcurrentHashMap();
    private String driver;
    private String url;
    private String username;
    private String password;
    private Boolean autoCommit;
    private Integer defaultTransactionIsolationLevel;
    private Integer defaultNetworkTimeout;
  	...
```

首先这个类中定义了很多的成员，包括数据库的连接信息、数据库驱动信息、事务相关信息等。我们接着来看，它是如何实现`DataSource`中提供的接口方法的：

```java
public Connection getConnection() throws SQLException {
    return this.doGetConnection(this.username, this.password);
}

public Connection getConnection(String username, String password) throws SQLException {
    return this.doGetConnection(username, password);
}
```

实际上，这两个方法都指向了内部的一个`doGetConnection`方法，那么我们接着来看：

```java
private Connection doGetConnection(String username, String password) throws SQLException {
    Properties props = new Properties();
    if (this.driverProperties != null) {
        props.putAll(this.driverProperties);
    }

    if (username != null) {
        props.setProperty("user", username);
    }

    if (password != null) {
        props.setProperty("password", password);
    }

    return this.doGetConnection(props);
}
```

这里将用户名和密码配置封装为一个Properties对象，然后执行另一个重载同名的方法：

```java
private Connection doGetConnection(Properties properties) throws SQLException {
  	//若未初始化驱动，需要先初始化，内部维护了一个Map来记录初始化信息，这里不多介绍了
    this.initializeDriver();
  	//传统的获取连接的方式，是不是终于找到熟悉的味道了
    Connection connection = DriverManager.getConnection(this.url, properties);
  	//对连接进行额外的一些配置
    this.configureConnection(connection);
    return connection;   //返回得到的Connection对象
}
```

到这里，就返回`Connection`对象了，而此对象正是通过`DriverManager`来创建的，因此，非池化的数据源实现依然使用的是传统的连接创建方式，那我们接着来看池化的数据源实现，它是`PooledDataSource`类：

```java
public class PooledDataSource implements DataSource {
    private static final Log log = LogFactory.getLog(PooledDataSource.class);
    private final PoolState state = new PoolState(this);
  	//内部维护了一个非池化的数据源，这是要干嘛？
    private final UnpooledDataSource dataSource;
    protected int poolMaximumActiveConnections = 10;
    protected int poolMaximumIdleConnections = 5;
    protected int poolMaximumCheckoutTime = 20000;
    protected int poolTimeToWait = 20000;
    protected int poolMaximumLocalBadConnectionTolerance = 3;
    protected String poolPingQuery = "NO PING QUERY SET";
    protected boolean poolPingEnabled;
    protected int poolPingConnectionsNotUsedFor;
    private int expectedConnectionTypeCode;
  	//并发相关类，我们在JUC篇视频教程中介绍过，感兴趣可以前往观看
    private final Lock lock = new ReentrantLock();
    private final Condition condition;
```

我们发现，在这里的定义就比非池化的实现复杂得多了，因为它还要考虑并发的问题，并且还要考虑如何合理地存放大量的链接对象，该如何进行合理分配，因此它的玩法非常之高级，但是，再高级的玩法，我们都要拿下。

首先注意，它存放了一个UnpooledDataSource，此对象是在构造时就被创建，其实创建Connection还是依靠数据库驱动创建，我们后面慢慢解析，首先我们来看看它是如何实现接口方法的：

```java
public Connection getConnection() throws SQLException {
    return this.popConnection(this.dataSource.getUsername(), this.dataSource.getPassword()).getProxyConnection();
}

public Connection getConnection(String username, String password) throws SQLException {
    return this.popConnection(username, password).getProxyConnection();
}
```

可以看到，它调用了`popConnection()`方法来获取连接对象，然后进行了一个代理，通过这方法名字我们可以猜测，有可能整个连接池就是一个类似于栈的集合类型结构实现的。那么我们接着来看看`popConnection`方法：

```java
private PooledConnection popConnection(String username, String password) throws SQLException {
    boolean countedWait = false;
  	//返回的是PooledConnection对象，
    PooledConnection conn = null;
    long t = System.currentTimeMillis();
    int localBadConnectionCount = 0;

    while(conn == null) {
        synchronized(this.state) {   //加锁，因为有可能很多个线程都需要获取连接对象
            PoolState var10000;
          	//PoolState存了两个List，一个是空闲列表，一个是活跃列表
            if (!this.state.idleConnections.isEmpty()) {   //有空闲的连接时，可以直接分配Connection
                conn = (PooledConnection)this.state.idleConnections.remove(0);  //ArrayList中取第一个元素
                if (log.isDebugEnabled()) {
                    log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");
                }
            //如果已经没有多余的连接可以分配，那么就检查一下活跃连接数是否达到最大的分配上限，如果没有，就new一个新的
            } else if (this.state.activeConnections.size() < this.poolMaximumActiveConnections) {
              	//注意new了之后并没有立即往List里面塞，只是存了一些基本信息
              	//我们发现，这里依靠UnpooledDataSource创建了一个Connection对象，并将其封装到PooledConnection中
              	//所以说内部维护的UnpooledDataSource对象其实是为了节省代码，因为创建数据库连接其实都是一样的方式
                conn = new PooledConnection(this.dataSource.getConnection(), this);
                if (log.isDebugEnabled()) {
                    log.debug("Created connection " + conn.getRealHashCode() + ".");
                }
              //以上条件都不满足，那么只能从之前的连接中寻找了，看看有没有那种卡住的链接（比如，由于网络问题有可能之前的连接一直被卡住，然而正常情况下早就结束并且可以使用了，所以这里相当于是优化也算是一种捡漏的方式）
            } else {
              	//获取最早创建的连接
                PooledConnection oldestActiveConnection = (PooledConnection)this.state.activeConnections.get(0);
                long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();
              	//判断是否超过最大的使用时间
                if (longestCheckoutTime > (long)this.poolMaximumCheckoutTime) {
                  	//超时统计信息（不重要）
                    ++this.state.claimedOverdueConnectionCount;
                    var10000 = this.state;
                    var10000.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
                    var10000 = this.state;
                    var10000.accumulatedCheckoutTime += longestCheckoutTime;
                  	//从活跃列表中移除此链接信息
                    this.state.activeConnections.remove(oldestActiveConnection);
                  	//如果开启事务，还需要回滚一下
                    if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
                        try {
                            oldestActiveConnection.getRealConnection().rollback();
                        } catch (SQLException var15) {
                            log.debug("Bad connection. Could not roll back");
                        }
                    }
										
                  	//这里就根据之前的连接对象直接new一个新的连接（注意使用的还是之前的Connection对象，并没有创建新的对象，只是被重新封装了）
                    conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
                    conn.setCreatedTimestamp(oldestActiveConnection.getCreatedTimestamp());
                    conn.setLastUsedTimestamp(oldestActiveConnection.getLastUsedTimestamp());
                  	//过期
                    oldestActiveConnection.invalidate();
                    if (log.isDebugEnabled()) {
                        log.debug("Claimed overdue connection " + conn.getRealHashCode() + ".");
                    }
                } else {
                  //没有超时，那就确实是没连接可以用了，只能卡住了（阻塞）
                  //然后顺手记录一下目前有几个线程在等待其他的任务搞完
                    try {
                        if (!countedWait) {
                            ++this.state.hadToWaitCount;
                            countedWait = true;
                        }

                        if (log.isDebugEnabled()) {
                            log.debug("Waiting as long as " + this.poolTimeToWait + " milliseconds for connection.");
                        }
                      	//最后再等等
                        long wt = System.currentTimeMillis();
                        this.state.wait((long)this.poolTimeToWait);
                      	//要是超过等待时间还是没等到，只能放弃了
                      	//注意这样的话con就为null了
                        var10000 = this.state;
                        var10000.accumulatedWaitTime += System.currentTimeMillis() - wt;
                    } catch (InterruptedException var16) {
                        break;
                    }
                }
            }
						
          	//经过之前的操作，并且已经成功分配到连接对象的情况下
            if (conn != null) {
                if (conn.isValid()) {  //首先验证是否有效
                    if (!conn.getRealConnection().getAutoCommit()) {  //清理之前可能存在的遗留事务操作
                        conn.getRealConnection().rollback();
                    }

                    conn.setConnectionTypeCode(this.assembleConnectionTypeCode(this.dataSource.getUrl(), username, password));
                    conn.setCheckoutTimestamp(System.currentTimeMillis());
                    conn.setLastUsedTimestamp(System.currentTimeMillis());
                  	//添加到活跃表中
                    this.state.activeConnections.add(conn);
                    //统计信息（不重要）
                    ++this.state.requestCount;
                    var10000 = this.state;
                    var10000.accumulatedRequestTime += System.currentTimeMillis() - t;
                } else {
                  	//无效的连接，直接抛异常
                    if (log.isDebugEnabled()) {
                        log.debug("A bad connection (" + conn.getRealHashCode() + ") was returned from the pool, getting another connection.");
                    }

                    ++this.state.badConnectionCount;
                    ++localBadConnectionCount;
                    conn = null;
                    if (localBadConnectionCount > this.poolMaximumIdleConnections + this.poolMaximumLocalBadConnectionTolerance) {
                        if (log.isDebugEnabled()) {
                            log.debug("PooledDataSource: Could not get a good connection to the database.");
                        }

                        throw new SQLException("PooledDataSource: Could not get a good connection to the database.");
                    }
                }
            }
        }
    }
	
  	//最后该干嘛干嘛，要是之前拿到的con是null的话，直接抛异常
    if (conn == null) {
        if (log.isDebugEnabled()) {
            log.debug("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
        }

        throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
    } else {
        return conn;   //否则正常返回
    }
}
```

经过上面一顿猛如虎的操作之后，我们可以得到以下信息：

> 如果最后得到了连接对象（有可能是从空闲列表中得到，有可能是直接创建的新的，还有可能是经过回收策略回收得到的），那么连接(Connection)对象一定会被放在活跃列表中(state.activeConnections)

那么肯定有一个疑问，现在我们已经知道获取一个链接会直接进入到活跃列表中，那么，如果一个连接被关闭，又会发生什么事情呢，我们来看看此方法返回之后，会调用`getProxyConnection`来获取一个代理对象，实际上就是`PooledConnection`类：

```java
class PooledConnection implements InvocationHandler {
  private static final String CLOSE = "close";
    private static final Class<?>[] IFACES = new Class[]{Connection.class};
    private final int hashCode;
  	//会记录是来自哪一个数据源创建的的
    private final PooledDataSource dataSource;
  	//连接对象本体
    private final Connection realConnection;
  	//代理的链接对象
    private final Connection proxyConnection;
  	...
```

它直接代理了构造方法中传入的Connection对象，也是使用JDK的动态代理实现的，那么我们来看一下，它是如何进行代理的：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    String methodName = method.getName();
  	//如果调用的是Connection对象的close方法，
    if ("close".equals(methodName)) {
      	//这里并不会真的关闭连接（这也是为什么用代理），而是调用之前数据源的pushConnection方法，将此连接改为为空闲状态
        this.dataSource.pushConnection(this);
        return null;
    } else {
        try {
            if (!Object.class.equals(method.getDeclaringClass())) {
                this.checkConnection();
              	//任何操作执行之前都会检查连接是否可用
            }

          	//原方法该干嘛干嘛
            return method.invoke(this.realConnection, args);
        } catch (Throwable var6) {
            throw ExceptionUtil.unwrapThrowable(var6);
        }
    }
}
```

这下，池化数据源的大致流程其实就已经很清晰了，那么我们最后再来看看`pushConnection`方法：

```java
protected void pushConnection(PooledConnection conn) throws SQLException {
    synchronized(this.state) {   //老规矩，先来把锁
      	//先从活跃列表移除此连接
        this.state.activeConnections.remove(conn);
      	//判断此链接是否可用
        if (conn.isValid()) {
            PoolState var10000;
          	//看看闲置列表容量是否已满（容量满了就回不去了）
            if (this.state.idleConnections.size() < this.poolMaximumIdleConnections && conn.getConnectionTypeCode() == this.expectedConnectionTypeCode) {
                var10000 = this.state;
                var10000.accumulatedCheckoutTime += conn.getCheckoutTime();
                if (!conn.getRealConnection().getAutoCommit()) {
                    conn.getRealConnection().rollback();
                }

              	//把唯一有用的Connection对象拿出来，然后重新创建一个PooledConnection包装
                PooledConnection newConn = new PooledConnection(conn.getRealConnection(), this);
              	//放入闲置列表，成功回收
                this.state.idleConnections.add(newConn);
                newConn.setCreatedTimestamp(conn.getCreatedTimestamp());
                newConn.setLastUsedTimestamp(conn.getLastUsedTimestamp());
                conn.invalidate();
                if (log.isDebugEnabled()) {
                    log.debug("Returned connection " + newConn.getRealHashCode() + " to pool.");
                }

                this.state.notifyAll();
            } else {
                var10000 = this.state;
                var10000.accumulatedCheckoutTime += conn.getCheckoutTime();
                if (!conn.getRealConnection().getAutoCommit()) {
                    conn.getRealConnection().rollback();
                }

                conn.getRealConnection().close();
                if (log.isDebugEnabled()) {
                    log.debug("Closed connection " + conn.getRealHashCode() + ".");
                }

                conn.invalidate();
            }
        } else {
            if (log.isDebugEnabled()) {
                log.debug("A bad connection (" + conn.getRealHashCode() + ") attempted to return to the pool, discarding connection.");
            }

            ++this.state.badConnectionCount;
        }

    }
}
```

这样，我们就已经完全了解了Mybatis的池化数据源的执行流程了。只不过，无论Connection管理方式如何变换，无论数据源再高级，我们要知道，它都最终都会使用`DriverManager`来创建连接对象，而最终使用的也是`DriverManager`提供的`Connection`对象。

### 整合Mybatis框架

通过了解数据源，我们已经清楚，Mybatis实际上是在使用自己编写的数据源（数据源实现其实有很多，之后我们再聊其他的）默认使用的是池化数据源，它预先存储了很多的连接对象。

那么我们来看一下，如何将Mybatis与Spring更好的结合呢，比如我们现在希望将SqlSessionFactory交给IoC容器进行管理，而不是我们自己创建工具类来管理（我们之前一直都在使用工具类管理和创建会话）

```xml
<!-- 这两个依赖不用我说了吧 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
  	<!-- 注意，对于Spring 6.0来说，版本需要在3.5以上 -->
    <version>3.5.13</version>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.31</version>
</dependency>
<!-- Mybatis针对于Spring专门编写的支持框架 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>3.0.2</version>
</dependency>
<!-- Spring的JDBC支持框架 -->
<dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-jdbc</artifactId>
     <version>6.0.10</version>
</dependency>
```

在mybatis-spring依赖中，为我们提供了SqlSessionTemplate类，它其实就是官方封装的一个工具类，我们可以将其注册为Bean，这样我们随时都可以向IoC容器索要对象，而不用自己再去编写一个工具类了，我们可以直接在配置类中创建。对于这种别人编写的类型，如果要注册为Bean，那么只能在配置类中完成：

```java
@Configuration
@ComponentScan("org.example.entity")
public class MainConfiguration {
  	//注册SqlSessionTemplate的Bean
    @Bean
    public SqlSessionTemplate sqlSessionTemplate() throws IOException {
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis-config.xml"));
        return new SqlSessionTemplate(factory);
    }
}
```

这里随便编写一个测试的Mapper类：

```java
@Data
public class Student {
    private int sid;
    private String name;
    private String sex;
}
```

```java
public interface TestMapper {
    @Select("select * from student where sid = 1")
    Student getStudent();
}
```

最后是配置文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/study"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
  	<mappers>
        <mapper class="org.example.mapper.TestMapper"/>
    </mappers>
</configuration>
```

我们来测试一下吧：

```java
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
    SqlSessionTemplate template = context.getBean(SqlSessionTemplate.class);
    TestMapper testMapper = template.getMapper(TestMapper.class);
    System.out.println(testMapper.getStudent());
}
```

![image-20221217142651610](https://s2.loli.net/2022/12/17/L83vrESxoXKO7fQ.png)

这样，我们就成功将Mybatis与Spring完成了初步整合，直接从容器中就能获取到SqlSessionTemplate，结合自动注入，我们的代码量能够进一步的减少。

虽然这样已经很方便了，但是还不够方便，我们依然需要手动去获取Mapper对象，那么能否直接得到对应的Mapper对象呢，我们希望让Spring直接帮助我们管理所有的Mapper，当需要时，可以直接从容器中获取，我们可以直接在配置类上方添加注解：

```java
@Configuration
@ComponentScan("org.example.entity")
@MapperScan("org.example.mapper")
public class MainConfiguration {
```

这样，Mybatis就会自动扫描对应包下所有的接口，并直接被注册为对应的Mapper作为Bean管理，那么我们现在就可以直接通过容器获取了：

```java
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
    TestMapper mapper = context.getBean(TestMapper.class);
    System.out.println(mapper.getStudent());
}
```

在我们后续的SpringBoot学习阶段，会有更加方便的方式来注册Mapper，我们只需要一个`@Mapper`注解即可完成，非常简单。

请一定注意，必须存在`SqlSessionTemplate`或是`SqlSessionFactoryBean`的Bean，否则会无法初始化（毕竟要数据库的链接信息）我们接着来看，如果我们希望直接去除Mybatis的配置文件，完全实现全注解配置，那么改怎么去实现呢？我们可以使用`SqlSessionFactoryBean`类：

```java
@Configuration
@ComponentScan("org.example.entity")
@MapperScan("org.example.mapper")
public class MainConfiguration {
    @Bean   //单独创建一个Bean，方便之后更换
    public DataSource dataSource(){
        return new PooledDataSource("com.mysql.cj.jdbc.Driver",
                "jdbc:mysql://localhost:3306/study", "root", "123456");
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){  //直接参数得到Bean对象
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        return bean;
    }
}
```

首先我们需要创建一个数据源的实现类，因为这是数据库最基本的信息，然后再给到`SqlSessionFactoryBean`实例，这样，我们相当于直接在一开始通过IoC容器配置了`SqlSessionFactory`，这里只需要传入一个`DataSource`的实现即可，我们采用池化数据源。

删除配置文件，重新再来运行，同样可以正常使用Mapper。从这里开始，通过IoC容器，Mybatis已经不再需要使用配置文件了，在我们之后的学习中，基于Spring的开发将不会再出现Mybatis的配置文件。

### 使用HikariCP连接池

前面我们提到了数据源还有其他实现，比如C3P0、Druid等，它们都是非常优秀的数据源实现（可以自行了解），不过我们这里要介绍的，是之后在SpringBoot中还会遇到的HikariCP连接池。

> HikariCP是由日本程序员开源的一个数据库连接池组件，代码非常轻量，并且速度非常的快。根据官方提供的数据，在酷睿i7开启32个线程32个连接的情况下，进行随机数据库读写操作，HikariCP的速度是现在常用的C3P0数据库连接池的数百倍。在SpringBoot 3.0中，官方也是推荐使用HikariCP。

![image-20221217145126777](https://s2.loli.net/2022/12/17/Q6gPI9RVe1X7Noq.png)

首先，我们需要导入依赖：

```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.0.1</version>
</dependency>
```

要更换数据源实现，非常简单，我们可以直接声明一个Bean：

```java
@Bean
public DataSource dataSource() {
    HikariDataSource dataSource = new HikariDataSource();
    dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/study");
    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUsername("root");
    dataSource.setPassword("123456");
    return dataSource;
}
```

最后我们发现，同样可以得到输出结果，但是出现了一个报错：

```
SLF4J: No SLF4J providers were found.
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#noProviders for further details.
```

此数据源实际上是采用了SLF4J日志框架打印日志信息，但是现在没有任何的日志实现（slf4j只是一个API标准，它规范了多种日志框架的操作，统一使用SLF4J定义的方法来操作不同的日志框架，我们会在SpringBoot篇进行详细介绍）我们这里就使用JUL作为日志实现，我们需要导入另一个依赖：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jdk14</artifactId>
    <version>1.7.25</version>
</dependency>
```

注意版本一定要和`slf4j-api`保持一致，我们可以在这里直接查看：

![image-20221217154216832](https://s2.loli.net/2022/12/17/93OSknRKXwdZsp7.png)

这样，HikariCP数据源的启动日志就可以正常打印出来了：

```
12月 17, 2022 3:41:38 下午 com.zaxxer.hikari.HikariDataSource getConnection
信息: HikariPool-1 - Starting...
12月 17, 2022 3:41:38 下午 com.zaxxer.hikari.pool.HikariPool checkFailFast
信息: HikariPool-1 - Added connection com.mysql.cj.jdbc.ConnectionImpl@4f8969b0
12月 17, 2022 3:41:38 下午 com.zaxxer.hikari.HikariDataSource getConnection
信息: HikariPool-1 - Start completed.
Student(sid=1, name=小明, sex=男)
```

在SpringBoot阶段，我们还会遇到`HikariPool-1 - Starting...`和`HikariPool-1 - Start completed.`同款日志信息。

当然，Lombok肯定也是支持这个日志框架快速注解的：

```java
@Slf4j
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
        TestMapper mapper = context.getBean(TestMapper.class);
        log.info(mapper.getStudent().toString());
    }
}
```

是不是感觉特别方便？

### Mybatis事务管理

我们前面已经讲解了如何让Mybatis与Spring更好地融合在一起，通过将对应的Bean类型注册到容器中，就能更加方便的去使用Mapper，那么现在，我们接着来看Spring的事务控制。

在开始之前，我们还是回顾一下事务机制。首先事务遵循一个ACID原则：

- 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
- 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。类比银行转账，从一个账号扣款，另一个账号增款，必须保证总金额不变。
- 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。类比多个人同时编辑同一文档，每个人看到的结果都是独立的，不会受其他人的影响，不过难免会存在冲突。
- 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。类比写入硬盘的文件，即使关机重启，文件仍然存在。

简单来说，事务就是要么完成，要么就啥都别做！并且不同的事务之间相互隔离，互不干扰。

那么我们接着来深入了解一下事务的**隔离机制**（在之前数据库入门阶段并没有提到）我们说了，事务之间是相互隔离互不干扰的，那么如果出现了下面的情况，会怎么样呢：

> 当两个事务同时在执行，并且同时在操作同一个数据，这样很容易出现并发相关的问题，比如一个事务先读取了某条数据，而另一个事务此时修改了此数据，当前一个事务紧接着再次读取时，会导致和前一次读取的数据不一致，这就是一种典型的数据虚读现象。

因此，为了解决这些问题，事务之间实际上是存在一些隔离级别的：

- ISOLATION_READ_UNCOMMITTED（读未提交）：其他事务会读取当前事务尚未更改的提交（相当于读取的是这个事务暂时缓存的内容，并不是数据库中的内容）
- ISOLATION_READ_COMMITTED（读已提交）：其他事务会读取当前事务已经提交的数据（也就是直接读取数据库中已经发生更改的内容）
- ISOLATION_REPEATABLE_READ（可重复读）：其他事务会读取当前事务已经提交的数据并且其他事务执行过程中不允许再进行数据修改（注意这里仅仅是不允许修改数据）
- ISOLATION_SERIALIZABLE（串行化）：它完全服从ACID原则，一个事务必须等待其他事务结束之后才能开始执行，相当于挨个执行，效率很低

我们依次来看看，不同的隔离级别会导致什么问题。首先是`读未提交`级别，此级别属于最低级别，相当于各个事务共享一个缓存区域，任何事务的操作都在这里进行。那么它会导致以下问题：

![image-20221217155511058](https://s2.loli.net/2022/12/17/hQpluLA2bFKo1O8.png)

也就是说，事务A最后得到的实际上是一个毫无意义的数据（事务B已经回滚了）我们称此数据为"脏数据"，这种现象称为**脏读**

我们接着来看`读已提交`级别，事务只能读取其他事务已经提交的内容，相当于直接从数据中读取数据，这样就可以避免**脏读**问题了，但是它还是存在以下问题：

![image-20221217155538073](https://s2.loli.net/2022/12/17/K1sJbDNyudOgAcV.png)

这正是我们前面例子中提到的问题，虽然它避免了脏读问题，但是如果事件B修改并提交了数据，那么实际上事务A之前读取到的数据依然不是最新的数据，直接导致两次读取的数据不一致，这种现象称为**虚读**也可以称为**不可重复读**

因此，下一个隔离级别`可重复读`就能够解决这样的问题（MySQL的默认隔离级别），它规定在其他事务执行时，不允许修改数据，这样，就可以有效地避免不可重复读的问题，但是这样就一定安全了吗？这里仅仅是禁止了事务执行过程中的UPDATE操作，但是它并没有禁止INSERT这类操作，因此，如果事务A执行过程中事务B插入了新的数据，那么A这时是毫不知情的，比如：

![image-20221217160023674](https://s2.loli.net/2022/12/17/uwiHT8AcobeBjL3.png)

两个人同时报名一个活动，两个报名的事务同时在进行，但是他们一开始读取到的人数都是5，而这时，它们都会认为报名成功后人数应该变成6，而正常情况下应该是7，因此这个时候就发生了数据的**幻读**现象。

因此，要解决这种问题，只能使用最后一种隔离级别`串行化`来实现了，每个事务不能同时进行，直接避免所有并发问题，简单粗暴，但是效率爆减，并不推荐。

最后总结三种情况：

- 脏读：读取到了被回滚的数据，它毫无意义。
- 虚读（不可重复读）：由于其他事务更新数据，两次读取的数据不一致。
- 幻读：由于其他事务执行插入删除操作，而又无法感知到表中记录条数发生变化，当下次再读取时会莫名其妙多出或缺失数据，就像产生幻觉一样。

（对于虚读和幻读的区分：虚读是某个数据前后读取不一致，幻读是整个表的记录数量前后读取不一致

最后这张图，请务必记在你的脑海，记在你的心中：

![image-20221217160052616](https://s2.loli.net/2022/12/17/nHfV8R1ZUybTSd2.png)

Mybatis对于数据库的事务管理，也有着相应的封装。一个事务无非就是创建、提交、回滚、关闭，因此这些操作被Mybatis抽象为一个接口：

```java
public interface Transaction {
    Connection getConnection() throws SQLException;

    void commit() throws SQLException;

    void rollback() throws SQLException;

    void close() throws SQLException;

    Integer getTimeout() throws SQLException;
}
```

对于此接口的实现，MyBatis的事务管理分为两种形式：

1. 使用**JDBC**的事务管理机制：即利用对应数据库的驱动生成的`Connection`对象完成对事务的提交（commit()）、回滚（rollback()）、关闭（close()）等，对应的实现类为`JdbcTransaction`
2. 使用**MANAGED**的事务管理机制：这种机制MyBatis自身不会去实现事务管理，而是让程序的容器（比如Spring）来实现对事务的管理，对应的实现类为`ManagedTransaction`
3. 如果需要自定义，那么得实现`org.apache.ibatis.transaction.Transaction`接口，然后在`type`属性中指定其类名。使用自定义的事务管理器可以根据具体需求来实现一些特定的事务管理行为。

而我们之前一直使用的其实就是JDBC的事务，相当于直接使用`Connection`对象（之前JavaWeb阶段已经讲解过了）在进行事务操作，并没有额外的管理机制，对应的配置为：

```xml
<transactionManager type="JDBC"/>
```

那么我们来看看`JdbcTransaction`是不是像我们上面所说的那样管理事务的，直接上源码：

```java
public class JdbcTransaction implements Transaction {
    private static final Log log = LogFactory.getLog(JdbcTransaction.class);
    protected Connection connection;
    protected DataSource dataSource;
    protected TransactionIsolationLevel level;
    protected boolean autoCommit;

    public JdbcTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
      	//数据源
        this.dataSource = ds;
      	//事务隔离级别，上面已经提到过了
        this.level = desiredLevel;
      	//是否自动提交
        this.autoCommit = desiredAutoCommit;
    }
  	
  //也可以直接给个Connection对象
   public JdbcTransaction(Connection connection) {
        this.connection = connection;
    }

    public Connection getConnection() throws SQLException {
      	//没有就通过数据源新开一个Connection
        if (this.connection == null) {
            this.openConnection();
        }
	
        return this.connection;
    }

    public void commit() throws SQLException {
      	//连接已经创建并且没开启自动提交才可以使用
        if (this.connection != null && !this.connection.getAutoCommit()) {
            if (log.isDebugEnabled()) {
                log.debug("Committing JDBC Connection [" + this.connection + "]");
            }
						//实际上使用的是数据库驱动提供的Connection对象进行事务操作
            this.connection.commit();
        }
    }
  	...
```

相当于`JdbcTransaction`只是为数据库驱动提供的`Connection`对象套了层壳，所有的事务操作实际上是直接调用`Connection`对象。那么我们接着来看`ManagedTransaction`的源码：

```java
public class ManagedTransaction implements Transaction {
    ...

    public void commit() throws SQLException {
    }

    public void rollback() throws SQLException {
    }

    ...
}
```

我们发现，大体内容和`JdbcTransaction`差不多，但是它并没有实现任何的事务操作。也就是说，它希望将实现交给其他的管理框架来完成，而Spring就为Mybatis提供了一个非常好的事务管理实现。

### 使用Spring事务管理

现在我们来学习一下Spring提供的事务管理（Spring事务管理分为编程式事务和声明式事务，但是编程式事务过于复杂并且具有高度耦合性，违背了Spring框架的设计初衷，因此这里只讲解声明式事务）声明式事务是基于AOP实现的。

使用声明式事务非常简单，我们只需要在配置类添加`@EnableTransactionManagement`注解即可，这样就可以开启Spring的事务支持了。接着，我们只需要把一个事务要做的所有事情封装到Service层的一个方法中即可，首先需要在配置文件中注册一个新的Bean，事务需要执行必须有一个事务管理器：

```java
@Configuration
@ComponentScan("org.example")
@MapperScan("org.example.mapper")
@EnableTransactionManagement
public class MainConfiguration {

    @Bean
    public TransactionManager transactionManager(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
  	...
```

接着我们来编写一个简单的Mapper操作：

```java
@Mapper
public interface TestMapper {
    ...

    @Insert("insert into student(name, sex) values('测试', '男')")
    void insertStudent();
}
```

这样会向数据库中插入一条新的学生信息，接着，假设我们这里有一个业务需要连续插入两条学生信息，首先编写业务层的接口：

```java
public interface TestService {
    void test();
}
```

接着，我们再来编写业务层的实现，我们可以直接将其注册为Bean，交给Spring来进行管理，这样就可以自动将Mapper注入到类中了，并且可以支持事务：

```java
@Component
public class TestServiceImpl implements TestService{

    @Resource
    TestMapper mapper;

    @Transactional   //此注解表示事务，之后执行的所有方法都会在同一个事务中执行
    public void test() {
        mapper.insertStudent();
        if(true) throw new RuntimeException("我是测试异常！");
        mapper.insertStudent();
    }
}
```

我们只需在方法上添加`@Transactional`注解，即可表示此方法执行的是一个事务操作，在调用此方法时，Spring会通过AOP机制为其进行增强，一旦发现异常，事务会自动回滚。最后我们来调用一下此方法：

```java
@Slf4j
public class Main {
    public static void main(String[] args) {
        log.info("项目正在启动...");
        ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);
        TestService service = context.getBean(TestService.class);
        service.test();
    }
}
```

得到的结果是出现错误：

```
12月 17, 2022 4:09:00 下午 com.zaxxer.hikari.HikariDataSource getConnection
信息: HikariPool-1 - Start completed.
Exception in thread "main" java.lang.RuntimeException: 我是测试异常！
	at org.example.service.TestServiceImpl.test(TestServiceImpl.java:17)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:343)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:196)
```

我们发现，整个栈追踪信息中包含了大量aop包下的内容，也就印证了它确实是通过AOP实现的，那么我们接着来看一下，数据库中的数据是否没有发生变化（出现异常回滚了）

![image-20221217161027254](https://s2.loli.net/2022/12/17/TQDbpK2JVP3d9wz.png)

结果显而易见，第一次的插入操作确实被回滚了，数据库中没有任何新增的内容。

我们接着来研究一下`@Transactional`注解的一些参数：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    String[] label() default {};

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;

    int timeout() default -1;

    String timeoutString() default "";

    boolean readOnly() default false;

    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}
```

我们来讲解几个比较关键的属性：

- transactionManager：指定事务管理器
- propagation：事务传播规则，一个事务可以包括N个子事务
- isolation：事务隔离级别，不多说了
- timeout：事务超时时间
- readOnly：是否为只读事务，不同的数据库会根据只读属性进行优化，比如MySQL一旦声明事务为只读，那么久不允许增删改操作了。
- rollbackFor和noRollbackFor：发生指定异常时回滚或是不回滚，默认发生任何异常都回滚。

除了事务的传播规则，其他的内容其实已经给大家讲解过了，那么我们就来看看事务的传播。事务传播一共有七种级别：

![image-20221217161156859](https://s2.loli.net/2022/12/17/C1RA4mBEoxNDFGl.png)

Spring默认的传播级别是`PROPAGATION_REQUIRED`，那么我们来看看，它是如何传播的，现在我们的`Service`类中一共存在两个事务，而一个事务方法包含了另一个事务方法：

```java
@Component
public class TestServiceImpl implements TestService{

    @Resource
    TestMapper mapper;

    @Transactional
    public void test() {
        test2();   //包含另一个事务
        if(true) throw new RuntimeException("我是测试异常！");  //发生异常时，会回滚另一个事务吗？
    }

    @Transactional
    public void test2() {
        mapper.insertStudent();
    }
}
```

最后我们得到结果，另一个事务也被回滚了，也就是说，相当于另一个事务直接加入到此事务中，也就是表中所描述的那样。如果单独执行`test2()`则会开启一个新的事务，而执行`test()`则会直接让内部的`test2()`加入到当前事务中。

现在我们将`test2()`的传播级别设定为`SUPPORTS`，那么这时如果单独调用`test2()`方法，并不会以事务的方式执行，当发生异常时，虽然依然存在AOP增强，但是不会进行回滚操作，而现在再调用`test()`方法，才会以事务的方式执行：

```java
@Transactional
public void test() {
    test2();
}

@Transactional(propagation = Propagation.SUPPORTS)
public void test2() {
    mapper.insertStudent();
    if(true) throw new RuntimeException("我是测试异常！");
}
```

我们接着来看`MANDATORY`，它非常严格，如果当前方法并没有在任何事务中进行，会直接出现异常：

```java
@Transactional
public void test() {
    test2();
}

@Transactional(propagation = Propagation.MANDATORY)
public void test2() {
    mapper.insertStudent();
    if(true) throw new RuntimeException("我是测试异常！");
}
```

直接运行`test2()`方法，报错如下：

```
Exception in thread "main" org.springframework.transaction.IllegalTransactionStateException: No existing transaction found for transaction marked with propagation 'mandatory'
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.getTransaction(AbstractPlatformTransactionManager.java:362)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.createTransactionIfNecessary(TransactionAspectSupport.java:595)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:382)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:215)
	at com.sun.proxy.$Proxy29.test2(Unknown Source)
	at com.test.Main.main(Main.java:17)
```

`NESTED`级别表示如果存在外层事务，则此方法单独创建一个子事务，回滚只会影响到此子事务，实际上就是利用创建Savepoint，然后回滚到此保存点实现的。`NEVER`级别表示此方法不应该加入到任何事务中，其余类型适用于同时操作多数据源情况下的分布式事务管理，这里暂时不做介绍。

### 集成JUnit测试

既然使用了Spring，那么怎么集成到JUnit中进行测试呢，首先大家能够想到的肯定是：

```java
public class TestMain {

    @Test
    public void test(){
        ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);
        TestService service = context.getBean(TestService.class);
        service.test();
    }
}
```

直接编写一个测试用例即可，但是这样的话，如果我们有很多个测试用例，那么我们不可能每次测试都去创建ApplicationContext吧？我们可以使用`@Before`添加一个测试前动作来提前配置ApplicationContext，但是这样的话，还是不够简便，能不能有更快速高效的方法呢？

Spring为我们提供了一个Test模块，它会自动集成Junit进行测试，我们可以导入一下依赖：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>6.0.10</version>
</dependency>
```

这里导入的是JUnit5和SpringTest模块依赖，然后直接在我们的测试类上添加两个注解就可以搞定：

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = TestConfiguration.class)
public class TestMain {

    @Autowired
    TestService service;
    
    @Test
    public void test(){
        service.test();
    }
}
```

`@ExtendWith`是由JUnit提供的注解，等同于旧版本的`@RunWith`注解，然后使用SpringTest模块提供的`@ContextConfiguration`注解来表示要加载哪一个配置文件，可以是XML文件也可以是类，我们这里就直接使用类进行加载。

配置完成后，我们可以直接使用`@Autowired`来进行依赖注入，并且直接在测试方法中使用注入的Bean，现在就非常方便了。

至此，SSM中的其中一个S（Spring）和一个M（Mybatis）就已经学完了，我们还剩下一个SpringMvc需要去学习，下一章，我们将重新回到Web开发，了解在Spring框架的加持下，我们如何更高效地开发Web应用程序。

***

## 实现原理探究（选学）

**注意：**本版块难度很大，所有内容都作为选学内容。

如果学习Spring基本内容对你来说已经非常困难了，建议跳过此小节，直接进入MVC阶段的学习，此小节会从源码角度解释Spring的整个运行原理，对初学者来说等同于小学跨越到高中，它并不是必学内容，但是对于个人阅历提升极为重要（推荐完成整个SSM阶段的学习并且加以实战之后再来看此部分），如果你还是觉得自己能够跟上节奏继续深入钻研底层原理，那么现在就开始吧。

### Bean工厂与Bean定义

实际上我们之前的所有操作都离不开一个东西，那就是IoC容器，那么它到底是如何实现呢？这一部分我们将详细介绍，首先我们大致了解一下ApplicationContext的加载流程：

![image-20221217162656678](https://s2.loli.net/2022/12/17/Un6qjPci2uvkL5X.png)

我们可以看到，整个过程极为复杂，一句话肯定是无法解释的。由于Spring的源码非常庞大，因此我们不可能再像了解其他框架那样直接自底向上逐行干源码了（各位可以自己点开看看，代码量非常多）

![image-20221217162821241](https://s2.loli.net/2022/12/17/QXqvO1sGh6d4ZSz.png)

我们只能对几个关键部分进行介绍，在了解这些内容之后，实际上不需要完全阅读所有部分的源码都可以有一个大致的认识。

首先，容器既然要管理Bean，那么肯定需要一个完善的管理机制，实际上，对Bean的管理都是依靠BeanFactory在进行，顾名思义BeanFactory就是对Bean进行生产和管理的工厂，我们可以尝试自己创建和使用BeanFactory对象：

```java
public static void main(String[] args) {
    BeanFactory factory = new DefaultListableBeanFactory();  //这是BeanFactory的一个默认实现类
    System.out.println("获取Bean对象："+factory.getBean("lbwnb"));  //我们可以直接找工厂获取Bean对象
}
```

我们可以直接找Bean工厂索要对象，只不过在一开始，工厂并不知道自己需要生产什么，可以生产什么，因此我们直接索要一个工厂不知道的Bean对象，会直接得到：

![image-20230214193208233](https://s2.loli.net/2023/02/14/n54N3iFQX7awHAl.png)

我们只有告诉工厂我们要生产什么，怎么生产，工厂才能开工：

```java
public static void main(String[] args) {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();  //这是BeanFactory的一个默认实现类

    BeanDefinition definition = BeanDefinitionBuilder   //使用BeanDefinitionBuilder快速创建Bean定义
            .rootBeanDefinition(Student.class)   //Bean的类型
            .setScope("prototype")    //设置作用域为原型模式
            .getBeanDefinition();     //生成此Bean定义
    factory.registerBeanDefinition("lbwnb", definition);   //向工厂注册Bean此定义，并设定Bean的名称

    System.out.println(factory.getBean("lbwnb"));  //现在就可以拿到了
}
```

实际上，我们的ApplicationContext中就维护了一个AutowireCapableBeanFactory对象：

```java
public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {
 	@Nullable
	private volatile DefaultListableBeanFactory beanFactory;   //默认构造后存放在这里的是一个DefaultListableBeanFactory对象
  
  ...
  
  @Override
  public final ConfigurableListableBeanFactory getBeanFactory() {   //getBeanFactory就可以直接得到上面的对象了
     DefaultListableBeanFactory beanFactory = this.beanFactory;
     if (beanFactory == null) {
        throw new IllegalStateException("BeanFactory not initialized or already closed - " +
              "call 'refresh' before accessing beans via the ApplicationContext");
     }
     return beanFactory;
  }
```

我们可以尝试获取一下：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
//我们可以直接获取此对象
System.out.println(context.getAutowireCapableBeanFactory());
```

正是因为这样，ApplicationContext才具有了管理和生产Bean对象的能力。

不过，我们的配置可能是XML、可能是配置类，那么Spring要如何进行解析，将这些变成对应的BeanDefinition对象呢？使用BeanDefinitionReader就可以：

```java
public static void main(String[] args) {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //比如我们要读取XML配置，我们直接使用XmlBeanDefinitionReader就可以快速进行扫描
    XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
    //加载此XML文件中所有的Bean定义到Bean工厂中
    reader.loadBeanDefinitions(new ClassPathResource("application.xml"));
    
    //可以看到能正常生产此Bean的实例对象
    System.out.println(factory.getBean(Student.class));
}
```

因此，针对于不同类型的配置方式，ApplicationContext有着多种实现，其中常用的有：

* ClassPathXmlApplicationContext：适用于类路径下的XML配置文件。
* FileSystemXmlApplicationContext：适用于非类路径下的XML配置文件。
* AnnotationConfigApplicationContext：适用于注解配置形式。

比如ClassPathXmlApplicationContext在初始化的时候就会创建一个对应的XmlBeanDefinitionReader进行扫描：

```java
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
   // 为给定的BeanFactory创建XmlBeanDefinitionReader便于读取XML中的Bean配置
   XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

   // 各种配置，忽略掉
   beanDefinitionReader.setEnvironment(this.getEnvironment());
   ...
   // 配置完成后，直接开始加载XML文件中的Bean定义
   loadBeanDefinitions(beanDefinitionReader);
}
```

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
   Resource[] configResources = getConfigResources();   //具体加载过程我就不详细介绍了
   if (configResources != null) {
      reader.loadBeanDefinitions(configResources);
   }
   String[] configLocations = getConfigLocations();
   if (configLocations != null) {
      reader.loadBeanDefinitions(configLocations);
   }
}
```

现在，我们就已经知道，Bean实际上是一开始通过BeanDefinitionReader进行扫描，然后将所有Bean以BeanDefinition对象的形式注册到对应的BeanFactory中进行集中管理，而我们使用的ApplicationContext实际上内部就有一个BeanFactory在进行Bean管理，这样容器才拥有了最基本的Bean管理功能。

当然，BeanFactory还可以具有父子关系，其中最关键的作用就是继承父容器中所有的Bean定义，这样的话，如果我们想要创建一个新的BeanFactory并且默认具有其他BeanFactory中所有的Bean定义外加一些其他的，那么就可以采用这种形式，这是很方便的。

我们可以来尝试一下，创建两个工厂：

```java
public class Main {
    public static void main(String[] args) {
        DefaultListableBeanFactory factoryParent = new DefaultListableBeanFactory();
        DefaultListableBeanFactory factoryChild = new DefaultListableBeanFactory();
        //在父工厂中注册A
        factoryParent.registerBeanDefinition("a", new RootBeanDefinition(A.class));
      	//在子工厂中注册B、C
        factoryChild.registerBeanDefinition("b", new RootBeanDefinition(B.class));
        factoryChild.registerBeanDefinition("c", new RootBeanDefinition(C.class));
        //最后设定子工厂的父工厂
        factoryChild.setParentBeanFactory(factoryParent);
    }
    
    static class A{ }
    static class B{ }
    static class C{ }
}
```

现在我们来看看是不是我们想的那样：

```java
System.out.println(factoryChild.getBean(A.class));  //子工厂不仅能获取到自己的，也可以拿到父工厂的
System.out.println(factoryChild.getBean(B.class));
System.out.println(factoryChild.getBean(C.class));

System.out.println(factoryParent.getBean(B.class));   //注意父工厂不能拿到子工厂的，就像类的继承一样
```

同样的，我们在使用ApplicationContext时，也可以设定这样的父子关系，效果相同：

```java
public static void main(String[] args) {
    ApplicationContext contextParent = new ClassPathXmlApplicationContext("parent.xml");
    ApplicationContext contextChild = new ClassPathXmlApplicationContext(new String[]{"child.xml"}, contextParent);  //第一个参数只能用数组，奇怪
}
```

当然，除了这些功能之外，BeanFactory还提供了很多其他的管理Bean定义的方法，比如移除Bean定义、拷贝Bean定义、销毁单例Bean实例对象等功能，这里就不一一列出了，各位小伙伴自己调用一下测试就可以了，很简单。

### 单例Bean的创建与循环依赖

前面我们讲解了配置的Bean是如何被读取并加载到容器中的，接着我们来了解一下Bean实例对象是如何被创建并得到的，我们知道，如果要得到一个Bean的实例很简单，通过`getBean`方法就可以直接拿到了：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
System.out.println(context.getBean(Student.class));   //通过此方法就能快速得到
```

那么，一个Bean的实例对象到底是如何创建出来的呢？我们还要继续对我们之前讲解的BeanFactory进行深入介绍。

我们可以直接找到BeanFactory接口的一个抽象实现`AbstractBeanFactory`类，它实现了`getBean()`方法：

```java
public Object getBean(String name) throws BeansException {
  	//套娃开始了，做好准备
    return this.doGetBean(name, (Class)null, (Object[])null, false);
}
```

那么我们`doGetBean()`接着来看方法里面干了什么，这个方法比较长，我们分段进行讲解：

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
    String beanName = this.transformedBeanName(name);   //虽然这里直接传的就是name，但是万一是别名呢，所以还得要解析一下变成原本的Bean名字
    Object sharedInstance = this.getSingleton(beanName);   //首先直接获取单例Bean对象
    Object beanInstance;
    if (sharedInstance != null && args == null) {   //判断是否成功获取到共享的单例对象
    ...
```

因为所有的Bean默认都是单例模式，对象只会存在一个，因此它会先调用父类的`getSingleton()`方法来直接获取单例对象，如果有的话，就可以直接拿到Bean的实例。如果Bean不是单例模式，那么会进入else代码块。这一部分我们先来看单例模式下的处理，其实逻辑非常简单：

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
    ...
    if (sharedInstance != null && args == null) {
        if (this.logger.isTraceEnabled()) {
          	//这里会判断Bean是否为正在创建状态，为什么会有这种状态呢？我们会在后面进行介绍
            if (this.isSingletonCurrentlyInCreation(beanName)) {
                this.logger.trace("Returning eagerly cached instance of singleton bean '" + beanName + "' that is not fully initialized yet - a consequence of a circular reference");
            } else {
                this.logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
            }
        }
				//这里getObjectForBeanInstance会进行最终处理
      	//因为Bean有两个特殊的类型，工厂Bena和空Bean，所以说需要单独处理
      	//如果是普通Bean直接原样返回beanInstance接收到最终结果
        beanInstance = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
    } else {
       ...
    }
  	//最后还会进行一次类型判断，如果都没问题，直接返回beanInstance作为结果，我们就得到Bean的实例对象了
  	return this.adaptBeanInstance(name, beanInstance, requiredType);
}
```

实际上整个单例Bean的创建路线还是很清晰的，并没有什么很难理解的地方，在正常情况下，其实就是简单的创建对象实例并返回即可。

其中最关键的是它对于循环依赖的处理。我们发现，在上面的代码中，得到单例对象后，会有一个很特殊的判断`isSingletonCurrentlyInCreation`，这个是干嘛的？对象不应该直接创建出来吗？为什么会有这种正在创建的状态呢？我们来探究一下。

开始之前先给大家提个问题：

> 现在有两个Bean，A和B都是以原型模式进行创建，而A中需要注入B，B中需要注入A，这时就会出现A还未创建完成，就需要B，而B这时也没创建完成，因为B需要A，而A等着B，B又等着A，这样就只能无限循环下去了（就像死锁那种感觉）所以就出现了循环依赖的问题（同理，一个对象注入自己，还有三个对象之间，甚至多个对象之间也会出现这种情况）

但是，在单例模式下，由于每个Bean只会创建一个实例，只要能够处理好对象之间的引用关系，Spring完全有机会解决单例对象循环依赖的问题。那么单例模式下是如何解决循环依赖问题的呢？

![image-20221217170912302](https://s2.loli.net/2022/12/17/aRjr1968Lc3BkKH.png)

我们回到一开始的`getSingleton()`方法中，研究一下它到底是如何处理循环依赖的，它是可以自动解决循环依赖问题的：

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
  	//先从第一层列表中拿Bean实例，拿到直接返回
    if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
      	//如果第一层拿不到，并且已经认定为处于循环状态，看看第二层有没有
        singletonObject = this.earlySingletonObjects.get(beanName);
      	//要是还是没有，继续往下
        if (singletonObject == null && allowEarlyReference) {
            synchronized(this.singletonObjects) {
              	//加锁再执行一次上述流程
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    if (singletonObject == null) {
                      	//仍然没有获取到实例，只能从singletonFactory中获取了
                        ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            singletonObject = singletonFactory.getObject();
                          	//丢进earlySingletonObjects中，下次就可以直接在第二层拿到了
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }

    return singletonObject;
}
```

看起来很复杂，实际上它使用了三级缓存的方式来处理循环依赖的问题，包括：

- singletonObjects，用于保存实例化、注入、初始化完成的 bean 实例
- earlySingletonObjects，用于保存实例化完成的 bean 实例
- singletonFactories，在初始创建Bean对象时都会生成一个对应的单例工厂用于获取早期对象

我们先来画一个流程图理清整个过程：

![image-20221218150012610](https://s2.loli.net/2022/12/18/xFfUuaozLpiVg96.png)

我们在了解这个流程之前，一定要先明确，单例Bean对象的获取，会有哪些结果，首先就是如果我们获取的Bean压根就没在工厂中注册，那得到的结果肯定是null；其次，如果我们获取的Bean已经注册了，那么肯定就可以得到这个单例对象，只是不清楚创建到哪一个阶段了。

现在我们根据上面的流程图，来模拟一下A和B循环依赖的情况：

![image-20230214222056632](https://s2.loli.net/2023/02/14/ezkOUv8Wjrb2tVF.png)

有的小伙伴就会有疑问了，看起来似乎两级缓存也可以解决问题啊，干嘛搞三层而且还搞个对象工厂？这不是多此一举吗？实际上这是为了满足Bean的生命周期而做的，通过工厂获取早期对象代码如下：

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
  	//这里很关键，会对一些特别的BeanPostProcessor进行处理，比如AOP代理相关的，如果这个Bean是被AOP代理的，我们需要得到的是一个经过AOP代理的对象，而不是直接创建出来的对象，这个过程需要BeanPostProcessor来完成（AOP产生代理对象的逻辑是在属性填充之后，因此只能再加一级进行缓冲）
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {
            exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);
        }
    }
    return exposedObject;
}
```

我们会在后面的部分中详细介绍BeanPostProcessor以及AOP的实现原理，届时各位再回来看就会明白了。

### 后置处理器与AOP

接着我们来介绍一下`PostProcessor`，它其实是Spring提供的一种后置处理机制，它可以让我们能够插手Bean、BeanFactory、BeanDefinition的创建过程，相当于进行一个最终的处理，而最后得到的结果（比如Bean实例、Bean定义等）就是经过后置处理器返回的结果，它是整个加载过程的最后一步。

而AOP机制正是通过它来实现的，我们首先来认识一下第一个接口`BeanPostProcessor`，它相当于Bean初始化的一个后置动作，我们可以直接实现此接口：

```java
//注意它后置处理器也要进行注册
@Component
public class TestBeanProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName);  //打印bean的名称
        return bean;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }
}
```

我们发现，此接口中包括两个方法，一个是`postProcessAfterInitialization`用于在Bean初始化之后进行处理，还有一个`postProcessBeforeInitialization`用于在Bean初始化之前进行处理，注意这里的初始化不是创建对象，而是调用类的初始化方法，比如：

```java
@Component
public class TestBeanProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("我是之后："+beanName);
        return bean;   //这里返回的Bean相当于最终的结果了，我们依然能够插手修改，这里返回之后是什么就是什么了
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("我是之前："+beanName);
        return bean;   //这里返回的Bean会交给下一个阶段，也就是初始化方法
    }
}
```

```java
@Component
public class TestServiceImpl implements TestService{

    public TestServiceImpl(){
        System.out.println("我是构造方法");
    }

    @PostConstruct
    public void init(){
        System.out.println("我是初始化方法");
    }

    TestMapper mapper;

    @Autowired
    public void setMapper(TestMapper mapper) {
        System.out.println("我是依赖注入");
        this.mapper = mapper;
    }
  	
  	...
```

而TestServiceImpl的加载顺序为：

```xml
我是构造方法
我是依赖注入
我是之前：testServiceImpl
我是初始化方法
我是之后：testServiceImpl
```

现在我们再来总结一下一个Bean的加载流程：

[Bean定义]首先扫描Bean，加载Bean定义 -> [依赖注入]根据Bean定义通过反射创建Bean实例 -> [依赖注入]进行依赖注入（顺便解决循环依赖问题）-> [初始化Bean]BeanPostProcessor的初始化之前方法 -> [初始化Bean]Bean初始化方法 -> [初始化Bean]BeanPostProcessor的初始化之后方法 -> [完成]最终得到的Bean加载完成的实例

利用这种机制，理解Aop的实现过程就非常简单了，AOP实际上也是通过这种机制实现的，它的实现类是`AnnotationAwareAspectJAutoProxyCreator`，而它就是在最后对Bean进行了代理，因此最后我们得到的结果实际上就是一个动态代理的对象（有关详细实现过程，这里就不进行列举了，感兴趣的可以继续深入）因此，实际上之前设计的三层缓存，都是由于需要处理AOP设计的，因为在Bean创建得到最终对象之前，很有可能会被PostProcessor给偷梁换柱！

那么肯定有人有疑问了，这个类没有被注册啊，那按理说它不应该参与到Bean的初始化流程中的，为什么它直接就被加载了呢？

还记得`@EnableAspectJAutoProxy`吗？我们来看看它是如何定义就知道了：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({AspectJAutoProxyRegistrar.class})
public @interface EnableAspectJAutoProxy {
    boolean proxyTargetClass() default false;

    boolean exposeProxy() default false;
}
```

我们发现它使用了`@Import`来注册`AspectJAutoProxyRegistrar`，那么这个类又是什么呢，我们接着来看：

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {
    AspectJAutoProxyRegistrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
      	//注册AnnotationAwareAspectJAutoProxyCreator到容器中
        AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
        AnnotationAttributes enableAspectJAutoProxy = AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
        if (enableAspectJAutoProxy != null) {
            if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
                AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
            }

            if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
                AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
            }
        }

    }
}
```

它实现了接口，这个接口也是Spring提供的一种Bean加载机制，它支持直接向容器中添加Bean定义，容器也会加载这个Bean：

- ImportBeanDefinitionRegistrar类只能通过其他类@Import的方式来加载，通常是启动类或配置类。
- 使用@Import，如果括号中的类是ImportBeanDefinitionRegistrar的实现类，则会调用接口中方法（一般用于注册Bean）
- 实现该接口的类拥有注册bean的能力。

我们可以看到此接口提供了一个`BeanDefinitionRegistry`正是用于注册Bean的定义的。

因此，当我们打上了`@EnableAspectJAutoProxy`注解之后，首先会通过`@Import`加载AspectJAutoProxyRegistrar，然后调用其`registerBeanDefinitions`方法，然后使用工具类注册AnnotationAwareAspectJAutoProxyCreator到容器中，这样在每个Bean创建之后，如果需要使用AOP，那么就会通过AOP的后置处理器进行处理，最后返回一个代理对象。

我们也可以尝试编写一个自己的ImportBeanDefinitionRegistrar实现，首先编写一个测试Bean：

```java
public class TestBean {
    
    @PostConstruct
    void init(){
        System.out.println("我被初始化了！");
    }
}
```

```java
public class TestBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        BeanDefinition definition = BeanDefinitionBuilder.rootBeanDefinition(Student.class).getBeanDefinition();
        registry.registerBeanDefinition("lbwnb", definition);
    }
}
```

观察控制台输出，成功加载Bean实例。

与`BeanPostProcessor`差不多的还有`BeanFactoryPostProcessor`，它和前者一样，也是用于我们自己处理后置动作的，不过这里是用于处理BeanFactory加载的后置动作，`BeanDefinitionRegistryPostProcessor`直接继承自`BeanFactoryPostProcessor`，并且还添加了新的动作`postProcessBeanDefinitionRegistry`，你可以在这里动态添加Bean定义或是修改已经存在的Bean定义，这里我们就直接演示`BeanDefinitionRegistryPostProcessor`的实现：

```java
@Component
public class TestDefinitionProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        System.out.println("我是Bean定义后置处理！");
        BeanDefinition definition = BeanDefinitionBuilder.rootBeanDefinition(TestBean.class).getBeanDefinition();
        registry.registerBeanDefinition("lbwnb", definition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println("我是Bean工厂后置处理！");
    }
}
```

在这里注册Bean定义其实和之前那种方法效果一样。

最后，我们再完善一下Bean加载流程（加粗部分是新增的）：

[Bean定义]首先扫描Bean，加载Bean定义 -> **[Bean定义]Bean定义和Bean工厂后置处理** -> [依赖注入]根据Bean定义通过反射创建Bean实例 -> [依赖注入]进行依赖注入（顺便解决循环依赖问题）-> [初始化Bean]BeanPostProcessor的初始化之前方法 -> [初始化Bean]Bean初始化方法 -> [初始化Bean]BeanPostProcessor的初始化之前后方法 -> [完成]最终得到的Bean加载完成的实例

### 应用程序上下文详解

前面我们详细介绍了BeanFactory是如何工作的，接着我们来研究一下ApplicationContext的内部，实际上我们真正在项目中使用的就是ApplicationContext的实现，那么它又是如何工作的呢。

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
	@Nullable
	String getId();
	String getApplicationName();
	String getDisplayName();
	long getStartupDate();
	@Nullable
	ApplicationContext getParent();
	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```

它本身是一个接口，同时集成了多种类型的BeanFactory接口，说明它应该具有这些BeanFactory的能力，实际上我们在前面已经提到过，ApplicationContext是依靠内部维护的BeanFactory对象来完成这些功能的，并不是它本身就实现了这些功能。

这里我们就先从构造方法开始走起，以我们常用的AnnotationConfigApplicationContext为例：

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
		this();                      //1. 首先会调用自己的无参构造
		register(componentClasses);  //2. 然后注册我们传入的配置类
		refresh();                   //3. 最后进行刷新操作（关键）
}
```

先来看第一步：

```java
public GenericApplicationContext() {
  	//父类首先初始化内部维护的BeanFactory对象
		this.beanFactory = new DefaultListableBeanFactory();
}
```

```java
public AnnotationConfigApplicationContext() {
		StartupStep createAnnotatedBeanDefReader = this.getApplicationStartup().start("spring.context.annotated-bean-reader.create");
  	//创建AnnotatedBeanDefinitionReader对象，用于后续处理 @Bean 注解
		this.reader = new AnnotatedBeanDefinitionReader(this);
		createAnnotatedBeanDefReader.end();
  	//创建ClassPathBeanDefinitionScanner对象，用于扫描类路径上的Bean
		this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

这样，AnnotationConfigApplicationContext的基本内容就初始化好了，不过这里结束之后会将ConfigurationClassPostProcessor后置处理器加入到BeanFactory中，它继承自BeanFactoryPostProcessor，也就是说一会会在BeanFactory初始化完成之后进行后置处理：

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		Assert.notNull(environment, "Environment must not be null");
		this.registry = registry;
		this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
  	//这里注册了注解处理配置相关的后置处理器
		AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

实际上这个后置处理器的主要目的就是为了读取配置类中的各种Bean定义以及其他注解，比如@Import、@ComponentScan等。

同时这里也会注册一个AutowiredAnnotationBeanPostProcessor后置处理器到BeanFactory，它继承自BeanPostProcessor，用于处理后续生成的Bean对象，其实看名字就知道，这玩意就是为了处理@Autowire、@Value这种注解，用于自动注入，这里就不深入讲解具体实现了。

所以，第一步结束之后，就会有这两个关键的后置处理器放在容器中：

![image-20230719152546148](https://s2.loli.net/2023/07/19/uY4zwEhArUMfP2d.png)

接着是第二步，注册配置类：

```java
@Override
public void register(Class<?>... componentClasses) {
		Assert.notEmpty(componentClasses, "At least one component class must be specified");
		StartupStep registerComponentClass = this.getApplicationStartup().start("spring.context.component-classes.register")
				.tag("classes", () -> Arrays.toString(componentClasses));
  	//使用我们上面创建的Reader注册配置类
		this.reader.register(componentClasses);
		registerComponentClass.end();
}
```

现在配置类已经成功注册到IoC容器中了，我们接着来看第三步，到目前为止，我们已知的仅仅是注册了配置类的Bean，而刷新操作就是配置所有Bean的关键部分了，刷新操作是在 AbstractApplicationContext 中实现的：

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
			// 准备当前应用程序上下文，进行刷新、设置启动事件和活动标志以及执行其他初始化
			prepareRefresh();
			// 这个方法由子类实现，对内部维护的BeanFactory进行刷新操作，然后返回这个BeanFactory
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			// 初始化配置Bean工厂，比如一些会用到的类加载器和后置处理器。
			prepareBeanFactory(beanFactory);
			try {
				// 由子类实现对BeanFactory的其他后置处理，目前没有看到有实现
				postProcessBeanFactory(beanFactory);
				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// 实例化并调用所有注册的 BeanFactoryPostProcessor 类型的Bean
        // 这一步中，上面提到的BeanFactoryPostProcessor就开始工作了，比如包扫描、解析Bean配置等
        // 这一步结束之后，包扫描到的其他Bean就注册到BeanFactory中了
				invokeBeanFactoryPostProcessors(beanFactory);
				// 实例化并注册所有 BeanPostProcessor 类型的 Bean，不急着执行
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();
				initMessageSource();
				initApplicationEventMulticaster();
				// 依然是提供给子类实现的，目的是用于处理一些其他比较特殊的Bean，目前似乎也没看到有实现
				onRefresh();
				// 注册所有的监听器
				registerListeners();
				// 将剩余所有非懒加载单例Bean全部实例化
				finishBeanFactoryInitialization(beanFactory);
				finishRefresh();
			} catch (BeansException ex) {
				...
				// 发现异常直接销毁所有Bean
				destroyBeans();
				// 取消本次刷新操作，重置标记
				cancelRefresh(ex);
				// 继续往上抛异常
				throw ex;
			} finally {
				resetCommonCaches();
				contextRefresh.end();
			}
		}
}
```

所以，现在流程就很清晰了，实际上最主要的就是`refresh`方法，它从初始化到实例化所有的Bean整个流程都已经完成，在这个方法结束之后，整个IoC容器基本就可以正常使用了。

我们继续来研究一下`finishBeanFactoryInitialization`方法，看看它是怎么加载所有Bean的：

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		...
		beanFactory.preInstantiateSingletons();   //套娃
}
```

```java
	@Override
	public void preInstantiateSingletons() throws BeansException {
		...
    // 列出全部bean名称
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
		// 开始初始化所有Bean
		for (String beanName : beanNames) {
      //得到Bean定义
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
      //Bean不能是抽象类、不能是非单例模式、不能是懒加载的
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
        //针对于Bean和FactoryBean分开进行处理
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof SmartFactoryBean<?> smartFactoryBean && smartFactoryBean.isEagerInit()) {
						getBean(beanName);
					}
				} else {
					getBean(beanName);  //最后都是通过调用getBean方法来初始化实例，这里就跟我们之前讲的连起来了
				}
			}
		}

		...
}
```

至此，关于Spring容器核心加载流程，我们就探究完毕了，实际 单易懂，就是代码量太大了。在后续的SpringBoot阶段，我们还会继续深挖Spring的某些机制的具体实现细节。

### Mybatis整合原理

通过之前的了解，我们再来看Mybatis的`@MapperScan`是如何实现的，现在理解起来就非常简单了。

我们可以直接打开查看：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({MapperScannerRegistrar.class})
@Repeatable(MapperScans.class)
public @interface MapperScan {
    String[] value() default {};

    String[] basePackages() default {};
  	...
```

我们发现，和Aop一样，它也是通过Registrar机制，通过`@Import`来进行Bean的注册，我们来看看`MapperScannerRegistrar`是个什么东西，关键代码如下：

```java
void registerBeanDefinitions(AnnotationMetadata annoMeta, AnnotationAttributes annoAttrs, BeanDefinitionRegistry registry, String beanName) {
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);
    builder.addPropertyValue("processPropertyPlaceHolders", true);
    ...
}
```

虽然很长很多，但是这些代码都是在添加一些Bean定义的属性，而最关键的则是最上方的`MapperScannerConfigurer`，Mybatis将其Bean信息注册到了容器中，那么这个类又是干嘛的呢？

```java
public class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware {
    private String basePackage;
```

它实现了BeanDefinitionRegistryPostProcessor，也就是说它为Bean信息加载提供了后置处理，我们接着来看看它在Bean信息后置处理中做了什么：

```java
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    if (this.processPropertyPlaceHolders) {
        this.processPropertyPlaceHolders();
    }

  	//初始化类路径Mapper扫描器，它相当于是一个工具类，可以快速扫描出整个包下的类定义信息
  	//ClassPathMapperScanner是Mybatis自己实现的一个扫描器，修改了一些扫描规则
    ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
    scanner.setAddToConfig(this.addToConfig);
    scanner.setAnnotationClass(this.annotationClass);
    scanner.setMarkerInterface(this.markerInterface);
    scanner.setSqlSessionFactory(this.sqlSessionFactory);
    scanner.setSqlSessionTemplate(this.sqlSessionTemplate);
    scanner.setSqlSessionFactoryBeanName(this.sqlSessionFactoryBeanName);
    scanner.setSqlSessionTemplateBeanName(this.sqlSessionTemplateBeanName);
    scanner.setResourceLoader(this.applicationContext);
    scanner.setBeanNameGenerator(this.nameGenerator);
    scanner.setMapperFactoryBeanClass(this.mapperFactoryBeanClass);
    if (StringUtils.hasText(this.lazyInitialization)) {
        scanner.setLazyInitialization(Boolean.valueOf(this.lazyInitialization));
    }

    if (StringUtils.hasText(this.defaultScope)) {
        scanner.setDefaultScope(this.defaultScope);
    }

  	//添加过滤器，这里会配置为所有的接口都能被扫描（因此即使你不添加@Mapper注解都能够被扫描并加载）
    scanner.registerFilters();
  	//开始扫描
    scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ",; \t\n"));
}
```

开始扫描后，会调用`doScan()`方法，我们接着来看（这是`ClassPathMapperScanner`中的扫描方法）：

```java
public Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);
  	//首先从包中扫描所有的Bean定义
    if (beanDefinitions.isEmpty()) {
        LOGGER.warn(() -> {
            return "No MyBatis mapper was found in '" + Arrays.toString(basePackages) + "' package. Please check your configuration.";
        });
    } else {
      	//处理所有的Bean定义，实际上就是生成对应Mapper的代理对象，并注册到容器中
        this.processBeanDefinitions(beanDefinitions);
    }

  	//最后返回所有的Bean定义集合
    return beanDefinitions;
}
```

通过断点我们发现，最后处理得到的Bean定义发现此Bean是一个MapperFactoryBean，它不同于普通的Bean，FactoryBean相当于为普通的Bean添加了一层外壳，它并不是依靠Spring直接通过反射创建，而是使用接口中的方法：

```java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    @Nullable
    T getObject() throws Exception;

    @Nullable
    Class<?> getObjectType();

    default boolean isSingleton() {
        return true;
    }
}
```

通过`getObject()`方法，就可以获取到Bean的实例了。

注意这里一定要区分FactoryBean和BeanFactory的概念：

- BeanFactory是个Factory，也就是 IOC 容器或对象工厂，所有的 Bean 都是由 BeanFactory( 也就是 IOC 容器 ) 来进行管理。
- FactoryBean是一个能生产或者修饰生成对象的工厂Bean(本质上也是一个Bean)，可以在BeanFactory（IOC容器）中被管理，所以它并不是一个简单的Bean。当使用容器中factory bean的时候，该容器不会返回factory bean本身，而是返回其生成的对象。要想获取FactoryBean的实现类本身，得在getBean(String BeanName)中的BeanName之前加上&,写成getBean(String &BeanName)。

我们也可以自己编写一个实现：

```java
@Component("test")
public class TestFb implements FactoryBean<Student> {
    @Override
    public Student getObject() throws Exception {
        System.out.println("获取了学生");
        return new Student();
    }

    @Override
    public Class<?> getObjectType() {
        return Student.class;
    }
}
```

```java
public static void main(String[] args) {
    log.info("项目正在启动...");
    ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);
    System.out.println(context.getBean("&test"));   //得到FactoryBean本身（得加个&搞得像C语言指针一样）
    System.out.println(context.getBean("test"));   //得到FactoryBean调用getObject()之后的结果
}
```

因此，实际上我们的Mapper最终就以FactoryBean的形式，被注册到容器中进行加载了：

```java
public T getObject() throws Exception {
    return this.getSqlSession().getMapper(this.mapperInterface);
}
```

这样，整个Mybatis的`@MapperScan`的原理就全部解释完毕了。

在了解完了Spring的底层原理之后，我们其实已经完全可以根据这些实现原理来手写一个Spring框架了。
