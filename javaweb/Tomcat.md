[TOC]

# Tomcat介绍

## 版本

![image-20211009163226046](Untitled.assets/image-20211009163226046.png)

## 目录介绍

- bin：存放可执行程序
- conf：存放配置文件
- lib：存放jar包
- logs：Tomcat运行时输出的日记信息
- temp：Tomcat运行时产生的临时文件
- webapps：存放部署的web工程（一个文件夹一个工程）
- work：工作时的目录，用来存放Tomcat运行时jsp翻译为Servlet的源码，和Session钝化（序列化）的目录。

## 启动Tomcat服务器

1. 找到Tomcat目录下的bin目录下的startup.bat，双击启动。
2. 进入命令行，cd进入bin目录下，输入命令catalina run 

- 在浏览器输入网址测试是否启动成功：

  1. http://localhost:8080
  2. http://127.0.0.1:8080
  3. http://192.168.43.146:8080（http://真实ip:8080）

- 常见启动失败情况

  双击startup.bat文件时黑屏一闪而过 -> 没有配置好JAVA_HOME环境变量

## 停止Tomcat服务器

1. 点击tomcat服务器窗口的x关闭按钮
2. 把Tomcat服务器窗口置为当前窗口,然后按快捷键ctrl+C
3. 找到Tomcat的 bin目录下的shutdown.bat双击，就可以停止Tomcat服务器

## 修改Tomcat的端口号

- 说明：

  - mysql的默认端口是3306
  - Tomcat的默认端口是8080

- 修改方式：

  进去Tomcat目录下的conf目录，找到server.xml配置文件，找到Connection标签，修改port（范围：1~65535）

- 修改完需要重启服务器

- http协议默认的端口号是80（默认省略）

# 部署web工程

- 方式：

  1. 直接将web工程的目录拷贝到Tomcat的webapps目录下。

     访问web工程：网址后面加文件名访问

  2. 在conf目录\Catalina\localhost\下，创建如下配置文件：

     ```xml
     <context path=" / abc" dolBase="E: book"/>
     ```

     - Concexc表示一个工程上下文
     - path表示工程的访问路径: / abc
     - docBase表示工程目录位置

- 区别：

  ![image-20211009172520438](Untitled.assets/image-20211009172520438.png)

## ROOT工程

- 在浏览器地址栏中输入访问地址如下：htp:/p:por/
  即没有工程名时，默认访问的是ROOT工程。
- 当我们在浏览器地址栏中输入的访问地址如下：htp://p:por/工程名1 
  即没有资源名时， 默认访问index.html页面。

## 整合Idea

- File | Settings | Build, Execution, Deployment | Application Servers
- 创建web工程：https://blog.csdn.net/weixin_50180191/article/details/116521420

## 动态web工程目录

- 新建模块后右击Add Frameworks Support，勾选Web Application

![image-20211009193041629](Untitled.assets/image-20211009193041629.png)

- src：自己编写的Java源代码
- web目录：存放web工程的资源文件
  - WEB-INF：受服务器保护的目录，浏览器无法直接访问
    - lib：存放第三方jar包（需配置）
    - web.xml：整个动态web工程的配置部署描述文件，可以在这配置很多web工程的组件（如Servlet程序、Filter过滤器、Listener监听器、Session超时…）