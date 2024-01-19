[TOC]

# Spring

[Spring | Projects](https://spring.io/projects/)

Spring是轻量级的开源的JavaEE框架，可以解决企业应用开发的复杂性
Spring有两个核心部分：IOC（控制反转）和Aop（面向切面）

**Spring Framework**：Spring 基础框架，有以下特性，先做理解，学完回头再来整理

* 非侵入式：使用 Spring Framework 开发应用程序时，Spring 对应用程序本身的结构影响非常小。对领域模型可以做到零污染；对功能性组件也只需要使用几个简单的注解进行标记，完全不会破坏原有结构，反而能将组件结构进一步简化。这就使得基于 Spring Framework 开发应用程序时结构清晰、简洁优雅。
* 控制反转：IOC—Inversion of Control，翻转资源获取方向。把自己创建资源、向环境索取资源变成环境将资源准备好，我们享受资源注入。
* 向切面编程：AOP—Aspect Oriented Programming，在不修改源代码的基础上增强代码功能。
* 容器：Spring IOC 是一个容器，因为它包含并且管理组件对象的生命周期。组件享受到了容器化的管理，替程序员屏蔽了组件创建过程中的大量细节，极大的降低了使用门槛，大幅度提高了开发效率。
* 组件化：Spring 实现了使用简单的组件配置组合成一个复杂的应用。在 Spring 中可以使用 XML和 Java 注解组合这些对象。这使得我们可以基于一个个功能明确、边界清晰的组件有条不紊的搭建超大型复杂应用系统。
* 声明式：很多以前需要编写代码才能实现的功能，现在只需要声明需求即可由框架代为实现。
* 一站式：在 IOC 和 AOP 的基础上可以整合各种企业应用的开源框架和优秀的第三方类库。而且Spring 旗下的项目已经覆盖了广泛领域，很多方面的功能性需求可以在 Spring Framework 的基础上全部使用 Spring 来实现。

# IOC

**Inversion of Control：控制反转**

1. 控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理
2. 使用IOC的目的：为了降低耦合度

**DI：Dependency Injection 依赖注入**

DI 是 IOC 的另一种表述方式：即组件以一些预先定义好的方式（例如：setter 方法）接受来自于容器的资源注入。相对于IOC而言，这种表述更直接。

即 IOC 是一种反转控制的思想， 而 DI 是对 IOC 的一种具体实现。

**IOC底层原理：**xml解析、工厂模式、反射

**IOC过程：**1.xml配置文件，配置创建的对象；2.创建工厂类，解析xml，通过反射创建对象

**引入依赖：**

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.1</version>
</dependency>
```

## 快速开始

1. 创建HelloWord类

```java
public class HelloWorld {
    public void sayHello(){
        System.out.println("Hello World~");
    }
}
```

2. 创建Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="helloWorld" class="com.winton.pojo.HelloWorld"/>
</beans>
```

3. 测试一下

```java
public void testIoc() {
    ApplicationContext context = new ClassPathXmlApplicationContext("spring-ioc.xml");
    HelloWorld helloWorld = context.getBean("helloWorld",HelloWorld.class);
    helloWorld.sayHello();
}
```

## IOC接口

1. IOC思想基于IOC容器完成，IOC容器底层就是对象工厂

2. Spring提供IOC容器实现两种方式：（两个接口）

   （1）BeanFactory：IOC容器基本实现，是Spring内部的使用接口，不提供开发人员进行使用；

   * 加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象

   （2）ApplicationContext：BeanFactory接口的子接口，提供更多更强大的功能，一般由开发人员进行使用；

   * 加载配置文件时候就会把在配置文件对象进行创建

3、ApplicationContext接口有实现类

```java
//ClassPathXmlApplicationContext 通过读取类路径下的XML格式的配置文件创建IOC容器对象。没有前缀"app.spring.xml"时，默认路径为项目的classpath下相对路径
//FileSystemXmlApplicationContext 通过文件系统路径读取XML格式的配置文件创建IOC容器对象。默认路径为项目工作路径，即项目的根目录，如"src/main/resources/spring-ioc.xml"
// 1.使用前缀"classpath:app.spring.xml"表示的是项目的classpath下相对路径
// 2.使用前缀"file:D:/app.spring.xml"表示的是文件的绝对路径
// 3.可以同时加载多个文件
   String[] xmlCfg = new String[] { "classpath:base.spring.xml","app.spring.xml"};
   ApplicationContext appCt = new ClassPathXmlApplicationContext(xmlCfg);
// 4.使用通配符加载所有符合要求的文件
   ApplicationContext appCt = new ClassPathXmlApplicationContext("*.spring.xml");
```

## IOC的bean管理（基于xml方式）

bean管理指的是两个操作：1.Spirng创建对象；2.Spirng注入属性

bean管理操作有两种方式：1.基于xml配置文件方式实现；2.基于注解方式实现

### 获取bean的三种方式

1. 根据bean的id获取
2. 根据bean的类型获取**（后续常用）**
   **注意：**根据类型获取bean时，要求IOC容器中有且只有一个类型匹配的bean；若没有任何一个类型匹配的bean，此时抛出异常：NoSuchBeanDefinitionException；若有多个类型匹配的bean，此时抛出异常：NoUniqueBeanDefinitionException
3. 根据bean的id和类型获取

根据类型来获取bean时，在满足bean唯一性的前提下，其实只是看：『对象 instanceof 指定的类型』的返回结果，只要返回的是true就可以认定为和类型匹配，能够获取到。即类实现了接口，那么根据接口类型也可以获取bean。

```java
//获取IOC容器
ApplicationContext ioc = new ClassPathXmlApplicationContext("spring-ioc.xml");
//获取bean
Object helloWorld1 = context.getBean("helloWorld");
HelloWorld helloWorld2 = context.getBean(HelloWorld.class);
HelloWorld helloWorld3 = context.getBean("helloWorld",HelloWorld.class);
```

### 基于xml方式：创建对象

在spring配置文件中，使用bean标签，即将该对象交给Spring的IOC容器管理，通过bean标签配置IOC容器所管理的bean，就可以实现对象创建

```xml
<!--bean标签 属性：id：设置bean的唯一标识 class：设置bean所对应类型的全类名-->
<bean id="helloWorld" class="com.winton.pojo.HelloWorld"/>
```

**Spring 底层默认通过 反射 调用类的无参构造器来创建对象。如果在需要无参构造器时，没有无参构造器，则会抛出BeanCreationException异常。大多情况下，反射都是通过无参构造器创建对象。**

### 基于xml方式：依赖注入

#### 第一种：使用set方法进行注入

1. 创建类，定义属性和对应的set方法

2. 在spring配置文件配置

```xml
<!-- property标签：通过组件类的setXxx()方法给对象设置属性 -->
<!-- 属性：name：指定属性名（这个属性名是getXxx()、setXxx()方法定义的，和成员变量无关） value：指定属性值-->
<bean id="user" class = "com.wyt.spring.User">
    <property name="uid" value="1"/>
    <property name="userName" value="张三"/>
</bean>
```

#### 第二种：使用有参数构造进行注入

1. 创建类，定义属性，创建属性对应有参数构造方法

2. 在spring配置文件配置

```xml
<!-- constructor-arg标签：通过构造器给对象设置属性 -->
<!-- value：指定属性值 name属性：指定参数名 index属性：指定参数所在位置的索引（从0开始）-->
<!-- 若不设置name和index属性，会自动按照构造器的参数个数去匹配构造器，当有参数个数相同的构造器时不能确保准确性 -->
<bean id="userTwo" class="com.winton.pojo.User">
    <constructor-arg name="uid" value="2"/>
    <constructor-arg name="userName" value="李四"/>
</bean>
```

#### 其他类型属性的注入

##### null值

```xml
<bean id="userThree" class="com.winton.pojo.User">
    <property name="uid" value="3"/>
    <property name="userName">
        <null/>
    </property>
</bean>
```

##### 属性值包含特殊符号

```xml
<!--属性值包含特殊符号
1.采用<![CDATA[ ]]>特殊标签，将包含特殊字符的字符串封装起来。
2.使用XML转义序列表示这些特殊的字符，这5个特殊字符所对应XML转义序列为：
    &  &amp;
    <  &lt;
    >  &gt;
    "  &quot;
    '  &apos;
--> 
<bean id="userThree" class="com.winton.pojo.User">
    <property name="uid" value="3"/>
    <property name="address">
        <value><![CDATA[<<南京>>]]>&quot;&amp;&quot;</value>
    </property>
</bean>
```

##### 对象：外部bean/内部bean/级联赋值

* **外部bean：直接在beans标签内部直接定义的bean对象，通过ref属性指向即可，外部bean可以被多个bean对象引用**

* **内部bean：在某个bean标签的内部定义的bean对象，内部bean只能被某个对象的某个属性引用**
* **级联属性：在实体类中提供该属性的get方法，然后在配置文件中通过 . 操作符来完成赋值**

```xml
<!--外部bean-->
<bean id="userFour" class="com.winton.pojo.User">
    <property name="uid" value="4"/>
    <property name="userName" value="小熊"/>
    <property name="company" ref="companyOuter"/>
</bean>
<bean name="companyOuter" class="com.winton.pojo.Company">
    <!--User对象中注入Company对象 name属性：类里面属性名称 ref属性：创建Company对象bean标签id值 -->
    <property name="companyName" value="Huawei"/>
</bean>
<!--内部bean-->
<bean id="userFive" class="com.winton.pojo.User">
    <property name="uid" value="5"/>
    <property name="userName" value="小白"/>
    <!--设置对象类型属性-->
    <property name="company">
        <bean id="companyInner" class="com.winton.pojo.Company">
            <property name="companyName" value="Apple"/>
        </bean>
    </property>
</bean>
<!--级联赋值-->
<bean id="userSix" class="com.winton.pojo.User">
    <property name="uid" value="6"/>
    <property name="userName" value="老六"/>
    <!--一定先引用某个bean为属性赋值或实例化，才可以使用级联方式更新属性-->
    <!--需要User类中提供getCompany()方法,才可以这么使用-->
    <property name="company" ref="companyOuter"/>
    <property name="company.companyName" value="AMD"/>
</bean>
```

##### 集合属性

注入数组类型、List集合、Map集合、Set集合

1. 创建类，定义数组、List、Map、Set等类型属性，生成对应set方法

2. 在spring配置文件进行配置

```xml
<bean id="userSeven" class="com.winton.pojo.User">
    <property name="uid" value="7"/>
    <property name="userName" value="凌凌漆"/>
    <property name="FavoritesArr">
        <array>
            <value>Arr1</value>
            <value>Arr2</value>
        </array>
    </property>
    <property name="FavoritesList">
        <list>
            <value>List1</value>
            <value>List2</value>
        </list>
    </property>
    <property name="FavoritesMap">
        <map>
            <entry key="Map1" value="Test1"/>
            <entry key="Map2" value="Test2"/>
        </map>
    </property>
    <property name="FavoritesSet">
        <set>
            <value>Set1</value>
            <value>Set2</value>
        </set>
    </property>
</bean>
```

###### 集合里包含对象类型值

```xml
<!--在对应集合类型中，使用ref引用创建的bean对象-->
<bean id="userEight" class="com.winton.pojo.User">
    <property name="uid" value="8"/>
    <property name="userName" value="粑粑"/>
    <property name="HisCompany">
        <list>
            <ref bean="companyOuter"/>
            <ref bean="companyOuterTwo"/>
        </list>
    </property>
</bean>
<bean name="companyOuter" class="com.winton.pojo.Company">
    <property name="companyName" value="Huawei"/>
</bean>
<bean name="companyOuterTwo" class="com.winton.pojo.Company">
    <property name="companyName" value="Intel"/>
</bean>
```

###### 把集合注入部分提取出来

1. 在spring配置文件中引入命名空间 util

```xml
<!--新增此行后面部分-->
<beans xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="后面为新增	http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">
```

2. 使用util:list标签完成list集合注入提取，也有util:map和util:set

```xml
<bean id="userNine" class="com.winton.pojo.User">
    <property name="uid" value="9"/>
    <property name="userName" value="玖"/>
    <property name="HisCompany" ref="utilList"/>
</bean>
<util:list id="utilList">
    <ref bean="companyOuter"/>
    <ref bean="companyOuterTwo"/>
</util:list>
```

##### 了解：p命名空间注入

使用p命名空间注入，可以简化基于xml配置方式

1. 添加p命名空间在配置文件中

```xml
xmlns:p="http://www.springframework.org/schema/p"
```

2. 在spring配置文件配置，在bean标签里面通过 p:属性 进行操作

```xml
<bean id="userTen" class = "com.winton.pojo.User" p:uid="10" p:userName="拾"/>
```

### bean的作用域

在Spring中可以通过配置bean标签的scope属性来指定bean的作用域范围，各取值含义参加下表：

| 取值              | 含义                                  | 创建对象的时机                          |
| ----------------- | ------------------------------------- | --------------------------------------- |
| singleton（默认） | 这个bean的对象在IOC容器中始终为单实例 | IOC容器初始化时（加载spring配置文件时） |
| prototype         | 这个bean在IOC容器中有多个实例         | 获取bean时                              |

如果是在WebApplicationContext环境下还会有另外两个作用域（但不常用）：request（在一个请求范围内有效）、session（在一个会话范围内有效）

```xml
<bean id="user" class="com.winton.pojo.User" scope="prototype"/>
```

### bean的生命周期

**生命周期：**从对象创建到对象销毁的过程

**具体的生命周期过程**

1. bean对象创建（调用无参构造器）
2. 为bean对象设置属性（调用set方法）
3. bean对象初始化之前操作（由bean的后置处理器负责）
4. bean对象初始化（需在配置bean时指定初始化方法）
5. bean对象初始化之后操作（由bean的后置处理器负责）
6. bean对象的使用
7. bean对象销毁（需在配置bean时指定销毁方法，在IOC容器关闭调用）

**后置处理器**

bean的后置处理器会在生命周期的初始化前后添加额外的操作，需要实现BeanPostProcessor接口，且配置到IOC容器中。

需要注意的是，bean后置处理器不是单独针对某一个bean生效，而是针对IOC容器中所有bean都会执行。

**测试**

```java
public class Order {
    private String oname;
    //无参数构造
    public Order() {
        System.out.println("1.实例化 执行无参数构造创建bean实例");
    }
    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("2.依赖注入 调用set方法设置属性值");
    }
    //    创建执行的初始化的方法
    public void initMethod() {
        System.out.println("3.执行初始化的方法");
    }
    // 创建执行的销毁的方法
    public void destroyMethod() {
        System.out.println("5.执行销毁的方法");
    }
}
```

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
public class BeanProcess implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("在初始化之前执行的方法" + beanName + " = " + bean);
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("在初始化之后执行的方法" + beanName + " = " + bean);
        return bean;
    }
}
```

```xml
<!-- 使用init-method属性指定初始化方法 -->
<!-- 使用destroy-method属性指定销毁方法 -->
<bean id="order" class="com.winton.pojo.Order" init-method="initMethod" destroy-method="destroyMethod">
    <property name="oname" value="1001"/>
</bean>
<!-- bean的后置处理器要放入IOC容器才能生效 -->
<bean id="beanProcessor" class="com.winton.process.BeanProcess"/>
```

```java
@Test
public void testLifecycle() {
    ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("spring-scopeAndBean.xml");
    Order order = ioc.getBean(Order.class);
    System.out.println("4.使用bean " + order);
    ioc.close();
}
```

### FactoryBean（工厂bean）

**普通bean：**在配置文件中定义bean类型就是返回类型

**FactoryBean：**在配置文件定义bean类型可以和返回类型不一样

FactoryBean是Spring提供的一种整合第三方框架的常用机制。和普通的bean不同，配置一个FactoryBean类型的bean，在获取bean的时候得到的并不是class属性中配置的这个类的对象，而是getObject()方法的返回值。

通过这种机制，Spring可以帮我们把复杂组件创建的详细过程和繁琐细节都屏蔽起来，只把展示最简洁的使用界面给我们。将来我们整合Mybatis时，Spring就是通过FactoryBean机制来帮我们创建SqlSessionFactory对象的。

**使用方法**

1. 创建类，让这个类作为工厂bean，实现接口 FactoryBean，实现接口里面的方法，在实现的方法中定义返回的bean类型

```java
public class OrderFactoryBean implements FactoryBean<Order> {
    @Override
    public Order getObject() throws Exception {
        return new Order();
    }
    @Override
    public Class<?> getObjectType() {
        return Order.class;
    }
}
```

2. 配置bean

```xml
<bean id="user" class="com.atguigu.bean.UserFactoryBean"></bean>
```

3. 测试

```java
@Test
public void testFactoryBean() {
    ApplicationContext ioc = new ClassPathXmlApplicationContext("spring-scopeAndBean.xml");
    Order order = ioc.getBean(Order.class);
    System.out.println(order);
}
```

### xml自动装配

**自动装配：**根据指定装配规则（属性名称或者属性类型），Spring自动将匹配的属性值进行注入

使用bean标签的autowire属性设置自动装配效果

**byType：**根据类型匹配IOC容器中的某个兼容类型的bean，为属性自动赋值。若在IOC中，没有任何一个兼容类型的bean能够为属性赋值，则该属性不装配，为默认值null；若有多个兼容类型的bean能够为属性赋值，则抛出NoUniqueBeanDefinitionException异常。

**byName：**将自动装配的属性的属性名，作为bean的id在IOC容器中匹配相对应的bean进行赋值，区分大小写

```xml
<bean class="com.winton.pojo.User" autowire="byType"/>
<bean id="company" class="com.winton.pojo.Company">
    <property name="companyName" value="Huawei"/>
</bean>
```

## 应用：Spring管理数据源和引入外部属性文件

1.添加依赖

```xml
<!-- MySQL驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.3</version>
</dependency>
<!-- 数据源 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.31</version>
</dependency>
```

2. 添加外部配置文件 如：jdbc.properties

```properties
#Mysql
jdbc.mysqlDriver=com.mysql.jdbc.Driver
jdbc.mysqlUrl=jdbc:mysql://114.115.167.249:3306/winton?characterEncoding=utf8
jdbc.mysqlUsername=root
jdbc.mysqlPassword=123456
```

3. bean文件中引入配置文件

```xml
<!-- 配置文件中需要先引入context命名空间 -->
<beans xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="后面为新增	http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
<!-- 引入外部属性文件 -->
<context:property-placeholder location="classpath:jdbc.properties"/>
```

4. 配置bean

```xml
<bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${jdbc.mysqlDriver}"/>
    <property name="url" value="${jdbc.mysqlUrl}"/>
    <property name="username" value="${jdbc.mysqlUsername}"/>
    <property name="password" value="${jdbc.mysqlPassword}"/>
</bean>
```

5. 测试一下

```java
public void testDataSource() throws SQLException {
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-datasource.xml");
    DataSource dataSource = ac.getBean(DataSource.class);
    Connection connection = dataSource.getConnection();
    System.out.println(connection);
}
```

## IOC的bean管理（基于注解方式）

**什么是注解：**注解是代码特殊标记，格式：@注解名称(属性名称=属性值, 属性名称=属性值..)
**怎么使用注解：**注解作用在类上面，方法上面，属性上面
**使用注解目的：**简化xml配置

**标识组件的常用注解：**
@Component：将类标识为普通组件
@Controller：将类标识为控制层组件
@Service：将类标识为业务层组件
@Repository：将类标识为持久层组件

对于Spring使用IOC容器管理这些组件来说，这四种注解没有任何区别，都可以用来创建bean实例，实际效果完全一样。所以@Controller、@Service、@Repository这三个注解只是给开发人员看的，让我们能够便于分辨组件的作用。

注意：虽然它们本质上一样，但是为了代码的可读性，为了程序结构严谨，严格不能随便标记。

### 添加注解

**例：**

```java
//控制层组件
@Controller
public class UserController {}

//接口UserService
public interface UserService {}

//业务层组件UserServiceImpl
@Service
public class UserServiceImpl implements UserService {}

//接口UserDao
public interface UserDao {}

//持久层组件UserDaoImpl
@Repository
public class UserDaoImpl implements UserDao {}
```

### 扫描组件

添加上注解后，需要指定Spring去扫描这些组件，以识别注解并创建bean，如果扫描多个包，多个包使用逗号隔开。

1. 最基本的扫描方式

```xml
<context:component-scan base-package="com.winton"/>
```

2. 指定要排除的组件

```xml
<context:component-scan base-package="com.winton">
    <!--context:exclude-filter标签：指定排除哪些内容 -->
    <!--type：设置排除或包含的依据
            type="annotation"：根据注解排除，expression中设置要排除的注解的全类名
            type="assignable"：根据类型排除，expression中设置要排除的类型的全类名-->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
    <!--<context:exclude-filter type="assignable" expression="com.winton.controller.UserController"/>-->
</context:component-scan>
```

3. 仅扫描指定组件

```xml
<context:component-scan base-package="com.winton" use-default-filters="false">
    <!-- context:include-filter标签：指定扫描哪些内容 -->
    <!-- use-default-filters属性：取值false表示关闭默认扫描规则 -->
    <!-- 使用include-filter标签指定扫描时必须设置use-default-filters="false"，因为默认规则即扫描指定包下所有类 -->
    <!--type：设置排除或包含的依据
        type="annotation"，根据注解扫描，expression中设置要扫描的注解的全类名
        type="assignable"，根据类型扫描，expression中设置要扫描的类型的全类名-->
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    <!--<context:include-filter type="assignable" expression="com.winton.controller.UserController"/>-->
</context:component-scan>
```

### 组件所对应的bean的id

在用XML方式管理bean的时候，每个bean都有一个唯一标识，便于在其他地方引用。现在使用注解后，每个组件仍然应该有一个唯一标识。

默认情况：类名首字母小写就是bean的id。如：UserController类对应的bean的id就是userController。

也可通过注解的value值自定义设置的bean的id
```java
@Controller("controllerOne")
public class UserController {}
```

### 注入属性

**@Qualifier：根据名称进行注入**

```java
@Qualifier("testUserService")
private UserService userService;
```

**@Resource：可以根据类型注入，也可以根据名称注入**

```java
//@Resource //根据类型进行注入
@Resource(name = "userDaoImpl") //根据名称进行注入
private UserDao userDao;
```

**@Value：注入普通类型属性**

```java
@Value(value = "abc")
private String name;
```

### 自动装配

在成员变量上直接标记@Autowired注解即可完成自动装配，不需要提供setXxx()方法。（常用）
@Autowired注解也可以标记在构造器和set方法上

```java
@Autowired
private UserService userService;
```

**@Autowired的工作流程**

首先根据所需要的组件类型到IOC容器中查找

* 能够找到唯一的bean：直接执行装配
* 如果完全找不到匹配这个类型的bean：装配失败
* 和所需类型匹配的bean不止一个
  * 没有@Qualifier注解时：根据@Autowired标记位置成员变量的变量名作为bean的id进行匹配；使用@Qualifier注解时：根据@Qualifier注解中指定的名称作为bean的id进行匹配
    * 能够找到：执行装配
    * 找不到：装配失败

@Autowired中有属性required，默认值为true，因此在自动装配无法找到相应的bean时，会装配失败

可以将属性required的值设置为false（@Autowired(required = false)），则表示能装就装，装不上就不装，此时自动装配的属性为默认值。但是实际开发时，基本上所有需要装配组件的地方都是必须装配的，用不上这个属性。

### 完全注解开发

创建配置类，替代xml配置文件

```java
@Configuration
@ComponentScan(basePackages = {"com.winton"})
public class SpringConfig {}

//调用时
ApplicationContext ioc = new AnnotationConfigApplicationContext(SpringConfig.class);
```
