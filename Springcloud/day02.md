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
- 无论profile如何变化，`[spring.application.name].yaml`这个文件一定会加载, 因此**多环境共享配置可以写入这个文件**
- **`服务名-profile.yaml` >`服务名称.yaml` >本地配置**
- 在idea修改服务的配置：
  1. <img src="Untitled.assets/image-20220925214235425.png" alt="image-20220925214235425" style="zoom:67%;" />
  2. ![image-20220925214302083](Untitled.assets/image-20220925214302083.png)

## 搭建Nacos集群

- **Nacos生产环境下一定要部署为集群状态**

  <img src="Untitled.assets/image-20220925215036877.png" alt="image-20220925215036877" style="zoom:80%;" />

- nacos集群的部署：

  1. 搭建数据库，初始化数据库表结构

     - 新建一个数据库，命名为nacos，导入下面的SQL：

     ```sql
     CREATE TABLE `config_info` (
       `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
       `data_id` varchar(255) NOT NULL COMMENT 'data_id',
       `group_id` varchar(255) DEFAULT NULL,
       `content` longtext NOT NULL COMMENT 'content',
       `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
       `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
       `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
       `src_user` text COMMENT 'source user',
       `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
       `app_name` varchar(128) DEFAULT NULL,
       `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
       `c_desc` varchar(256) DEFAULT NULL,
       `c_use` varchar(64) DEFAULT NULL,
       `effect` varchar(64) DEFAULT NULL,
       `type` varchar(64) DEFAULT NULL,
       `c_schema` text,
       PRIMARY KEY (`id`),
       UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';
     
     /******************************************/
     /*   数据库全名 = nacos_config   */
     /*   表名称 = config_info_aggr   */
     /******************************************/
     CREATE TABLE `config_info_aggr` (
       `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
       `data_id` varchar(255) NOT NULL COMMENT 'data_id',
       `group_id` varchar(255) NOT NULL COMMENT 'group_id',
       `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
       `content` longtext NOT NULL COMMENT '内容',
       `gmt_modified` datetime NOT NULL COMMENT '修改时间',
       `app_name` varchar(128) DEFAULT NULL,
       `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
       PRIMARY KEY (`id`),
       UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';
     
     
     /******************************************/
     /*   数据库全名 = nacos_config   */
     /*   表名称 = config_info_beta   */
     /******************************************/
     CREATE TABLE `config_info_beta` (
       `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
       `data_id` varchar(255) NOT NULL COMMENT 'data_id',
       `group_id` varchar(128) NOT NULL COMMENT 'group_id',
       `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
       `content` longtext NOT NULL COMMENT 'content',
       `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
       `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
       `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
       `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
       `src_user` text COMMENT 'source user',
       `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
       `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
       PRIMARY KEY (`id`),
       UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';
     
     /******************************************/
     /*   数据库全名 = nacos_config   */
     /*   表名称 = config_info_tag   */
     /******************************************/
     CREATE TABLE `config_info_tag` (
       `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
       `data_id` varchar(255) NOT NULL COMMENT 'data_id',
       `group_id` varchar(128) NOT NULL COMMENT 'group_id',
       `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
       `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
       `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
       `content` longtext NOT NULL COMMENT 'content',
       `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
       `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
       `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
       `src_user` text COMMENT 'source user',
       `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
       PRIMARY KEY (`id`),
       UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';
     
     /******************************************/
     /*   数据库全名 = nacos_config   */
     /*   表名称 = config_tags_relation   */
     /******************************************/
     CREATE TABLE `config_tags_relation` (
       `id` bigint(20) NOT NULL COMMENT 'id',
       `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
       `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
       `data_id` varchar(255) NOT NULL COMMENT 'data_id',
       `group_id` varchar(128) NOT NULL COMMENT 'group_id',
       `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
       `nid` bigint(20) NOT NULL AUTO_INCREMENT,
       PRIMARY KEY (`nid`),
       UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
       KEY `idx_tenant_id` (`tenant_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';
     
     /******************************************/
     /*   数据库全名 = nacos_config   */
     /*   表名称 = group_capacity   */
     /******************************************/
     CREATE TABLE `group_capacity` (
       `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
       `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
       `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
       `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
       `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
       `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
       `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
       `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
       `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
       `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
       PRIMARY KEY (`id`),
       UNIQUE KEY `uk_group_id` (`group_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';
     
     /******************************************/
     /*   数据库全名 = nacos_config   */
     /*   表名称 = his_config_info   */
     /******************************************/
     CREATE TABLE `his_config_info` (
       `id` bigint(64) unsigned NOT NULL,
       `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
       `data_id` varchar(255) NOT NULL,
       `group_id` varchar(128) NOT NULL,
       `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
       `content` longtext NOT NULL,
       `md5` varchar(32) DEFAULT NULL,
       `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
       `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
       `src_user` text,
       `src_ip` varchar(50) DEFAULT NULL,
       `op_type` char(10) DEFAULT NULL,
       `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
       PRIMARY KEY (`nid`),
       KEY `idx_gmt_create` (`gmt_create`),
       KEY `idx_gmt_modified` (`gmt_modified`),
       KEY `idx_did` (`data_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';
     
     
     /******************************************/
     /*   数据库全名 = nacos_config   */
     /*   表名称 = tenant_capacity   */
     /******************************************/
     CREATE TABLE `tenant_capacity` (
       `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
       `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
       `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
       `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
       `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
       `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
       `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
       `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
       `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
       `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
       PRIMARY KEY (`id`),
       UNIQUE KEY `uk_tenant_id` (`tenant_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';
     
     
     CREATE TABLE `tenant_info` (
       `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
       `kp` varchar(128) NOT NULL COMMENT 'kp',
       `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
       `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
       `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
       `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
       `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
       `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
       PRIMARY KEY (`id`),
       UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
       KEY `idx_tenant_id` (`tenant_id`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';
     
     CREATE TABLE `users` (
     	`username` varchar(50) NOT NULL PRIMARY KEY,
     	`password` varchar(500) NOT NULL,
     	`enabled` boolean NOT NULL
     );
     
     CREATE TABLE `roles` (
     	`username` varchar(50) NOT NULL,
     	`role` varchar(50) NOT NULL,
     	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
     );
     
     CREATE TABLE `permissions` (
         `role` varchar(50) NOT NULL,
         `resource` varchar(255) NOT NULL,
         `action` varchar(8) NOT NULL,
         UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
     );
     
     INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);
     
     INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
     ```

     

  2. 下载nacos安装包、配置nacos：

     1. 进入nacos的conf目录，修改配置文件`cluster.conf.example`，重命名为`cluster.conf`：

        <img src="Untitled.assets/image-20220925220155019.png" alt="image-20220925220155019" style="zoom:80%;" />

     2. 编辑`cluster.conf`，添加内容：

        ```
        127.0.0.1:8845
        127.0.0.1.8846
        127.0.0.1.8847
        ```

        ![image-20220925220345559](Untitled.assets/image-20220925220345559.png)

     3. 修改`application.properties`文件，添加数据库配置

        ```properties
        #*************** Config Module Related Configurations ***************#
        ### If use MySQL as datasource:
        spring.datasource.platform=mysql
        
        ### Count of DB:
        db.num=1
        
        ### Connect URL of DB:
        db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
        db.user.0=root
        db.password.0=ning
        ```

        - 打开数据源
        - 配置数据库数量
        - 配置数据库连接信息

  3. 启动nacos集群 

     1. 将nacos文件夹复制三份，分别命名为：nacos1、nacos2、nacos3

     2. 分别修改三个文件夹中的application.properties

        nacos1:

        ```properties
        server.port=8845
        ```

        nacos2:

        ```properties
        server.port=8846
        ```

        nacos3:

        ```properties
        server.port=8847
        ```

     3. 分别启动三个nacos节点：

        ```
        startup.cmd
        ```

  4. nginx反向代理

# Feign远程调用

## Feign替代RestTemplate

- `RestTemplate`方式调用存在的问题

  - 以前利用`RestTemplate`发起远程调用的代码：

    ```java
    String urL = "http: //userservice/user/" + order. getuserId() ;
    User user = restTemplate. getForobject(urL,user. cLass) ;
    ```

  - 存在的问题：

    - 代码可读性差，编程体验不统一
    - 参数复杂URL难以维护

- Feign的介绍

  - Feign是一个**声明式的http客户端**
  - 官方地址: https://qithub.com/OpenFeign/feign
  - 作用：帮助我们优雅的实现http请求的发送，解决上面提到的问题。

- **使用Feign：**

  1. 引入依赖：

     ```xml
     <!--        feign客户端依赖-->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-openfeign</artifactId>
             </dependency>
     ```

  2. 在order-service的**启动类**添加注解开启Feign的功能：

     ```java
     @EnableFeignClients
     public class OrderApplication {}
     ```

  3. 编写Feign客户端：

     ```java
     @FeignClient("userservice")
     public interface UserClient {
     //    返回值是想返回的结果，参数对应需要传入的参数
         @GetMapping("/user/{id}")
         User findById(@PathVariable("id") Long id);
     }
     ```

     - 对比`resttemplate`：

       ```java
       String urL = "http: //userservice/user/" + order. getuserId() ;
       User user = restTemplate. getForobject(urL,user. cLass) ;
       ```

     - 调用Feign客户端实现：

       ```java
           @Autowired
           private UserClient userClient;
       
           public Order queryOrderById(Long orderId) {
       //        1.查询订单
               Order order = orderMapper.findById(orderId);
               
       //        2. 用Feign进行远程调用
               User user = userClient.findById(order.getUserId());
               
       //        3.封装user到Order
               order.setUser(user);
               // 4.返回
               return order;
           }
       ```

- feign集成了ribbon，会实现**负载均衡**：

  <img src="Untitled.assets/image-20220926110044572.png" alt="image-20220926110044572" style="zoom:80%;" />

## 自定义Feign配置

- Feign运行**自定义配置来覆盖默认配置**，可以修改的配置如下：

  ![image-20220926111251840](Untitled.assets/image-20220926111251840.png)

- **修改日志级别**：

  - 方式一：配置文件方式

    1. 全局生效：

       ```yaml
       feign:
         client:
           config:
             default: #这里用default就是全局配置
               loggerLevel: FULL #日志级别
       ```

    2. 局部生效：

       ```yaml
       feign:
         client:
           config:
             userservice: #这里写服务名称，则是针对某个微服务的配置
               loggerLevel: FULL #日志级别
       ```

  - 方式二：java代码方式

    1. 需要先声明一个Bean：

       ```java
       public class DefaultFeignConfiguration {
           @Bean
           public Logger.Level logLevel(){
               return Logger.Level.BASIC;
           }
       }
       ```

    2. **全局配置**：把它放到`@EnableFeignClients`这个注解中：

       ```java
       @EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration.class)
       ```

    3. **局部配置**：把它放到`@FeignClient`这个注解中:

       ```java
       @FeignClient(value = "userservice",configuration = DefaultFeignConfiguration.class)
       ```

## Feign性能优化

- Feign**底层的客户端实现**:

  - `URLConnection`：默认实现，不支持连接池
  - `Apache HttpClient`：支持连接池
  - `OKHttp`：支持连接池

- 优化Feign的性能：

  - 日志级别，最好用basic或none
  - 使用连接池（`HttpClient`或`0KHttp`）代替默认的`URLConnection`

- Feign的性能优化 - **连接池配置**

  1. 添加`HttpClient`的支持：

     - 引入依赖：

       ```xml
       <!--        引入HttpClient依赖-->
               <dependency>
                   <groupId>io.github.openfeign</groupId>
                   <artifactId>feign-httpclient</artifactId>
               </dependency>
       ```

     - 配置连接池：

       ```yaml
       feign:
         httpclient:
           enabled: true #支持httpClient的开关
           max-connections: 200 #最大连接数
           max-connections-per-route: 50 #单个路径的最大连接数
       ```

## 最佳实践

### 方式一（继承）

- 给消费者的FeignClient和提供者的controller定义统一的父接口作为标准。

  ![image-20220927104122021](day02.assets/image-20220927104122021.png)

- 不推荐共享接口：

  - 服务紧耦合
  - 父接口参数列表中的映射不会被继承

### 方式二（抽取）

- 将**FeignClient抽取为独立模块**，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用。

  ![image-20220927103741378](day02.assets/image-20220927103741378.png)

- 步骤：

  1. 创建一个module，命名为feign-api，然后引入feign的starter依赖：

     ```xml
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-openfeign</artifactId>
     </dependency>
     ```

  2. 将order-service中编写的UserClient、User、DefaultFeignConfiguration都复制到feign-api项目中：

     <img src="day02.assets/image-20220927104916489.png" alt="image-20220927104916489" style="zoom:67%;" />

  3. 在order-service中引入feign-api的依赖：

     ```xml
     <!--            引入feign的统一api-->
     <dependency>
         <groupId>cn.itcast.demo</groupId>
         <artifactId>feign-api</artifactId>
         <version>1.0</version>
     </dependency>
     ```

  4. 修改order-service中的所有与上述三个组件有关的import部分，改成导入feign-api中的包

- 错误分析：当定义的FeignClient不在SpringBootApplication的扫描包范围时，这些FeignClient无法使用。

  1. 指定FeignClient所在包：

     ```java
     @EnableFeignClients(basePackages = "cn.itcast.feign.clients")
     ```

  2. 指定FeignClient字节码：

     ```java
     EnableFeignclients(clients = {Userclient.class})
     ```

# Gateway服务网关

## 为什么需要网关

![image-20220927110629276](day02.assets/image-20220927110629276.png)

- 网关功能：
  - 身份认证和权限校验
  - 服务路由、负载均衡
  - 请求限流
- 网关的**技术实现**
  - 在SpringCloud中网关的实现包括两种:
    - gateway：
      - 基于Spring5中提供的WebFlux，属于响应式编程的实现，具备更好的性能。
    - zuul（早期）：
      - 基于Servlet的实现，属于阻塞式编程

## gateway快速入门

### 搭建网关服务

1. 创建新的module，引入**SpringCloudGateway依赖**和**nacos服务发现依赖**：

   ```xml
   <!--        nacos服务注册发现依赖-->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>
   <!--        网关gateway依赖-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-gateway</artifactId>
           </dependency>
   ```

2. 编写路由配置及nacos地址：

   ```yml
   server:
     port: 10010
   spring:
     application:
       name: gateway
     cloud:
       nacos:
         server-addr: localhost:8848
       gateway:
         routes:
           - id: user-service #路由标示，必须唯一
             uri: lb://userservice #路由的目标地址
             predicates: #路由断言,判断请求是否符合规则
               - Path=/user/** #判断路径是否以/user开头，如果是则符合
           - id: order-service
             uri: lb://orderservice
             predicates:
               - Path=/order/**
   ```

   - **路由id**：路由的唯一标示
   - **路由目标（uri)**：路由的目标地址，http代表固定地址，lb代表根据服务名负载均衡
   - **路由断言（ predicates)** ：判断路由的规则
   - **路由过滤器（ filters)** ：对请求或响应做处理

3. 总结：

   ![image-20220928104223623](day02.assets/image-20220928104223623.png)

   

## 路由断言工厂 Route Predicate Factory

- 我们在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory**读取并处理，转变为路由判断的条件**

- 例如Path=/user/**是按照路径匹配，这个规则是由
  `org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory`类来处理的，类似的断言工厂在SpringCloudGateway还有十几个

- Spring提供了11种基本的Predicate工厂:

  ![image-20220928104854078](day02.assets/image-20220928104854078.png)

- 文档：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories

## 路由过滤器GatewayFilter

- GatewayFilter是网关中提供的一种过滤器，可以对**进入网关的请求**和**微服务返回的响应**做处理：

  ![image-20220928110335315](day02.assets/image-20220928110335315.png)

- Spring提供了31种不同的路由过滤器工厂：

  ![image-20220928110430152](day02.assets/image-20220928110430152.png)

- 文档：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories

- demo

  - 给所有进入userservice的请求添加一个请求头：`Truth=itcast is freaking awesome!`

  - 实现方式:在gateway中修改application.yml文件，给userservice的路由**添加过滤器：**

    ```yml
    spring:
      cloud:
        gateway:
          routes:
            - id: user-service
              uri: lb://userservice 
              predicates: 
                - Path=/user/** 
              filters:
                - AddRequestHeader=Truth,itcast is freaking awesome!
    ```

- **默认过滤器**：

  - 如果要**对所有的路由都生效**，则可以将过滤器工厂写到default下

    ```yml
    spring:
      cloud:
        gateway:
          routes:
            - id: user-service 
              uri: lb://userservice 
              predicates: 
                - Path=/user/** 
          default-filters:
            - AddRequestHeader=Truth,itcast is freaking awesome!
    ```

## 全局过滤器GlobalFilter

- 全局过滤器的作用也是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样

- 区别：

  - GatewayFilter通过**配置定义**，处理逻辑是固定的。
  - GlobalFilter的逻辑需要自己**写代码实现**。

- 定义方式：

  - 实现GlobalFilter接口：

    ![image-20220928145728075](day02.assets/image-20220928145728075.png)

- demo

  - 定义全局过滤器，拦截并判断用户身份，判断请求的参数是否满足下面条件：

    - 参数中是否有authorization
    - authorization参数值是否为admin

  - 实现：

    ```java
    //@Order(-1)//越小过滤器的优先级越高(方式一)
    @Component
    public class AuthorizeFilter implements GlobalFilter, Ordered {
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    //        1.获取请求参数
            ServerHttpRequest request = exchange.getRequest();
            MultiValueMap<String, String> params = request.getQueryParams();
    //        2.获取参数中的authorization参数
            List<String> auth = params.get("authorization");
    //        3.判断参数值是否等于admin
            if ("admin".equals(auth)) {
    //        4.是,放行
                return chain.filter(exchange);
            }else
    //        5.否,拦截
    //        5.1 设置状态码
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
    //        5.2 拦截请求
            return exchange.getResponse().setComplete();
        }
    
    //    方式二
        @Override
        public int getOrder() {
            return -1;
        }
    }
    ```

  - 实现GlobalFilter接口

  - 添加@0rder注解或实现Ordered接口（定义过滤器顺序）

  - 编写处理逻辑

## 过滤器执行顺序

- 请求进入网关会碰到三类过滤器：

  - 当前路由的过滤器
  - DefaultFilter
  - GlobalFilter

- 请求路由后，会将三类过滤器合并到一个**过滤器链（集合）**中，排序后依次执行每个过滤器

  ![image-20220928152818457](day02.assets/image-20220928152818457.png)

- 每个过滤器都必须指定一个int类型的order值，**order值越小，优先级越高，执行顺序越靠前。**

  - GlobalFilter由我们自己指定
  - 路由过滤器和defaultFilter的order由Spring指定，默认是**按照声明顺序从1递增**。
    - 写得越前，优先级越高
    - 当过滤器的order值一样时，会按照**defaultFilter >路由过滤器>GlobalFilter**的顺序执行

## 跨域问题

- 跨域：域名不一致就是跨域，主要包括

  - 域名不同： www.taobao.com和www.taobao.org和www.jd.com和miaosha.jd.com
  - 域名相同，端口不同：`localhost:8080`和`localhost8081`

- 跨域问题：**浏览器禁止**请求的发起者与服务端发生**跨域ajax请求**，请求被浏览器拦截的问题

- 解决方案：CORS

- 网关处理跨域采用的同样是CORS方案，并且只需要简单配置即可实现：

  ```yml
  spring:
    cloud:
      gateway:
        globalcors: # 全局的跨域处理
          add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
          corsConfigurations:
            '[/**]':
              allowedOrigins: # 允许哪些网站的跨域请求
                - "http://localhost:8090"
                - "http://www.leyou.com"
              allowedMethods: # 允许的跨域ajax的请求方式
                - "GET"
                - "POST"
                - "DELETE"
                - "PUT"
                - "OPTIONS"
              allowedHeaders: "*" # 允许在请求中携带的头信息
              allowCredentials: true # 是否允许携带cookie
              maxAge: 360000 # 这次跨域检测的有效期
  ```