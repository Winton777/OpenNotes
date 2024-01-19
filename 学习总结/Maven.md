# Maven

[What is Maven?](https://maven.apache.org/what-is-maven.html)

Maven 是一款为 Java 项目管理构建、依赖管理的工具（软件），使用 Maven 可以自动化构建、测试、打包和发布项目，可以解决 `jar包的规模、jar包的来源、jar包的导入、jar包之间的依赖` 等问题；

## 安装

解压即可，需要本机安装Java环境

**bin**：含有Maven的运行脚本

boot：含有plexus-classworlds类加载器框架

**conf**：含有Maven的核心配置文件

lib：含有Maven运行时所需要的Java类库

LICENSE、NOTICE、README.txt：针对Maven版本，第三方软件等简要介绍

**配置Maven环境变量**

```
MAVEN_HOME - E:\Program\MAVEN_HOME\bin
path - %MAVEN_HOME%
检查mvn -v
```

**修改本地仓库、开发环境**

```
1.打开配置文件:maven目录/conf/settings.xml
2.添加仓库位置配置
<localRepository>E:/Program/maven-repository</localRepository>
（仓库位置改为自己指定地址）
3.更换阿里镜像,加快依赖下载
<mirror>  
  <id>nexus-aliyun</id>  
  <mirrorOf>central</mirrorOf>    
  <name>Nexus aliyun</name>  
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
4.修改IDEA开发Maven环境
	settings->Build,Exception,Deployment->BuildTools->Maven
	Maven home directory:选择安装目录(bin目录的上级目录)
	User settings file:勾选override修改,选择settings.xml目录
	Local repository:自动修改
```

## Maven工程

### 基本属性

Maven工程相对之前的项目，多出一组gavp属性。这四个属性主要为每个项目在maven仓库中做一个标识，方便后期项目之间相互引用依赖等！

推荐格式：

* **GroupID**：com.{公司/BU }.业务线.\[子业务线]，最多 4 级。如：com.taobao.tddl 或 com.alibaba.sourcing.multilang

* **ArtifactID**：产品线名-模块名。如：tc-client / uic-api / tair-tool / bookstore

* **Version**：主版本号.次版本号.修订号

* **Packaging**：项目打包类型

```xml
<!-- 模型版本 -->
<modelVersion>4.0.0</modelVersion>
<!-- 公司或者组织的唯一标志,并且配置时生成的路径也是由此生成 -->
<groupId>com.companyname.project-group</groupId>
<!-- 项目的唯一ID,一个groupId下面可能多个项目,是用artifactId来区分的 -->
<artifactId>project</artifactId>
<!-- 版本号 -->
<version>1.0.0</version>
<!--打包方式（默认：jar）
    jar指的是普通的java项目,打成jar包; war指的是web项目，打成war包;
    pom不会将项目打包,这个项目作为父工程，被其他工程聚合或者继承 -->
<packaging>jar/pom/war</packaging>
```

### 项目结构

```
|-- pom.xml                               # Maven 项目管理文件(用于管理：源代码、配置文件、开发者的信息和角色、问题追踪系统、组织信息、项目授权、项目的url、项目的依赖关系等)
|-- src
    |-- main                              # 项目主要代码
    |   |-- java                          # Java 源代码目录
    |   |   `-- com/example/myapp         # 开发者代码主目录
    |   |       |-- controller            # 存放 Controller 层代码的目录
    |   |       |-- service               # 存放 Service 层代码的目录
    |   |       |-- dao                   # 存放 DAO 层代码的目录
    |   |       `-- model                 # 存放数据模型的目录
    |   |-- resources                     # 资源目录，存放配置文件、静态资源等
    |   |   |-- log4j.properties          # 日志配置文件
    |   |   |-- spring-mybatis.xml        # Spring Mybatis 配置文件
    |   |   `-- static                    # 存放静态资源的目录
    |   |       |-- css                   # 存放 CSS 文件的目录
    |   |       |-- js                    # 存放 JavaScript 文件的目录
    |   |       `-- images                # 存放图片资源的目录
    |   `-- webapp                        # 存放 WEB 相关配置和资源
    |       |-- WEB-INF                   # 存放 WEB 应用配置文件
    |       |   |-- web.xml               # Web 应用的部署描述文件
    |       |   `-- classes               # 存放编译后的 class 文件
    |       `-- index.html                # Web 应用入口页面
    `-- test                              # 项目测试代码
        |-- java                          # 单元测试目录
        `-- resources                     # 测试资源目录
```

### 项目构建

| 命令        | 描述                                               |
| ----------- | -------------------------------------------------- |
| mvn compile | 编译项目，生成target文件                           |
| mvn package | 打包项目，生成jar或war文件                         |
| mvn clean   | 清理编译或打包后的项目结构                         |
| mvn install | 打包后上传到maven本地仓库                          |
| mvn deploy  | 只打包，上传到maven私服仓库                        |
| mvn site    | 生成站点                                           |
| mvn test    | 执行测试源码（测试类、方法命名格式均需要包含test） |

**构建生命周期**：执行周期后的命令，会自动触发周期前的命令，可以简化构建过程，如：项目打包 `mvn clean package` 即可。&#x20;

**主要两个构建生命周期**：

* 清理周期：主要是对项目编译生成文件进行清理 `clean` ；
* 默认周期：定义了真正构件时所需要执行的所有步骤，它是生命周期中最核心的部分 `compile -  test - package - install - deploy`

### 依赖管理

[Maven Repository（依赖查询网站）/ 也可用mavensearch插件搜索](https://mvnrepository.com/)

通过定义POM文件，Maven可以自动解析项目的依赖关系，并通过Maven仓库自动下载和管理依赖，从而避免了手动下载和管理依赖的繁琐工作和可能引发的版本冲突问题。

```xml
<!-- 通过编写依赖jar包的gav必要属性,引入第三方依赖;
     scope属性是可选的，可以指定依赖生效范围 -->
<dependencies>
    <!-- 引入具体的依赖包 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
        <!-- 依赖范围 -->
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <!--引用properties声明版本 -->
        <version>${junit.version}</version>
    </dependency>
</dependencies>

<!-- 声明版本 -->
<properties>
  <!-- 可以定义参数,统一管理依赖的版本,命名随便,内部制定版本号即可! -->
  <junit.version>4.12</junit.version>
  <!-- 也可以通过 maven规定的固定的key，配置maven的参数！如下配置编码格式！-->
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
```

#### 依赖范围

通过设置坐标的依赖范围(scope)，可以设置依赖的作用范围：编译环境、测试环境、运行环境

| 依赖范围     | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| **compile**  | **默认**，编译依赖范围。使用此依赖范围的 Maven 依赖，在上述三种 classpath 均会被引入。如，log4j 在编译、测试、运行过程都是必须的。 |
| **test**     | 测试依赖范围。使用此依赖范围的 Maven 依赖，只对测试 classpath 有效。如，Junit 依赖只有在测试阶段才需要。 |
| **provided** | 已提供依赖范围。使用此依赖范围的 Maven 依赖，只对编译和测试 classpath 有效。如，servlet-api 依赖对于编译、测试阶段而言是需要的，但在运行阶段，因外部容器已经提供，故不需要 Maven 重复引入该依赖。 |
| runtime      | 运行时依赖范围。只对测试 classpath、运行 classpath 有效。    |
| system       | 系统依赖范围，其效果与 provided 的一致。通过 systemPath 属性指定本地依赖，可移植性低，一般不推荐使用。 |
| import       | 导入依赖范围，该依赖范围只能与 dependencyManagement 元素配合使用，其功能是将目标 pom.xml 文件中 dependencyManagement 的配置导入合并到当前 pom.xml 的 dependencyManagement 中。 |

#### 依赖下载失败的解决方案

**解决方案：**

1. 检查网络连接和 Maven 仓库服务器状态。

2. 确保依赖项的版本号与项目对应的版本号匹配，并检查 POM 文件中的依赖项是否正确。

3. 清除本地 Maven 仓库缓存（依赖包文件夹 或 lastUpdated 文件），删除后刷新重新下载即可。

   ```bat
   #删除脚本
   cls 
   @ECHO OFF 
   SET CLEAR_PATH=D: 
   SET CLEAR_DIR=D:\maven-repository(本地仓库路径)
   color 0a 
   TITLE ClearLastUpdated For Windows 
   GOTO MENU 
   :MENU 
   CLS
   ECHO. 
   ECHO. * * * *  ClearLastUpdated For Windows  * * * * 
   ECHO. * * 
   ECHO. * 1 清理*.lastUpdated * 
   ECHO. * * 
   ECHO. * 2 查看*.lastUpdated * 
   ECHO. * * 
   ECHO. * 3 退 出 * 
   ECHO. * * 
   ECHO. * * * * * * * * * * * * * * * * * * * * * * * * 
   ECHO. 
   ECHO.请输入选择项目的序号： 
   set /p ID= 
   IF "%id%"=="1" GOTO cmd1 
   IF "%id%"=="2" GOTO cmd2 
   IF "%id%"=="3" EXIT 
   PAUSE 
   :cmd1 
   ECHO. 开始清理
   %CLEAR_PATH%
   cd %CLEAR_DIR%
   for /r %%i in (*.lastUpdated) do del %%i
   ECHO.OK 
   PAUSE 
   GOTO MENU 
   :cmd2 
   ECHO. 查看*.lastUpdated文件
   %CLEAR_PATH%
   cd %CLEAR_DIR%
   for /r %%i in (*.lastUpdated) do echo %%i
   ECHO.OK 
   PAUSE 
   GOTO MENU 
   ```


### 构建配置

默认情况下，构建不需要额外配置，也可以在pom.xml定制一些配置，来修改默认的构建。

例如：

1.  指定构建打包文件的名称，非默认名称
2.  制定构建打包时，指定包含文件格式和排除文件
3.  打包插件版本过低，配置更高版本插件

构建配置是在pom.xml / build标签中指定！

**指定打包命名**

```xml
<!-- 默认的打包名称：artifactid+verson.打包方式 -->
<build>
  <finalName>定义打包名称</finalName>
</build>  
```

**指定打包文件**

在java文件夹中添加java类，会自动打包编译到classes文件夹下，但若有非java文件则不会打包。

默认情况下，只有按照maven工程结构放置的文件才会默认被编译和打包，也可以使用resources标签，指定要打包资源的文件夹要把哪些静态资源打包到 classes根目录下！

```xml
<build>
    <!--设置要打包的资源位置-->
    <resources>
        <resource>
            <!--设置资源所在目录-->
            <directory>src/main/java</directory>
            <includes>
                <!--设置包含的资源类型-->
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```

**配置依赖插件**

可以在build/plugins/plugin标签引入插件。

常用的插件：修改jdk版本、tomcat插件、mybatis分页插件、mybatis逆向工程插件等等。

```xml
<build>
  <plugins>
      <!-- java编译插件，配jdk的编译版本 -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
          <encoding>UTF-8</encoding>
        </configuration>
      </plugin>
      <!-- tomcat插件 -->
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
         <version>2.2</version>
          <configuration>
          <port>8090</port>
          <path>/</path>
          <uriEncoding>UTF-8</uriEncoding>
          <server>tomcat7</server>
        </configuration>
      </plugin>
    </plugins>
</build>
```

### 依赖传递和依赖冲突

若Maven项目A依赖Maven项目B，Maven项目B依赖Maven项目C，那么执行项目A时，会自动将B和C都自动下载。

**传递的原则**

依赖能否传递，取决于依赖范围以及配置

- compile范围，可以传递；其他范围，不能传递

- 若配置了以下 `<optional>true</optional>` 标签，则不能传递

  ```xml
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.2.15</version>
      <optional>true</optional>
  </dependency>
  ```

-   冲突的依赖可能不会传递（根据解决冲突的规则）

**解决依赖冲突**

当直接引用或者间接引用了相同的jar包，就是依赖冲突，maven有自动解决依赖冲突的规则，也提供了手动解决冲突的方式（据说不常用）。

1. 自动选择原则

   - 短路优先原则（第一原则）

     A—>B—>C—>D—>E—>X(version 0.0.1)

     A—>F—>X(version 0.0.2)

     则A依赖于X(version 0.0.2)。

   - 依赖路径长度相同情况下，则“先声明优先”（第二原则）

     A—>E—>X(version 0.0.1)

     A—>F—>X(version 0.0.2)

     路径相同时，在\<depencies/>标签中先声明的，会优先选择。

2. 手动排除

   ```xml
   <dependency>
       <groupId>com.atguigu.maven</groupId>
       <artifactId>pro01-maven-java</artifactId>
       <version>1.0-SNAPSHOT</version>
       <scope>compile</scope>
       <!-- 使用excludes标签配置依赖的排除  -->
       <exclusions>
           <!-- 在exclude标签中配置一个具体的排除 -->
           <exclusion>
               <!-- 指定要排除的依赖的坐标（不需要写version） -->
               <groupId>commons-logging</groupId>
               <artifactId>commons-logging</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

## Maven工程继承和聚合关系

### 继承关系

Maven 继承是指在 Maven 的项目中，让一个项目从另一个项目中继承配置信息的机制。继承可以让我们在多个项目中共享同一配置信息，简化项目的管理和维护工作。

**语法**

* 父工程 (父工程可以只保留pom.xml文件，用来管理依赖，无需src等文件)

```xml
<groupId>com.atguigu.maven</groupId>
<artifactId>pro03-maven-parent</artifactId>
<version>1.0-SNAPSHOT</version>
<!-- 当前工程作为父工程，它要去管理子工程，所以打包方式必须是 pom -->
<packaging>pom</packaging>
```

* 子工程

```xml
<!-- 使用parent标签指定当前工程的父工程 -->
<parent>
    <!-- 父工程的坐标 -->
    <groupId>com.atguigu.maven</groupId>
    <artifactId>pro03-maven-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<!-- 如果子工程坐标中的groupId和version与父工程一致，则可以省略 -->
<!-- <groupId>com.atguigu.maven</groupId> -->
<artifactId>pro04-maven-module</artifactId>
<!-- <version>1.0-SNAPSHOT</version> -->
```

**父工程依赖统一管理**

* 父工程使用dependencyManagement标签配置对依赖的管理

```xml
<!-- 使用dependencyManagement标签配置对依赖的管理 -->
<!-- 被管理的依赖并没有真正被引入到工程,子工程可以按需决定使用父工程的部分依赖 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>6.0.10</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>6.0.10</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.0.10</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>6.0.10</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>6.0.10</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

- 子工程引用版本

  ```xml
  <!-- 子工程引用父工程中的依赖信息时,可以把版本号去掉  -->
  <!-- 把版本号去掉就表示子工程中这个依赖的版本由父工程的dependencyManagement来决定 -->
  <dependencies>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-core</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-beans</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-expression</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-aop</artifactId>
      </dependency>
  </dependencies>
  ```

### 聚合关系

Maven 聚合是指将多个项目组织到一个父级项目中，以便一起构建和管理的机制。聚合可以更好地管理一组相关的子项目，同时简化它们的构建和部署过程。

**作用**

1.  管理多个子项目：通过聚合，可以将多个子项目组织在一起，方便管理和维护。
2.  构建和发布一组相关的项目：通过聚合，可以在一个命令中构建和发布多个相关的项目，简化了部署和维护工作。
3.  优化构建顺序：通过聚合，可以对多个项目进行顺序控制，避免出现构建依赖混乱导致构建失败的情况。
4.  统一管理依赖项：通过聚合，可以在父项目中管理公共依赖项和插件，避免重复定义。

* 通过触发父工程构建命令，会引发所有子模块构建。

**语法**

```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <packaging>pom</packaging>
    <version>1.0.0</version>
    <modules>
        <!-- moudle为子项目的路径,一般继承后会自动生成 -->
        <module>child-project1</module>
        <module>child-project2</module>
    </modules>
</project>
```

## 问题整理

1. **IDEA 中 Maven 控制台中文乱码**

```
打开File->Settings->Build,Execution,Deployment->Build Tools->Maven->Runner在VM Options中，添加-Dfile.encoding=GBK
```