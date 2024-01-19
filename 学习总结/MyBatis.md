# MyBatis

[MyBatis SQL mapper framework for Java (github.com)](https://github.com/mybatis/mybatis-3)

- MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架**持久层框架**（关于持久层的理解：执行持久化过程，如将内存数据存储到硬盘上）
- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集的过程
- MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 实体类 【Plain Old Java Objects,普通的 Java对象】映射成数据库中的记录。
- MyBatis 是一个 半自动的ORM（Object Relation Mapping）框架

# 开始使用

**测试环境为**：JDK8，Mysql 8.0.35， Mybatis 3.5.7

## 创建MyBatis的核心配置文件

习惯上命名为 `mybatis-config.xml` ，核心配置文件主要用于配置连接数据库的环境以及MyBatis的全局配置信息

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 标签有固定的顺序要求 The content of element type "configuration" must match "
    (properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,
    reflectorFactory?,plugins?,environments?,databaseIdProvider?,mappers?)".-->

    <!--引入properties文件，此时就可以${属性名}的方式访问属性值-->
    <properties resource="jdbc.properties"/>

    <settings>
        <!--将表中字段的下划线自动转换为驼峰-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!--开启延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>


    <typeAliases>
        <!-- typeAlias：设置某个具体的类型的别名 属性：type：需要设置别名的类型的全类名
        alias：设置此类型的别名，若不设置此属性，该类型拥有默认的别名，即类名且不区分大小写;若设置此属性，此时该类型的别名只能使用alias所设置的值 -->
        <typeAlias type="com.winton.pojo.User"/>
        <!-- 也可以以包为单位，设置改包下所有的类型都拥有默认的别名，即类名且不区分大小写-->
        <!-- <package name="com.winton.pojo"/> -->
    </typeAliases>

    <!-- 配置数据库连接环境 -->
    <!-- environments：设置多个连接数据库的环境 属性：default：设置默认使用的环境的id -->
    <environments default="mysql">
        <!-- environment：设置具体的连接数据库的环境信息 属性：id：设置环境的唯一标识 -->
        <environment id="mysql">
            <!-- transactionManager：设置事务管理方式 属性：type：设置事务管理方式，
            type="JDBC"：设置当前环境的事务管理都必须手动处理
            type="MANAGED"：设置事务被管理，例如spring中的AOP    -->
            <transactionManager type="JDBC"/>
            <!-- dataSource：设置数据源 属性：type：设置数据源的类型
            type="POOLED"：使用数据库连接池，即会将创建的连接进行缓存，下次使用可以从缓存中直接获取，不需要重新创建
            type="UNPOOLED"：不使用数据库连接池，即每次使用连接都需要重新创建
            type="JNDI"：调用上下文中的数据源 -->
            <dataSource type="POOLED">
                <!--设置驱动类的全类名-->
                <property name="driver" value="${jdbc.driver}"/>
                <!--设置连接数据库的连接地址-->
                <property name="url" value="${jdbc.url}"/>
                <!--设置连接数据库的用户名-->
                <property name="username" value="${jdbc.username}"/>
                <!--设置连接数据库的密码-->
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <!--引入映射文件-->
    <mappers>
        <!-- 可以单独引入 -->
        <!-- <mapper resource="UserMapper.xml"/> -->
        <!-- 也可以以包为单位，将包下所有的映射文件引入核心配置文件
        注意：此方式必须保证mapper接口和mapper映射文件包名相同 -->
        <package name="com.winton.mapper"/>
    </mappers>
</configuration>
```

## 创建实体类

相关概念：ORM（Object Relationship Mapping）对象关系映射。
对象：Java的实体类对象
关系：关系型数据库
映射：二者之间的对应关系

| Java概念 | 数据库概念 |
| :------: | :--------: |
|    类    |     表     |
|   属性   |  字段/列   |
|   对象   |  记录/行   |

```java
package com.winton.pojo;
public class User {
    private Integer id;
    private String name;
    private Integer age;
    private String sex;
    private String email;
	//还需要
    //构造器,有参,无参
    //Getter/Setter
    //toString()
}
```

## 创建Mapper接口

MyBatis中的Mapper接口相当于以前的Dao。但是区别在于，Mapper仅仅是接口，我们不需要提供实现类。

```java
package com.winton.mapper;
import com.winton.pojo.User;
import java.util.List;
public interface UserMapper {
    //添加用户
    int insertUser();
    //删除用户
    void deleteUser();
    //修改用户信息
    void updateUser();
    //查询用户
    User getUserById();
    //查询所有用户
    List<User> getAllUser();
}
```

## 创建MyBatis的映射文件

1. 映射文件的命名规则：表所对应的实体类的类名+Mapper.xml；因此一个映射文件对应一个实体类，对应一张表的操作。
   MyBatis映射文件用于编写SQL，访问以及操作表中的数据
2. MyBatis中可以面向接口操作数据，要保证两个一致：
   mapper接口的全类名和映射文件的命名空间（namespace）保持一致
   mapper接口中方法名和映射文件中编写SQL的标签的id属性保持一致

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.winton.mapper.UserMapper">
    <!-- int insertUser(); -->
    <insert id="insertUser">
        insert into t_user_basic values (null,'admin','admin','30','男','admin@dcits.com')
    </insert>
    <!-- void deleteUser(); -->
    <delete id="deleteUser">
        delete from t_user_basic where id = '3' and username = 'admin'
    </delete>
    <!-- void updateUser(); -->
    <update id="updateUser">
        update t_user_basic set username = '欧文' where id = '4'
    </update>
    <!-- User getUserById(); -->
    <select id="getUserById" resultType="com.winton.pojo.User">
        select * from t_user_basic where id = '1'
    </select>
    <!-- List<User> getAllUser(); -->
    <select id="getAllUser" resultType="com.winton.pojo.User">
        select * from t_user_basic
    </select>
</mapper>
```

## 测试代码

```java
@Test
public void testInsert() throws IOException {
    SqlSession sqlSession = DatabaseUtils.getSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    int result = mapper.insertUser();
    //sqlSession.commit();//不自动提交事务时需要手动提交
    System.out.println("result:" + result);
}
@Test
public void testSelect() throws IOException {
    SqlSession sqlSession = DatabaseUtils.getSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User user = mapper.getUserById();
    System.out.println(user);

    List<User> list = mapper.getAllUser();
    System.out.println(list);
}

/**
 * 获取数据库会话，并自动提交事务
 */
public static SqlSession getSession() {
    SqlSession sqlSession = null;
    try {
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        sqlSession = sqlSessionFactory.openSession(true);//true 表示自动提交事务
    } catch (IOException e) {
        e.printStackTrace();
    }
    return sqlSession;
}
```

# 模板

Idea 中 `Settings->Editor->File and Code Templates` 可以添加模板，方便快速创建文件。

# Mapper的写法

## 传参方式

MyBatis获取参数值的两种方式：${}和#{}

${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值时，需要手动加单引号；

#{}使用占位符赋值的方式拼接sql，此时为字符串类型或日期类型的字段进行赋值时，会自动添加单引号;

```java
User getUserById(String id);
User checkLogin(String str1, String str2);
User checkLoginByMap(HashMap<String, Object> param);
int insertUserPlus(User user);
User checkLoginByParam(@Param("username") String str1, @Param("password") String str2);
```

1. 若mapper接口中的方法参数为单个的字面量类型，如上第一种，此时可以使用${}和#{}以任意的名称获取参数的值；
2. 若mapper接口中的方法参数为多个时，如上第二种，此时MyBatis会自动将这些参数放在一个map集合中，以arg0,arg1或param1,param2...为键，以参数为值，此时可以使用${arg0}或#{arg0}等集合的键获取参数的值；
3. 若mapper接口中的方法参数为多个时，可以手动创建map集合来传递参数，如上第三种，此时可以使用${}或#{}访问该集合的键获取参数的值；
4. 若mapper接口中的方法参数为实体类对象时，如上第四种，此时可以使用${}和#{}，通过访问实体类对象中的属性名获取属性值
5. 可以通过@Param注解标识mapper接口中的方法参数，这样会自动以@Param注解的value属性值为键，以参数为值，放入一个map集合，此时可以使用${}和#{}，访问注解的属性值就可以获取相对应的值；

## SELECT

1. 查询的标签select必须设置属性resultType或resultMap，用于设置实体类和数据库表的映射关系
   resultType：自动映射，用于属性名和表中字段名一致的情况
   resultMap：自定义映射，用于一对多或多对一或字段名和属性名不一致的情况

2. 当查询的数据为多条时，不能使用实体类作为返回值，只能使用集合，否则会抛出异常TooManyResultsException；但是若查询的数据只有一条，可以使用实体类或集合作为返回值;

```
User getUserById(@Param("id") int id);
<!--User getUserById(@Param("id") int id);-->
<select id="getUserById" resultType="User">
select * from t_user where id = #{id}
</select>
```

### 按查询结果分类

```xml
1.查询单个实体类对象
<!--User getUserById(@Param("id") String id);-->
<select id="getUserById" resultType="User">
    select * from t_user_basic where id = #{id}
</select>
2.查询一个list集合
<!--List<User> getAllUser();-->
<select id="getAllUser" resultType="User">
    select * from t_user_basic
</select>
3.查询单个数据
<!--Integer getUserCount();-->
<select id="getUserCount" resultType="Integer">
    select count(*) from t_user_basic
</select>
4.查询一条数据为map集合
<!--Map<String,Object> getUserByIdToMap(@Param("id") String id);-->
<select id="getUserByIdToMap" resultType="Map">
    select * from t_user_basic where id = #{id}
</select>
5.查询多条数据为map集合
<!--可以将多条数据分别对应的Map放入一个List返回-->
<!--List<Map<String,Object>> getAllUserToMap();-->
<!--也可以将多条数据分别对应的Map，通过@MapKey注解设置新Map集合的键，放入一个Map返回-->
<!--@MapKey("id")
	Map<String,Object> getAllUserToMap();-->
<select id="getAllUserToMap" resultType="Map">
    select * from t_user_basic
</select>
```

## LIKE 模糊查询

**Mysql模糊查询常见写法**

```xml
<!--List<Map<String,Object>> getUserByLike(@Param("username") String username);-->
<select id="getUserByLike" resultType="Map">
    <!--select * from t_user_basic where username like '%${username}%'-->
    <!--select * from t_user_basic where username like concat('%',#{username},'%')-->
    select * from t_user_basic where username like "%"#{username}"%"
</select>
```

**在Oracle注意：**Oracle中concat函数只能传两个参数，若按此写需要嵌套一下；第三种写法不适用于Oracle；

还可以在代码中先进行拼接，后直接传入sql中，如`String username = "%abc%";`

## IN

```xml
<!--void deleteMore(@Param("ids") String ids);-->
<delete id="deleteMore">
    delete from t_user_basic where id in (${ids})
</delete>
```

## 动态设置表名

```xml
<!--List<Map<String,Object>> getSomething(@Param("tableName") String tableName);-->
<select id="getSomething" resultType="Map">
    select * from ${tableName}
</select>
```

## 获取自增的主键

**useGeneratedKeys：**设置使用自增的主键

**keyProperty：**因为增删改有统一的返回值是受影响的行数，因此只能将获取的自增的主键放在传输的参数user对象的某个属性中

```xml
<!--void  insertUserReturn(User user);-->
<insert id="insertUserReturn" useGeneratedKeys="true" keyProperty="id">
    insert into t_user_basic values (null,#{username},#{password},#{age},#{sex},#{email})
</insert>
```

# 自定义映射 resultMap

## 处理字段和属性的映射关系

若字段名和实体类中的属性名不一致，如字段名符合数据库的规则（使用\_），实体类中的属性名符合Java的规则（使用驼峰）;
此时可通过以下方式处理字段名和实体类中的属性的映射关系：

1. 可以通过为字段起别名的方式，保证和实体类中的属性名保持一致

```xml
<!--List<Emp> getAllEmp();-->
<select id="getAllEmp" resultType="Emp">
    select eid, emp_name empName, sex from t_emp;
</select> 
```

2. 可以在MyBatis的核心配置文件中设置一个全局配置信息mapUnderscoreToCamelCase，可以在查询表中数据时，自动将下划线字段名转换为驼峰。例如：字段名user_name，会转换为userName

```xml
<settings>
    <!--将表中字段的下划线自动转换为驼峰-->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

3. 可以通过resultMap设置自定义映射

```xml
<!-- resultMap：设置自定义映射 属性：id：表示自定义映射的唯一标识 type：查询的数据要映射的实体类的类型
	子标签：id：设置主键的映射关系 result：设置普通字段的映射关系
	属性：property：设置映射关系中实体类中的属性名 column：设置映射关系中表中的字段名-->
<resultMap id="EmpResultMap" type="Emp">
    <id property="eid" column="eid"/>
    <result property="empName" column="emp_name"/>
    <result property="sex" column="sex"/>
</resultMap>
<!--List<Emp> getAllEmp();-->
<select id="getAllEmp" resultMap="EmpResultMap">
    select * from t_emp;
</select>
```

## 多对一映射处理

### 级联方式处理映射关系

通过对象.属性的方式将多表联查结果字段与属性进行映射

```xml
<resultMap id="EmpResultMap" type="Emp">
    <id property="eid" column="eid"/>
    <result property="empName" column="emp_name"/>
    <result property="sex" column="sex"/>
    <result property="group.gid" column="gid"/>
    <result property="group.groupName" column="group_name"/>
</resultMap>
<!--Emp getEmpAndGroupByEid(int eid);-->
<select id="getEmpAndGroupByEid" resultMap="EmpResultMap">
    select * from t_emp e, t_group g where e.eid = #{eid} and e.gid = g.gid
</select>
```

### 使用association处理映射关系

使用association标签处理多表联查需要映射的字段

```xml
<resultMap id="EmpResultMap" type="Emp">
    <id property="eid" column="eid"/>
    <result property="empName" column="emp_name"/>
    <result property="sex" column="sex"/>
    <association property="group" javaType="group">
        <id property="gid" column="gid"/>
        <result property="groupName" column="group_name"/>
    </association>
</resultMap>
```

### 分步查询

将多表联查的情况分为几步，分别执行查询sql

```xml
<!--select：设置分步查询，查询某个属性的值的sql的标识（namespace.sqlId）
	column：将sql以及查询结果中的某个字段设置为分步查询的条件-->
<resultMap id="EmpStepResultMap" type="Emp">
    <id property="eid" column="eid"/>
    <result property="empName" column="emp_name"/>
    <result property="sex" column="sex"/>
    <association property="group" select="com.winton.mapper.GroupMapper.getGroupByGid" column="gid"/>
</resultMap>
<!--Emp getEmpByEid(@Param("eid") int eid);-->
<select id="getEmpByEid" resultMap="EmpStepResultMap">
    select * from t_emp where eid = #{eid}
</select>

<!--Group getGroupByGid(@Param("gid") int gid);-->
<select id="getGroupByGid" resultType="Group">
    select * from t_group where gid = #{gid}
</select>
```

## 一对多映射处理

### 使用collection处理映射关系

```xml
<resultMap id="GroupResultMap" type="Group">
    <id property="gid" column="gid"/>
    <result property="groupName" column="group_name"/>
    <collection property="emps" ofType="Emp"><!--ofType：设置collection标签所处理的集合属性中存储数据的类型-->
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="sex" column="sex"/>
    </collection>
</resultMap>
<!--List<Group> getGroupEmpByGid(@Param("gid") int gid);-->
<select id="getGroupEmpByGid" resultMap="GroupResultMap">
    select * from t_group g, t_emp e where g.gid = e.gid and g.gid = #{gid}
</select>
```

### 分步查询

```xml
<resultMap id="GroupResultMapStep" type="Group">
    <id property="gid" column="gid"/>
    <result property="groupName" column="group_name"/>
    <collection property="emps" select="com.winton.mapper.EmpMapper.getEmpByGid" column="gid"/>
</resultMap>
<!--Group getGroupByGidStep(@Param("gid") int gid);-->
<select id="getGroupByGidStep" resultMap="GroupResultMapStep">
    select * from t_group where gid = #{gid}
</select>

<!--List<Emp> getEmpByGid(@Param("gid") int gid);-->
<select id="getEmpByGid" resultType="Emp">
    select * from t_emp where gid = #{gid}
</select>
```

## 分步查询的延迟加载

分步查询的优点：可以实现延迟加载，即按需加载，访问到某属性时才去执行相应的查询sql。

```xml
<!--实现延迟加载，须在核心配置文件中设置全局配置信息-->
<settings>
    <!--开启延迟加载-->
    <setting name="lazyLoadingEnabled" value="true"/>
</settings>
<!--lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。(默认为false)
aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。(默认为false)-->

<!--在全局开启延迟加载时，在association和collection中，有fetchType属性可以设置当前的分步查询是否使用延迟加载，
fetchType="lazy(延迟加载)或eager(立即加载)"
```

# 动态Sql

## if

```xml
<!--List<User> getUserByCondition(@Param("username") String username,@Param("sex") String sex);-->
<select id="getUserByCondition" resultType="User">
    select * from t_user_basic where password is not null
    <if test="id != null and id != ''">
        and id = #{id}
    </if>
    <if test="sex != null and sex != ''">
        and sex = #{sex}
    </if>
</select>
```

## where

where和if一般结合使用：

1. 若where标签中的if条件都不满足，则不会添加where关键字
2. 若where标签中的if条件满足，则where标签会自动添加where关键字，并将条件最前方多余的and或or去掉（条件后面的不能自动去掉）

```xml
<!--List<User> getUserByCondition(@Param("username") String username,@Param("sex") String sex);-->
<select id="getUserByCondition" resultType="User">
    select * from t_user_basic
    <where>
        <if test="id != null and id != ''">
            and id = #{id}
        </if>
        <if test="sex != null and sex != ''">
            and sex = #{sex}
        </if>
    </where>
</select>
```

## trim

trim用于去掉或添加标签中的内容

**常用属性：**

prefix：在trim标签中的内容的前面添加某些内容
prefixOverrides：在trim标签中的内容的前面去掉某些内容

suffix：在trim标签中的内容的后面添加某些内容
suffixOverrides：在trim标签中的内容的后面去掉某些内容

```xml
<!--int updateUserByCondition(HashMap<String,Object> param);-->
<update id="updateUserByCondition">
    update t_user_basic set
    <trim prefix="" prefixOverrides=",">
        <if test="password != null">
            password = #{password}
        </if>
        <if test="email != null">
            , email = #{email}
        </if>
    </trim>
    <trim prefix="where" prefixOverrides="and">
        <if test="id != null">
            id = #{id}
        </if>
        <if test="username != null">
            and username = #{username}
        </if>
    </trim>
</update>
```

## choose、when、overwise

choose、when、otherwise类似于switch..case..default

```xml
<!--List<User> getUserByChoose(HashMap<String,Object> param);-->
<select id="getUserByChoose" resultType="User">
    select * from t_user_basic where
    <choose>
        <when test="id != null">
            id = #{id}
        </when>
        <when test="email != null">
            email = #{email}
        </when>
        <otherwise>
            username = #{username}
        </otherwise>
    </choose>
</select>
```

## foreach

foreach用于循环处理参数中的数组或集合

**常用属性：**

collection：设置要循环的数组或集合
item：表示集合或数组中的每一个数据
separator：设置循环体之间的分隔符
open：设置foreach标签中的内容的开始符
close：设置foreach标签中的内容的结束符

```xml
<!--List<User> getUserByForeach(@Param("ids") int[] ints);-->
<select id="getUserByForeach" resultType="User">
    select * from t_user_basic where id in
    <foreach collection="ids" open="(" close=")" separator="," item="id">
        #{id}
    </foreach>
</select>
```

## sql片段

sql片段，可以记录一段公共sql片段，在使用的地方通过include标签进行引入

```xml
<sql id="userBasicColumn">
    id, username, password, age, sex, email
</sql>
<!--List<User> getAllUser();-->
<select id="getAllUser" resultType="User">
    select <include refid="userBasicColumn"/> from t_user_basic
</select>
```

# 缓存

**仅针对查询功能，将查询结果放在内存里，某些场景下查询时，将不从数据库而是从缓存中获取数据**

## MyBatis的一级缓存

**一级缓存是SqlSession级别的，**通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问

使一级缓存失效的四种情况：

1. 不同的SqlSession对应不同的一级缓存
2. 同一个SqlSession但是查询条件不同
3. 同一个SqlSession两次查询期间执行了任何一次增删改操作
4. 同一个SqlSession两次查询期间手动清空了缓存 `sqlSession.clearCache();`

## MyBatis的二级缓存

**二级缓存是SqlSessionFactory级别，**通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取

二级缓存开启的条件：

1. 在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认即为true
2. 在映射文件中设置标签 `<cache/>`
3. 二级缓存必须在SqlSession关闭或提交之后才有效
4. 查询的数据所转换的实体类类型必须实现序列化的接口 `implements Serializable`

使二级缓存失效的情况：两次查询之间执行了任意的增删改，会使一级和二级缓存同时失效

### 二级缓存的相关配置

cache标签中包含属性：

* eviction属性：缓存回收策略
  LRU（Least Recently Used） – 最近最少使用的：移除最长时间不被使用的对象。
  FIFO（First in First out） – 先进先出：按对象进入缓存的顺序来移除它们。
  SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
  WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
  默认的是 LRU。

* flushInterval属性：刷新间隔，单位毫秒
  默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新

* size属性：引用数目，正整数
  代表缓存最多可以存储多少个对象，太大容易导致内存溢出

* readOnly属性：只读，true/false
  true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了
  很重要的性能优势。
  false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是
  false。
* type属性：二级缓存的类型，可以修改为第三方的缓存插件包。

## MyBatis缓存查询的顺序

先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用。如果二级缓存没有命中，再查询一级缓存，如果一级缓存也没有命中，则查询数据库。SqlSession关闭之后，一级缓存中的数据会写入二级缓存

# 分页插件

**添加依赖和插件**

```xml
<!--在pom.xml中添加依赖-->
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.2.0</version>
</dependency>

<!--在mabatis核心配置文件添加插件-->
<plugins>
    <!--设置分页插件-->
    <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>
```

**使用步骤：**

1. 在查询功能之前使用 `PageHelper.startPage(int pageNum, int pageSize)` 开启分页功能
   pageNum：当前页的页码	pageSize：每页显示的条数
2. 在查询获取list集合之后，可以使用 `PageInfo<T> pageInfo = new PageInfo<>(List<T> list, int
   navigatePages)` 获取分页相关数据
   list：分页之后的数据	navigatePages：导航分页的页码个数
3. 分页相关数据

> pageNum：当前页的页码
> pageSize：每页显示的条数
> size：当前页显示的真实条数
> total：总记录数
> startRow：开始记录数
> endRow：结束记录数
> pages：总页数
> prePage：上一页的页码
> nextPage：下一页的页码
> isFirstPage/isLastPage：是否为第一页/最后一页
> hasPreviousPage/hasNextPage：是否存在上一页/下一页
> navigatePages：导航分页的页码个数
> navigateFirstPage：导航分页的最前页码
> navigateLastPage：导航分页的最后页码
> navigatepageNums：导航分页的页码，[1,2,3,4,5]

```java
public void testPageHelper() {
    SqlSession session = DatabaseUtils.getSession();
    SelectMapper mapper = session.getMapper(SelectMapper.class);
    PageHelper.startPage(3, 2);
    List<User> list = mapper.getAllUser();
    PageInfo<User> pageInfo = new PageInfo<>(list, 3);
    System.out.println(pageInfo);
}
//PageInfo{pageNum=3, pageSize=2, size=2, startRow=5, endRow=6, total=9, pages=5, list=Page{count=true, pageNum=3, pageSize=2, startRow=4, endRow=6, total=9, pages=5, reasonable=false, pageSizeZero=false}[User{id=7, username='syl', password='123456', age=18, sex='0', email='777@qq.com'}, User{id=10, username='admin', password='admin', age=30, sex='男', email='admin@qq.com'}], prePage=2, nextPage=4, isFirstPage=false, isLastPage=false, hasPreviousPage=true, hasNextPage=true, navigatePages=3, navigateFirstPage=2, navigateLastPage=4, navigatepageNums=[2, 3, 4]}
```

# 逆向工程

* 正向工程：先创建Java实体类，由框架负责根据实体类生成数据库表。Hibernate是支持正向工程的。
* 逆向工程：先创建数据库表，由框架负责根据数据库表，反向生成Java实体类、Mapper接口、Mapper映射文件。

## 创建步骤：

**1. pom.xml添加依赖和插件**

```xml
<!-- 控制Maven在构建过程中相关配置 -->
<build>
    <!-- 构建过程中用到的插件 -->
    <plugins>
        <!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.0</version>
            <!-- 插件的依赖 -->
            <dependencies>
                <!-- 逆向工程的核心依赖 -->
                <dependency>
                    <groupId>org.mybatis.generator</groupId>
                    <artifactId>mybatis-generator-core</artifactId>
                    <version>1.3.2</version>
                </dependency>
                <!-- 数据库连接池 -->
                <dependency>
                    <groupId>com.mchange</groupId>
                    <artifactId>c3p0</artifactId>
                    <version>0.9.2</version>
                </dependency>
                <!-- MySQL驱动 -->
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>5.1.3</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

**2.创建逆向工程的配置文件**

文件名固定为：generatorConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--targetRuntime: 执行生成的逆向工程的版本
        MyBatis3Simple: 生成基本的CRUD（清新简洁版）    MyBatis3: 生成带条件的CRUD（奢华尊享版）-->
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://114.115.167.249:3306/winton?characterEncoding=utf8"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.winton.bean"
                            targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- SQL映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="com.winton.mapper"
                         targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.winton.mapper" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 逆向分析的表 -->
        <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
        <!-- domainObjectName属性指定生成出来的实体类的类名 -->
        <table tableName="t_emp" domainObjectName="Emp"/>
        <table tableName="t_group" domainObjectName="Group"/>
    </context>
</generatorConfiguration>
```

**3.点击侧边栏Maven工程的Plugins中mabatis-generator插件完成逆向构建**

**4.创建mybatis核心配置文件，开始使用**

## QBC查询

逆向工程自动生成了一种创建XxxExample条件对象，拼接条件的执行SQL方法，详细过程可以看自动生成的XxxMapper.xml

```java
public void testMBG() throws IOException {
    InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSession sqlSession = new SqlSessionFactoryBuilder().build(is).openSession(true);
    EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
    //创建条件对象，通过andXXX方法为SQL添加查询添加，每个条件之间是and关系
    EmpExample empExample = new EmpExample();
    empExample.createCriteria().andEidBetween(1, 3);
    //将之前添加的条件通过or拼接其他条件
    empExample.or().andEmpNameEqualTo("老七");
    List<Emp> list = mapper.selectByExample(empExample);
    System.out.println(list);
}
```



# 问题整理

1. **1 字节的 UTF-8 序列的字节 1 无效**

**解决方案：**

```
在Mybatis配置文件中添加
<properties>
    <project.build.sourceEncoding>UTF8</project.build.sourceEncoding>
</properties>
```

2. **java.lang.IllegalArgumentException: Result Maps collection does not contain value**

**原因：**

```
在mapper.xml文件中，将resultType和resultMap搞混了，该用resultType写了resultMap却又没定义这个resultMap
```

3. **Cause: org.apache.ibatis.builder.BuilderException: Ambiguous collection type for property 'emps'. You must specify 'javaType' or 'resultMap'.**

**原因：**

```
在mapper.xml文件中，多对一映射<collection property="emps" ofType="Emp">中的emps，同样需要在对象中建立属性
```
