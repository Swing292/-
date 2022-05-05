[TOC]

# SpringMVC的基本概念

1. MVC：model view controller

2. 基于 Java的实现 **MVC设计模型**的请求驱动类型的轻量级web框架

3. 是表现层框架：

   ![image-20220503183445899](SpringMVC.assets/image-20220503183445899.png)

# 入门案例

1. 搭建开发环境

   1. 新建项目：**下图选错了，是maven-archetype-webapp**![image-20220503184157164](SpringMVC.assets/image-20220503184157164.png)

   2. 添加一个键值对，解决maven工程创建过慢：

      1. archetypeCatalog
      2. internal

      ![image-20220503185412732](SpringMVC.assets/image-20220503185412732.png)

   3. 新增java和resources目录，导坐标：

      ```xml
      <properties>
      <!--    版本锁定-->		<spring.version>5.0.2.RELEASE</spring.version>
        </properties>
      
        <dependencies>
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
          </dependency>
      
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
          </dependency>
      
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
          </dependency>
      
          <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
          </dependency>
      
          <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.0</version>
            <scope>provided</scope>
          </dependency>
        </dependencies>
      ```

   4. 在web.xml中配置前端控制器：

      ```xml
      <web-app>
        <display-name>Archetype Created Web Application</display-name>
        
        <servlet>
          <servlet-name>dispatcherServlet</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        </servlet>
        <servlet-mapping>
          <servlet-name>dispatcherServlet</servlet-name>
          <url-pattern>/</url-pattern>
        </servlet-mapping>
      </web-app>
      ```

   5. 新建配置文件springmvc：

      ![image-20220504225523943](SpringMVC.assets/image-20220504225523943.png)

   6. 部署服务器，把项目构建进来（fix一下就好）：

      ![image-20220504225823176](SpringMVC.assets/image-20220504225823176.png)

2. 编写入门程序

   1. 编写控制类：

      ```java
      /**
       * 控制器类
       */
      @Controller
      public class HelloController {
          @RequestMapping
          public String sayHello(){
              System.out.println("hello SpringMVC");
              return null;
          }
      }
      ```

      给请求方法添加`@RequestMapping`注释，path属性添加路径。

   2. 在web.xml中提供参数加载配置文件：

      ```xml
      <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      
        <!--  提供全局参数-->
        <init-param>
          <param-name>contextConfigLocation</param-name>
        <!--  帮助启动加载配置文件-->
          <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
      </servlet>
      ```

   3. 配置视图解析器对象，帮关注跳转到指定的页面：

      ![image-20220504234056856](SpringMVC.assets/image-20220504234056856.png)

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
                 xmlns:mvc="http://www.springframework.org/schema/mvc"
                 xmlns:context="http://www.springframework.org/schema/context"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="
              http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/mvc
              http://www.springframework.org/schema/mvc/spring-mvc.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context.xsd">
      
      <!--    开启注解的扫描-->
          <context:component-scan base-package="cn.ning"></context:component-scan>
      
      <!--    配置视图解析器对象：可以帮助跳转到指定的页面-->
          <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
              <!--表示文件所在的目录-->
              <property name="prefix" value="/WEB-INF/pages/"></property>
              <!--指定文件的后缀名-->
              <property name="suffix" value=".jsp"></property>
          </bean>
      
      <!--    开启SpringMVC注解的支持-->
          <mvc:annotation-driven></mvc:annotation-driven>
      </beans>
      ```

3. 总结：

   ![image-20220504235227499](SpringMVC.assets/image-20220504235227499.png)

4. 详细总结：

   ![image-20220505084919377](SpringMVC.assets/image-20220505084919377.png)

   1. DispatcherServlet：

      1. 前端控制器
      2. 用户请求到达前端控制器，它就相当于 mvc 模式中的 c，dispatcherServlet 是整个流程控制的中心，由 它调用其它组件处理用户的请求，dispatcherServlet 的存在降低了组件之间的耦合性。

   2. HandlerMapping：

      1. 处理器映射器
      2. HandlerMapping 负责根据用户请求找到 Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的 映射方式，例如：配置文件方式，实现接口方式，注解方式等。

   3. Handler：

      1. 处理器
      2. 它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由 Handler 对具体的用户请求进行处理。

   4. HandlAdapter：

      1. 处理器适配器
      2. 通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理 器进行执行。

   5. View Resolver：

      1. 视图解析器
      2. View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名 即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。

   6. View：

      1. 视图

      2. SpringMVC 框架提供了很多的 View 视图类型的支持，包括：jstlView、freemarkerView、pdfView 等。我们最常用的视图就是 jsp。 

         一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开 发具体的页面。

   7. `<mvc:annotation-driven>`说明:

      1. 在 SpringMVC 的各个组件中，处理器映射器、处理器适配器、视图解析器称为 SpringMVC 的三大组件。 

         使用自动加载RequestMappingHandlerMapping （处理映射器）RequestMappingHandlerAdapter （ 处 理 适 配 器 ），可 用 在 SpringMVC.xml 配 置 文 件 中 使 用 替代注解处理器和适配器的配置。

         ```xml
         <!--    开启SpringMVC注解的支持-->
             <mvc:annotation-driven></mvc:annotation-driven>
         ```

## RequestMapping注解

1. 作用：建立请求URL和处理请求方法之间的对应关系

2. 位置：

   1. 方法前

   2. 类前：一级目录

      ![image-20220505085833600](SpringMVC.assets/image-20220505085833600.png)

3. 属性：

   1. path属性（value属性）：指定请求的URL

   2. method属性：

      1. 指定请求方法类型

      2. 格式：`method = RequestMethod.GET`

      3. RequestMethod是一个枚举类：

         ![image-20220505090328322](SpringMVC.assets/image-20220505090328322.png)

   3. params属性：指定限制请求参数的条件

      1. 格式：`params = {"username"}`

      2. 页面就必须传入参数username

         ```html
         <a href="user/testRequestMapping?username">RequestMapping注释</a>
         ```

      3. 限制传入参数值的写法：`params = {"username=root"}`

   4. headers属性：

      1. 指定限制请求消息头的条件
      2. 格式：`headers = "Accept"
      3. 必须有Accept请求头才能接收请求![image-20220505091615446](SpringMVC.assets/image-20220505091615446.png)

# 请求参数的绑定

1. 基本数据类型绑定：

   ![image-20220505093218575](SpringMVC.assets/image-20220505093218575.png)

2. 实体类型绑定：

   1. 把数据封装到JavaBean的类中：

      ![image-20220505094716038](SpringMVC.assets/image-20220505094716038.png)

   2. 类中有类时：

      ```html
      用户姓名：<input type="text" name="user.uname">
      用户年龄：<input type="text" name="user.age">
      ```

3. 集合类型绑定：

   ![image-20220505155546289](SpringMVC.assets/image-20220505155546289.png)

## 配置解决中文乱码的过滤器

在web.xml中配置filter：

```xml
<!--  配置解决中文乱码的过滤器-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

<!--    初始化参数-->
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>

  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

如果是Tomcat乱码就配置一下`-Dfile.encoding=UTF-8`

修改后重新加载一下工程

## 自定义类型转换器

### 步骤

1. 定义一个类，实现Converter接口，该接口有两个泛型