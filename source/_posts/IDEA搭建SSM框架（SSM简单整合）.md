---
title: IDEA搭建SSM框架（SSM简单整合）
date: 2020-01-15 16:49:50
tags:Java,SSM,Mybatis,Spring,Spring MVC
---
SSM（Spring、Spring MVC和Mybatis）是什么我想也不用在赘述。
许多童鞋现在开始学习这个流行的框架来进行Java开发，想要寻找一个最简单的SSM框架搭建方法，这里我不说什么废话，直接上手开始搭建，代码部分都做了详细的注释，可以快速上手！
**本文使用的是注解，我希望在搭建SSM框架之前最好对基础知识有所掌握，能够知道每个注解的作用以及配置方式，可以节省不少时间，我希望还不知道什么是注解的朋友，先去学习一下。**
## 创建Java Web项目
用到的开发工具是IntelliJ IDEA，项目创建可能和eclipse和myeclipse有所不同，按照自己的需要来创建就好，用什么IDE就按照什么步骤来创建。
以下是完整目录结构，不论什么IDE都可以是这种结构，具体内容看图：
![](/media/img/ef39bb0a.PNG)
## 导入jar包
因为我们不采用maven的方式配置jar包,所以需要我们手动导入jar文件。这一步就不再多说，以下会和源代码一同给出。
> 更新：使用MAVEN的话，最好在开发电脑中安装MAVEN，SSM的MAVEN配置文件在网上有很多不同的版本，本文中不再给出MAVEN的pom.xml文件，具体用法可以先学MAVEN之后，再到网上找自己对应的包的配置方式，自己实现一下就好。

## 配置文件
这里我们分别将Spring 和 Mybatis的配置文件放在两个文件夹中，Spring的有applicationContext.xml和applicationContext-mvc.xml,Mybatis的有mybatis-config.xml。
### Mybatis配置文件
Mybatis的配置文件就是mybatis-config.xml，主要是配置typeAlias，将实体类匹配成XXXMapper.xml中可以直接使用的类型，相当于一个别名，在XXXMapper.xml中就无需再写完整的实体类全路径，直接用alias的值来代替。
```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD SQL Map Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
<!--之后用于测试-->
        <typeAlias type="com.javafeng.entity.User" alias="User" />
    </typeAliases>
</configuration>
```
其中的XXXMapper.xml当然就是Mybatis动态实现所需要的Mapper文件，Dao接口就可以不用再编写实现类。这里的Mapper和Dao中的接口是对应的，再接下来的测试中我们会给出具体的配置。
### Spring配置文件

- applicationContext.xml

在这个配置文件中,我们主要配置数据源,Spring的事务管理和Dao接口的扫描，以及对Mybatis的一些列相关配置文件的扫描。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!--数据源-链接数据库的基本信息,这里直接写,不放到*.properties资源文件中-->
    <bean id="dataSource"
          class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/javafeng" />
        <property name="username" value="root" />
        <property name="password" value="root" />
    </bean>
    <!-- 配置数据源,加载配置,也就是dataSource -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <!--mybatis的配置文件-->
        <property name="configLocation" value="classpath:mybatis/mybatis-config.xml" />
        <!--扫描 XXXmapper.xml映射文件,配置扫描的路径-->
        <property name="mapperLocations" value="classpath:com/javafeng/mapping/*.xml"></property>
    </bean>
    <!-- DAO接口所在包名，Spring会自动查找之中的类 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.javafeng.dao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>

    <!--事务管理-->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入dataSource-->
        <property name="dataSource" ref="dataSource" />
    </bean>
    <!--开启事务注解扫描-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
</beans>
```

- applicationContext-mvc.xml

这个配置文件中我们主要启用Sping注解驱动,进行静态资源的配置,注解扫描配置和视图解析器配置。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 告知Spring，我们启用注解驱动 -->
    <mvc:annotation-driven/>
    <!-- org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，
    它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，
    就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。 -->
    <mvc:default-servlet-handler/>
    <!-- 指定要扫描的包的位置 -->
    <context:component-scan base-package="com.javafeng" />
    <!-- 对静态资源文件的访问,因为Spring MVC会拦截所有请求,导致jsp页面中对js和CSS的引用也被拦截,配置后可以把对资源的请求交给项目的
    默认拦截器而不是Spring MVC-->
    <mvc:resources mapping="/static/**" location="/WEB-INF/static/" />

    <!-- 配置Spring MVC的视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 有时我们需要访问JSP页面,可理解为在控制器controller的返回值加前缀和后缀,变成一个可用的URL地址 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

- web.xml

我们在web.xml中加载Spring配置,并且将所有的请求都过滤给Spring MVC来处理,同时设置编码过滤器解决编码问题(最后一项可以不配置)。
其中Spring MVC的请求过滤就是一个简单的Servlet配置。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!-- 加载Spring容器配置 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- Spring容器加载所有的配置文件的路径 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring/applicationContext.xml</param-value>
    </context-param>
    <!-- 配置SpringMVC核心控制器,将所有的请求(除了刚刚Spring MVC中的静态资源请求)都交给Spring MVC -->
    <servlet>
        <servlet-name>springMvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:spring/applicationContext-mvc.xml</param-value>
        </init-param>
        <!--用来标记是否在项目启动时就加在此Servlet,0或正数表示容器在应用启动时就加载这个Servlet,
        当是一个负数时或者没有指定时，则指示容器在该servlet被选择时才加载.正数值越小启动优先值越高  -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!--为DispatcherServlet建立映射-->
    <servlet-mapping>
        <servlet-name>springMvc</servlet-name>
        <!-- 拦截所有请求,千万注意是(/)而不是(/*) -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- 设置编码过滤器 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
到这里我们的框架基本上就成形了，接下来可以进行简单的测试。

## 测试

接下来就是应用实例，我会在文章结尾给出源码下载地址。
我们在数据库新建如下表（我的数据库名为javafeng，用的MySql）：

```sql
CREATE TABLE `user` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `Name` varchar(255) DEFAULT NULL,
  `Age` int(11) DEFAULT NULL,
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

这里我们用逆向工程来生成实体类，Dao接口和对应的Mapper文件，具体方法参考：

[点击打开链接](https://blog.csdn.net/zhshulin/article/details/23912615 "点击打开链接")（版权归原作者所有），逆向工程的工具同代码一并奉上。我在里面分别添加了可以查询表内全部数据的代码。
实体类User.java  
```java
package com.javafeng.entity;

public class User {
    private Integer id;

    private String name;

    private Integer age;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name == null ? null : name.trim();
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```
Mapper文件UserMapper.xml（自动生成，篇幅长也不可怕，前提是你已经基本掌握）：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<!--namespace就是与此文件对应的Dao接口的全路径-->
<mapper namespace="com.javafeng.dao.IUserDao" >
  <!--如下type的User就是mybatis-config.xml中配置的user-->
  <resultMap id="BaseResultMap" type="User" >
    <id column="ID" property="id" jdbcType="INTEGER" />
    <result column="Name" property="name" jdbcType="VARCHAR" />
    <result column="Age" property="age" jdbcType="INTEGER" />
  </resultMap>
<!--自己配置的查询表所有数据的sql-->
  <select id="selectAllUser" resultType="User">
    select * FROM USER;
  </select>


  <sql id="Base_Column_List" >
    ID, Name, Age
  </sql>
  <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Integer" >
    select 
    <include refid="Base_Column_List" />
    from user
    where ID = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer" >
    delete from user
    where ID = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="User" >
    insert into user (ID, Name, Age
      )
    values (#{id,jdbcType=INTEGER}, #{name,jdbcType=VARCHAR}, #{age,jdbcType=INTEGER}
      )
  </insert>
  <insert id="insertSelective" parameterType="User" >
    insert into user
    <trim prefix="(" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        ID,
      </if>
      <if test="name != null" >
        Name,
      </if>
      <if test="age != null" >
        Age,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        #{id,jdbcType=INTEGER},
      </if>
      <if test="name != null" >
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="age != null" >
        #{age,jdbcType=INTEGER},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="User" >
    update user
    <set >
      <if test="name != null" >
        Name = #{name,jdbcType=VARCHAR},
      </if>
      <if test="age != null" >
        Age = #{age,jdbcType=INTEGER},
      </if>
    </set>
    where ID = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="User" >
    update user
    set Name = #{name,jdbcType=VARCHAR},
      Age = #{age,jdbcType=INTEGER}
    where ID = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```
Dao层接口IUserDao.java，生成时为UserMapper.java，我这里进行了重命名，注意一定要记得同时修改Mapper文件中的namespace：
```java
package com.javafeng.dao;

import com.javafeng.entity.User;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository("userDao")
public interface IUserDao {
    int deleteByPrimaryKey(Integer id);

    int insert(User record);

    int insertSelective(User record);

    User selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(User record);

    int updateByPrimaryKey(User record);
    //自己添加的，已匹配Mapper中的Sql
    List<User> selectAllUser();
}

```
接下来是Service层接口UserService.java：
```java
package com.javafeng.service;

import com.javafeng.entity.User;
import java.util.List;

public interface UserService {
    public List<User> getUser();
}

```
Service接口实现类UserServiceImpl.java：
```java
package com.javafeng.service.impl;

import com.javafeng.dao.IUserDao;
import com.javafeng.entity.User;
import com.javafeng.service.UserSercice;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

@Service("userService")
public class UserServiceImpl implements UserService{

    @Resource(name = "userDao")
    private IUserDao userDao;

    @Override
    public List<User> getUser() {
        return userDao.selectAllUser();
    }
}

```
控制器UserController.java
```java
package com.javafeng.controller;

import com.javafeng.entity.User;
import com.javafeng.service.UserSercice;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.annotation.Resource;
import java.util.List;

@Controller
@RequestMapping(value = "/user")
public class UserController {
    @Resource(name = "userService")
    UserSercice userService;

    @RequestMapping(value = "/list")
    public ModelAndView list()
    {
        ModelAndView mv=new ModelAndView();
        List<User>  userList=userService.getUser();
        mv.addObject("userList",userList);
        mv.setViewName("/show");
        return mv;
    }

}

```
我们要做的就是把这个表的数据显示在一个Jsp页面上，所以在WEB-INF/jsp下新建一个show.jsp来显示数据
```html
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%--
  Created by IntelliJ IDEA.
  User: 13926
  Date: 2017/7/18
  Time: 23:07
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<table border="1">
    <tr>
        <td>序号</td>
        <td>姓名</td>
        <td>年龄</td>
    </tr>
        <c:choose>
            <c:when test="${not empty userList}">
                <c:forEach items="${userList}" var="user" varStatus="vs">
                    <tr>
                        <td>${user.id}</td>
                        <td>${user.name}</td>
                        <td>${user.age}</td>
                    </tr>
                </c:forEach>
            </c:when>
            <c:otherwise>
               <tr>
                   <td colspan="2">无数据!</td>
               </tr>
            </c:otherwise>
        </c:choose>
</table>
</body>
</html>
```
我们在index.html中做如下更改来使项目启动时自动访问user/list路径（其实就是懒得每次都输这个地址，因为测试时大多数时候都不是一次就成）
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
  <jsp:forward page="/user/list"/>
  </body>
</html>
```
项目部署不用多讲，直接发布到Tomcat服务器即可，因为已经才index.html做了修改，所以已不需要在地址栏输可恶的user/list路径了，直接http://localhost:8080/SSMDemoByIdea
这样我们就能把User表的数据显示在show.jsp了。
![](/media/img/a7ec20d8.PNG)
到这里我们的SSM框架搭建算是圆满完成，要是你还在报错中，恭喜，你可以接着继续调试了！

## 注意

-  无论是注解的扫描还是配置文件的扫描，路径千万要写对，路径千万要写对，路径千万要写对
- 再写Mapper和Dao接口时，一定要对应上，否则```Invalid bound statement (not found)```
-  Mapper在自动生成后，一定要按照自己项目的内容进行修改，比如namespace要和Dao接口对应，以及其中parameterType,resultType所对应的类型时你mybatis-config.xml中配置为alias值等等等等，总之千万要注意！
- 再次说明，本文使用的是注解，我希望在搭建SSM框架之前最好对基础知识有所掌握，能够知道每个注解的作用以及配置方式，可以节省不少时间

## 文件下载地址

[点击下载](https://pan.baidu.com/s/1eSGLRJc "点击下载")