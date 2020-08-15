
### `JAVA`
#### Java的反射机制
`写在Spring Ioc之前`
类装载器就是寻找类的字节码文件并构造出类在JVM内部表示的对象组件，主要工作由ClassLoader及其子类负责，ClassLoader是一个重要的java运行时系统组件，他负责在运行时查找和装入Class字节码文件

```
public class ReflectCar {
    public static Car initCarByDefaultConst() throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        ClassLoader loader=Thread.currentThread().getContextClassLoader();
        Class clazz=loader.loadClass("com.beijinghuayi.ioc.Car");
        //获取类默认实例化对象
        Constructor constructor=clazz.getDeclaredConstructor((Class[])null);
        Car car= (Car) constructor.newInstance();

        Method setBrand=clazz.getMethod("setBrand",String.class);
        setBrand.invoke(car,"奔驰");
        Method setColor=clazz.getMethod("setColor",String.class);
        setColor.invoke(car,"红色");
        Method setMaxspeed=clazz.getMethod("setMaxspeed",String.class);
        setMaxspeed.invoke(car,"200码");
        return car;
    }
    public static Car initCarByParams() throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        ClassLoader loader=Thread.currentThread().getContextClassLoader();
        Class clazz=loader.loadClass("com.beijinghuayi.ioc.Car");
        Constructor constructor=clazz.getDeclaredConstructor(new Class[]{String.class,String.class,String.class});
        Car car= (Car) constructor.newInstance(new Object[]{"红色","宝马","180"});
        return car;
    }
}
```
`工作机制：`
- 1.装载：查找和导入Class文件
- 2.链接：执行校验，准备和解析步骤
- 3.初始化：对类的静态变量、静态代码块执行初始化操作

`Tip:`JVM在运行时会产生3个`classLoader`
- 根装载器（C++实现、不是classloader的子类）装载jre核心类库
- ExtClassLoader（扩展类装载器）装载jre扩展目录ext中的jar类
- AppClassLoader（系统类装载器）装载classpath中的内容

`ClassLoader`重要方法
- Class loadClass(String name);从文件中装在类
- Class defineClass(String name,byte[]b,int off,int len)
- Class findSystemClass(String name)
- Class findLoadedClass(String name)
- ClassLoader getParent()

Class反射对象描述类语义结构，可以从Class对象中获得构造函数，成员变量，方法等元素的反射对象，并以编程的方法通过这些反射对象对目标类对象进行操作。这些反射对象类在java.reflect包中定义，下面是最主要的三个反射类
- 1.Constructor类对象的反射类（通过getConstructor方法可以获得类的所有构造函数反射对象数组）NewInstance。
- 2.Method类方法的反射类invoke() getReturnType(),getParameterTypes();
- 3.Field 获取类的成员变量反射类（ 获取成员变量反射数组）
`Tip`访问private，protect成员变量或方法时需添加`Field.setAccessible(true)`，`Method.setAccessible(true)`方法取消java语言检查，否则将会抛出`IllegalAccessException`异常.
```
 ClassLoader loader=Thread.currentThread().getContextClassLoader();
        System.out.println("classLoader:"+loader);
        System.out.println("parent classLoader:"+loader.getParent());
        System.out.println("grandParent classLoader:"+loader.getParent().getParent());
```
Spring的IoC的实现原理利用的就是java的反射机制，Spring的工厂类会帮助我们完成配置文件的读取、利用反射机制注入对象工作，我们可以通过bean的名称获得对应的对象。
##### 手动创建自定义spring工厂类
```
public class BeanFactory {
    private Map<String,Object> beanmap=new HashMap<>();
    public void init(String xml) throws DocumentException, ClassNotFoundException, IntrospectionException, IllegalAccessException, InstantiationException, InvocationTargetException {
        //1.创建读取配置文件的reader对象
        SAXReader reader=new SAXReader();
        //2.获取当前线程的类加载器
        ClassLoader loader=Thread.currentThread().getContextClassLoader();
        //3.从class目录下获取指定的xml文件
        InputStream ins=loader.getResourceAsStream(xml);
        Document doc=reader.read(ins);
        Element root=doc.getRootElement();
        Element foo;

        //4.遍历xml文件中的Bean实例
        for(Iterator i=root.elementIterator("bean");i.hasNext();){
            foo= (Element) i.next();
            //5.针对每一个bean实例，获取bean的属性id和class
            Attribute id=foo.attribute("id");
            Attribute cls=foo.attribute("class");

            //6.利用Java反射机制，通过class的名称获取Class对象
            Class bean=Class.forName(cls.getText());
            //7.获取丢应class信息
            java.beans.BeanInfo info =java.beans.Introspector.getBeanInfo(bean);
            //8.获取其属性描述
            java.beans.PropertyDescriptor pd[]=info.getPropertyDescriptors();
            //9.创建一个对象，并在接下来的代码中为对象的属性赋值
            Object obj=bean.newInstance();
            //10.遍历该bean的property属性
            for(Iterator ite=foo.elementIterator("property");ite.hasNext();){
                Element foo2= (Element) ite.next();
                //11.获取该property的name属性
                Attribute name=foo2.attribute("name");
                String value=null;
                //12.获取该property的子元素的值
                for (Iterator ite1 = foo2.elementIterator("value"); ite1.hasNext();)
                {
                    Element node = (Element) ite1.next();
                    value = node.getText();
                    break;
                }

                //13.利用Java的反射机制调用对象的某个set方法，并将值设置进去
                for (int k = 0; k < pd.length; k++) {
                    if (pd[k].getName().equalsIgnoreCase(name.getText()))
                    {
                        Method mSet = null;
                        mSet = pd[k].getWriteMethod();
                        mSet.invoke(obj, value);
                    }
                }
            }
            //14.将对象放入beanMap中，其中key为id值，value为对象
            beanmap.put(id.getText(), obj);
        }
    }
    /**
     * 通过bean的id获取bean的对象.
     *
     * @param beanName
     *            bean的id
     * @return 返回对应对象
     */
    public Object getBean(String beanName) {
        Object obj = beanmap.get(beanName);
        return obj;
    }
}
```
`config.xml`
```
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    <bean id="javaBean" class="JavaBean">
        <property name="username">
            <value>mic_swift</value>
        </property>
        <property name="password">
            <value>010101010110</value>
        </property>
    </bean>
</beans>
```

##### 引出Spring Ioc
在Spring中，通过IOC可以将实现类、参数信息等配置在对应的配置文件中，那么当需要更改实现类或参数信息时，只需要修改配置文件即可，我们还可以对某对象所需要的其他对象进行注入，这种注入都是在配置文件中实现。
##### Spring资源访问工具类
JDK所提供的访问资源类并不能很好的满足各种底层资源的访问需求，因此，Spring设计了一个Resource接口，它为应用提供了更强大的访问底层资源的能力：
 `主要方法：`
 - boolean exists()
 - boolean isOpen()
 - URL getURL();
 - File getFile();
 - inputStream getInputStream();

`具体实现类`
- ByteArrayResource
- ClassPathResource
- FileSystemResource
- InputStreamResource
- ServletContextResource
- UrlResource

为了访问不同类型的资源，必须使用相应的Resource实现类，Spring提供了一个强大的加载资源的机制，能够自动识别不同的资源类型。
`资源类型地址前缀`
- classpath classpath:com/example/config.xml
- File file:/com/example/config.xml
- Http http://www.baidu.com
- Ftp ftp://www.baidu.com
- 无前缀 com/example/config.xml
###### BeanFactory和ApplicationContext
BeanFactory时Spring框架的最核心接口，它提供高级的Ioc配置机制，AppliactionContext建立在BeanFactory基础上，提供了更多面向应用的功能，它提供了国际化支持和框架事件体系，更易于创建实际应用一般成BeanFactory为Ioc容器，而称ApplicationContext为应用上下文

BeanFactory是一个类工厂，可以常见并管理各种类的对象，Spring称这些创建和管理的java对象为bean，在Spring中，java对象的范围更加宽泛，接下来我们对BeanFactory的类体系结构以及装载初始化顺序进行说明：
###### 类体系结构
- XmlBeanFactory
- ListableBeanFactory
- HierarhicalBeanFactory
- ConfigurableBeanFactory
- AutowireCapableBeanFactory
- SingletonBeanFactory
- BeanDefinitionRegistry
##### 初始化顺序
- 创建配置文件
- 装载配置文件
- 启动Ioc容器
- 获取Bean实例

ApplicationContext由BeanFactory派生而来，提供了更多面向实际的功能。在BeanFactory中，很多功能需要以编程的方式实现，而在ApplicationContext中则可以通过配置的方式实现，接下来介绍一下ApplicationContext的实现类以及类体系结构：

`具体实现类`
- ClassPathXmlApplicationContext
- FileSystemXmlApplicationContext
- ConfigurableApplicationContext
`类继承体系`(扩展接口)
- ApplicationEventPublisher
- MessageSource
- ResourcePatternResolver
- LifeCycle(用于处理异步)

Spring容器中的Bean拥有明确的生命周期，由多个特定的生命阶段组成，每个生命阶段都允许外界对Bean进行控制，在Spring中，我们从Bean的作用范围和实例化Bean时所经历的一系列阶段来描述Bean的生命周期
- BeanFactory中的Bean的生命周期
- ApplicationContext中的Bean的生命周期

###### Spring容器启动基本条件
- Spring框架类包
- Bean配置信息
- Bean类满足

###### Bean的元数据信息
- Bean的实现类
- Bean的属性信息
- Bean的依赖关系
- Bean的行为配置
- Bean的创建方式

使用静态工厂的方式除了指定必须的class属性，还要指定factory-method属性来指定实例化Bean的方法，而且使用静态工厂方法也允许指定方法参数，SpringIoc容器将调用此属性的方法来获取Bean。

使用实例工厂方法不能指定clas属性，此时必须使用factory-bean来指定工厂Bean，factory-method属性指定实例化Bean的方法，而且使用实例工厂方法允许指定方法参数，方式和使用构造器方式一样
```
<bean id="beanInstanceFactory" class="com.beijinghuayi.spring.Instance"/>
	<bean id="helloWorldInstance" factory-bean="beanInstanceFactory" factory-method="newInstance">
		<constructor-arg index="0" value="Hello Instance Factory!"></constructor-arg>
</bean>
```
##### Spring多个配置文件的整合
```
<beans>
    <import resource="common/Spring-Common.xml"/>
    <import resource="common/Spring-Connect.xml"/>
    <import resource="common/Spring-Moudel.xml"/>
</beans>
```

#### JSP
##### JSP内置对象
|JSP内置对象|对象描述|
|---|---|
|Out|向客户端对象输出数据|
|Request|向服务器提交数据|
|Response|服务器项目信息|
|Exception|异常信息|
|Config|配置信息|
|Page|指向当前page本身|
|Session|保存会话信息（统一用户的不同对象之间共享信息）|
|Application|上下文（不同用户间共享信息）|
|PageContext|队jsp页面所有对象及命名空间的访问|
##### out对象
`out.flush()`
`out.clearBuffer`
`out.clear`
```
out.print("获取当前缓冲区大小："+out.getBufferSize());
out.print("当前缓冲区剩余字节数目"+out.getRemaining());
```
##### Request对象
|Request对象|对象描述|
|---|---|
|||


##### 简单语法格式整理
```
<jsp:forward page="login.jsp">
    <jsp:param value="jikexueyuan" name="username"/>
    <jsp:param value="sunxiaohang" name="password"/>
</jsp:forward>
<%@page errorPage="error_page.jsp" isErrorPage="true" %>


  String username=request.getParameter("username");
    String password=request.getParameter("password");
    out.println("username "+username);
    out.println("<br/>");
    out.println("password "+password);

<jsp:include page="body.jsp">
    <jsp:param name="bgcolor" value="red"/>
</jsp:include>

<body bgcolor="<%=request.getParameter("bgcolor")%>">
</body>

<jsp:useBean id="person" class="Person"></jsp:useBean>
<jsp:setProperty property="name" name="person"/>
<jsp:setProperty property="sex" name="person"/>

<jsp:getProperty property="name" name="person"/>
<jsp:getProperty property="sex" name="person"/>
需要添加statdard.jar jstl.jar包
```
##### JSP内置对象
```
response.setHeader("Cache-Control","no-cache");//无缓存
response.setIntHeader("refresh",2);//设置两秒钟自动刷新
out.print("this date is :"+new java.util.Date().toString()+"<br>");
response.sendRedirect("https://www.baidu.com");//重定向页面
Cookie cookie=new Cookie("username","password");
cookie.setMaxAge(3600);
response.addCookie(cookie);
session.getId();
```
#### Spring概述
#####面向`方面`的程序设计（AOP）
Spring框架的一个关键组件是面向方面的程序设计（AOP）框架。一个程序中跨越多个点的功能被称为`横切关注点`，其在概念上独立于应用程序的业务逻辑（sample：日志记录、声明性事务）

`OOP`中模块化的关键单元是类，`AOP`中模块化的关键单元室方面。`AOP`帮助你将横切关注点从他们所影响的对象中分离出来，`依赖注入`帮助你将你的应用程序对象从彼此中分离出来。

- `控制反转IOC`
在编写一个复杂的java程序时应用程序类应当尽可能的独立于其他的java类来增加这些类的可重用性，当进行单元测试时，可以使他们独立于其他类进行测试。

- `依赖注入DI`
依赖注入可以以向构造函数传递参数的方式发生，或者通过使用setter方法post-construction。

##### Spring体系结构
![Alt text](./1521077784467.png)
#####  核心容器
核心容器由核心、bean、上下文的表达式语言模块组成
- `核心`提供框架的基本组成部分、包括IOC和依赖注入功能。
- `Bean`提供BeanFactory，工厂模式的复杂实现。
- `上下文`建立在由core和bean提供的坚实基础上，他是访问定义和配置的任何对象的媒介。ApplicationContext接口是上下文模块的重点。
- `表达式语言`模块在运行时提供了查询和操作一个对象图的强大的表达式语言

##### 数据集成、访问
- `JDBC`提供删除冗余的JDBC相关编码的JDBC抽象层
- `ORC`为流行的对象关系映射API，包括JPA、JDO、Hibernate和iBatis提供集成层
- `OXM`提供抽象层、它支持对JAXB、Castor、XMLBeans，JiBx和XStream的对象/XML映射实现
- `JMS`java消息服务包含生产和消费的信息的功能
- `事务`事务模块为实现特殊接口的类及所有的POJO支持编程式和声明式事务管理

##### Web
- `Web`提供基本的面向web的集成功能，例如多个文件上传的功能和使用servlet监听器和面向web应用程序的上下文来初始化IOC容器
- `Web-MVC`包含Spring的模型-视图-控制（MVC），实现了web应用程序
- `Web-Socket`为`WebSocket-Based`提供支持，而且在web应用程序中提供客户端和服务器端之间的通信的两种方式。
- `Web-Portlet`提供在portlet环境中实现MVC，并且反映了Web-Servlet模块的功能。

##### Spring实例
- 1.生成工厂对象，加载完指定路径下bean配置文件，利用框架提供的`FileSystemXmlApplicationContext` API生成工厂bean`FileSystemXmlApplicationContext`负责生成和初始化所有对象，比如：所有XML bean配置文件中的bean
- 利用第一步生成的上下文中的`getBean()`方法得到所需要的bean，这个方法通过配置中的`beanID`来返回一个真正的对象，一旦得到这个对象就可以利用这个对象条用任何方法。

tip:`<!--property需要提供set方法才能使用-->`
```
ApplicationContext applicationContext=new ClassPathXmlApplicationContext("/configure/beansConfigure.xml");
HelloWord helloWord= (HelloWord) applicationContext.getBean("helloWord");
helloWord.getMessage();
```
##### bean property
|属性|描述|
| --- | --- |
|class|强制属性，制定用来创建bean的bean类|
|name/id|指定唯一的bean的标识符|
|scope|指定由特定bean定义创建的对象的作用域|
|constructor-arg|用来注入依赖关系|
|properties|用来注入依赖关系|
|autowiring mode|用来注入依赖关系|
|lazy-initialization mode|第一次调用时创建对象（懒汉模式）|
|initialization|在bean的所有必须属性被容器设置之后，调用回调方法|
|destruction|当包含该bean的容器被销毁是，使用回调方法|
##### Spring配置元数据
`Spring IoC`容器完全由实际编写的配置元数据的格式解耦
- 基于XML的配置文件
- 基于注解的配置
- 基于java的配置

##### Spring Bean的作用域
在Spring定义一个bean时，必须声明该bean的作用域.
|作用域|描述|
|---|---|
|`singleTon`|单例模式|
|`prototype`|普通模式，每次调用创建一个新的对象|
|`request`|作用域定义为HTTP请求，只在web-aware spring ApplicationContext的上下文中有效|
|`session`|作用域定义限制为HTTP会话，只在web-aware spring ApplicationContext的上下文中有效|
|`global-session`|作用域将bean的定义限制为全局HTTP绘画，只在web-aware spring ApplicationContext的上下文中有效|
作用域设置实例
```
 <bean id="singleTon" class="com.example.SingleTon" 
      scope="singleton">
```
##### Spring bean的生命周期
声明带有 `init-method `和 `destroy-method` 参数的 。
- `init-method` 属性指定一个方法，在实例化 bean 时调用该方法。
- `destroy-method` 指定一个方法，只有从容器中移除 bean 后，才能调用该方法。


在`org.springframework.beans.factory.InitializingBean` 接口指定一个单一的方法：
```
void afterPropertiesSet() throws Exception;
```
我们只需要在实现`InitializingBean`接口就可以在对象创建后做一些事情
```
public class TestBean implements InitializingBean {
   public void afterPropertiesSet() throws Exception{
      // do some initialization work
   }
}
```
同样的在`org.springframework.beans.factory.DisposableBean` 接口指定一个单一的方法：
```
void destroy() throws Exception;
```
然后在类对象中实现`DisposableBean`接口
```
public class TestBean implements DisposableBean{
    @Override
    public void destroy() throws Exception {
        // do some initialization work
    }
}
```
除此之外，在基于XML元数据配置的境况下还可以通过设置`destroy-method`属性实现
```
<bean id="exampleBean"
         class="examples.ExampleBean" destroy-method="destroy"/>
```
在类里面我们可以这样定义
```
public class TestBean {
   public void destroy() {
      // do some destruction work
   }
}
``` 
##### BeanPostProcessor 
有时候会需要在bean实例化对象前后去做一些准备工作或预处理，可以在创建bean类时实现`BeanPostProcessor`接口去完成自定义工作。
该接口定义了`postProcessBeforeInitialization`和`postProcessAfterInitialization`两个带参函数，示例代码如下
```
public class TestBean implements BeanPostProcessor {
   public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("BeforeInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
   public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("AfterInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
}
```
`注意`：在main方法中需要注册一个在 AbstractApplicationContext 类中声明的关闭 hook 的 registerShutdownHook() 方法。它将确保正常关闭，并且调用相关的 destroy 方法。
```
context.registerShutdownHook();
```
##### Spring Bean 定义继承
- Bean可以通过设置配置文件来定义继承关系
- 子 bean 的定义继承父定义的配置数据。子定义可以根据需要重写一些值，或者添加其他值。`tip:`Spring Bean 定义的继承与 Java 类的继承无关，但是继承的概念是一样的。
- 当你使用基于 XML 的配置元数据时，通过使用父属性，指定父 bean 作为该属性的值来表明子 bean 的定义。

通过XML配置文件实现继承关系实例
```
<bean id="person" class="com.example.Person">
      <property name="name" value="swift"/>
      <property name="details" value="animal can speak"/>
   </bean>

   <bean id="student" class="com.example.Student" parent="person">
      <property name="name" value="study"/>
      <property name="details" value="good good study, day day up!"/>
   </bean>
```
`tip：`在定义Student类的时候我们不再需要继承Person类
##### Bean 定义模板
```
<bean id="person" abstract="true">
      <property name="name" value="swift"/>
      <property name="details" value="animal can speak"/>
   </bean>

   <bean id="student" class="com.example.Student" parent="person">
      <property name="name" value="study"/>
      <property name="details" value="good good study, day day up!"/>
   </bean>
```
person 自身不能被实例化，因为它是不完整的(没有指定class属性)，而且它也被明确地标记为抽象的。当一个定义是抽象的，它仅仅作为一个纯粹的模板 bean 定义来使用的，充当子定义的父定义使用。
##### Spring 依赖注入
当编写一个复杂的 Java 应用程序时，应用程序的 java 有多个对象，应用程序类应该尽可能独立于其他 Java 类来增加这些类重用的可能性，依赖注入DI
（或有时称为布线）有助于把这些类粘合在一起，同时保持他们`独立`。
- 构造函数注入
```
public class Person{
   private Speak speak;
   public Person(Speak speak) {
      this.speak= speak;
   }
}
```
依赖关系通过`类构造函数`被注入到 Person类中。
- setter方法注入
```
public class Person{
   private Speak speak;
   public void setSpeak(Speak speak){
	   this.speak=speak;
   }
}
```
依赖关系通过`类构造函数`被注入到 Person类中。控制流通过依赖注入（DI）已经“反转”，因为你已经有效地委托依赖关系到一些外部系统。
- XML配置文件注入
```
 <bean id="person" class="com.example.Person">
      <constructor-arg ref="speak"/>
</bean>
<!-- Definition for speak bean -->
<bean id="speak" class="com.example.Speak">
</bean>
```
##### Spring 基于设值函数的依赖注入
当容器调用一个无参的构造函数或一个无参的静态 factory 方法来初始化你的 bean 后，通过容器在你的 bean 上调用设值函数，基于设值函数的 DI 就完成了。
```
 <bean id="person" class="com.example.Person">
      <property name="speak" ref="speak">
</bean>
<!-- Definition for speak bean -->
<bean id="speak" class="com.example.Speak">
</bean>
```
- `Tip：`此方法和构造函数注入唯一的区别就是在基于构造函数注入中，我们使用的是标签中的元素，而在基于设值函数的注入中，我们使用的是标签。

- `tip：`如果你要把一个引用传递给一个对象，那么你需要使用 标签的 ref 属性，而如果你要直接传递一个值，那么你应该使用 value 属性。
#####`p-namespace`
```
<bean id="test" class="com.example.Person">
      <property name="name" value="swift"/>
      <property name="method" ref="speak"/>
</bean>
//可以通过如下方式简化表示
<bean id="test" class="com.example.Person"
      p:name="swift"
      p:method="speak"/>
</bean>
``` 
##### 注入内部bean
inner bean是在bean类中添加的内部类（java内部类）
```
<bean id="person" class="com.sunxiaohang.Person">
        <property name="speak">
            <bean id="speak" class="com.sunxiaohang.Speak"></bean>
        </property>
</bean>
```
tip:`<!--property需要提供set方法才能使用-->`
##### Spring注入集合
Spring提供了四种集合类
|Spring集合类|
|----|
|Map|
|Properties|
|Set|
|List|

以下是`Collections`类和`springbean.xml`的配置代码
```
package com.sunxiaohang;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

public class Collections {
    private Set collectionSet;
    private Map collectionMap;
    private Properties collectionProperties;
    private List collectionList;

    public Collections() {
    }

    public Set getCollectionSet() {
        System.out.println("Set Element:"+collectionSet);
        return collectionSet;
    }

    public void setCollectionSet(Set collectionSet) {
        this.collectionSet = collectionSet;
    }

    public Map getCollectionMap() {
        System.out.println("Map Element:"+collectionMap);
        return collectionMap;
    }

    public void setCollectionMap(Map collectionMap) {
        this.collectionMap = collectionMap;
    }

    public Properties getCollectionProperties() {
        System.out.println("Properties Element:"+collectionProperties);
        return collectionProperties;
    }

    public void setCollectionProperties(Properties collectionProperties) {
        this.collectionProperties = collectionProperties;
    }

    public List getCollectionList() {
        System.out.println("list Element:"+collectionList);
        return collectionList;
    }

    public void setCollectionList(List collectionList) {
        this.collectionList = collectionList;
    }
}

```
```
<bean id="collections" class="com.sunxiaohang.Collections">
    <property name="collectionList">
        <list>
            <value>张三</value>
            <value>李四</value>
            <value>王五</value>
            <value>马六</value>
        </list>
    </property>
    <property name="collectionSet">
        <set>
            <value>北京</value>
            <value>天津</value>
            <value>上海</value>
            <value>广州</value>
        </set>
    </property>
    <property name="collectionMap">
        <map>
            <entry key="1" value="one"/>
            <entry key="2" value="two"/>
            <entry key="3" value="three"/>
            <entry key="4" value="four"/>
        </map>
    </property>
    <property name="collectionProperties">
        <props>
            <prop key="1">姓名</prop>
            <prop key="2">性别</prop>
            <prop key="3">年龄</prop>
            <prop key="4">出生日期</prop>
        </props>
    </property>
</bean>
```
##### Spring的自动装配
`ByName`
这种模式由属性名称自动装配。Spring容器看作beans，在XML配置文件中的beans的auto-wire属性设置为byName，然后，他尝试将它的属性与配置文件中定义为相同名称的beans进行匹配和链接。




##### SpringMVC
自从Struct2爆出漏洞之后，其使用率每况愈下，还在坚持使用struct的多半是历史包袱比较重的老项目维护，SSH框架也变成了SSM，Spring开始如日终天一家独大，对于没有历史报复的JEE从业者，当然是选择更新更好的框架，总体来讲SpringMVC相对与Struct而言还是比较优秀的。
###### SpringMVC框架处理流程图
![Alt text](./1521771024324.png)