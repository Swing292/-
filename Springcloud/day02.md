[TOC]

# Nacos配置管理

## 统一配置管理

- 在Nacos中添加配置文件

  1. 配置管理-配置列表：

     ![image-20220923223647837](Untitled.assets/image-20220923223647837.png)

  2. 新建配置-发布：

     ![image-20220923223502562](Untitled.assets/image-20220923223502562.png)

- 配置获取的过程：

  ![image-20220923234728149](Untitled.assets/image-20220923234728149.png)

- 步骤：

  1. 在Nacos中添加配置文件

  2. 引入Nacos的配置管理客户端依赖：

     ```xml
     <!--        nacos的配置管理依赖-->
             <dependency>
                 <groupId>com.alibaba.cloud</groupId>
                 <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
             </dependency>
         </dependencies>
     ```

  3. 在userservice中的resource目录添加一个bootstrap.ym|文件：

     ```yml
     spring:
       application:
         name: userservice
       profiles:
         active: dev #环境
       cloud:
         nacos:
           server-addr: localhost:8848 #nacos地址
           config:
             file-extension: yaml #文件后缀名
     ```

     - 这个文件是引导文件，优先级高于application.yml
     - 配置nacos地址、当前环境、服务名称、文件后缀名。这些决定了程序
       启动时去nacos读取哪个文件。

- demo：

  ```java
  @RestController
  @RequestMapping("/user")
  public class UserController {
  
      @Value("${pattern.dateformat}")
      private String dateformat;
  
      @GetMapping("now")
      public String now(){
          return LocalDateTime.now().format(DateTimeFormatter. ofPattern(dateformat));
      }
  ```

  读取成功：

  <img src="Untitled.assets/image-20220924000912247.png" alt="image-20220924000912247" style="zoom:50%;" />

## 配置热更新

- **配置自动刷新**：Nacos中的配置文件变更后，微服务无需重启就可以感知。

- 通过两种配置实现：

  1. 在`@Value`注入的变量**所在类**上添加注解`@RefreshScope`

     <img src="Untitled.assets/image-20220924001251324.png" alt="image-20220924001251324" style="zoom: 67%;" />

  2. （建议）使用`@ConfigurationProperties`注解：

     1. 新建配置类用于属性加载：

        ```java
        @Data
        @Component
        @ConfigurationProperties(prefix = "pattern")
        public class PatternProperties {
            private String dateformat;
        }
        ```

     2. 注入配置类后直接使用：

        ```java
        @RestController
        @RequestMapping("/user")
        public class UserController {
            
            @Autowired
            private PatternProperties properties;
        
            @GetMapping("now")
            public String now(){
                return LocalDateTime.now().format(DateTimeFormatter. ofPattern(properties.getDateformat()));
            }
        }
        ```

- 不是所有的配置都适合放到配置中心，维护起来比较麻烦

- 建议将一些关键参数，需要运行时调整的参数放到nacos配置中心，一般都是自定义配置

## 配置共享

- 微服务启动时会从nacos读取多个配置文件:
  1. `[spring.application.name]-[spring.profiles.active].yaml`, 例如: `userservice-dev.yaml`
  2. `[spring.application.namel.yaml`, 例如: `userservice.yaml`
- 无论profile如何变化，`[spring.application.name].yaml`这个文件一定会加载, 因此多环境共享配置可以写入这个文件

## 搭建Nacos集群

# Feign远程调用

# Gateway服务网关