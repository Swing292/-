[TOC]

# NoSql数据库

1. 技术分类：

   ![image-20220523103724839](Redis.assets/image-20220523103724839.png)

2. Redis是一种NoSql数据库

3. 好处：

   1. 解决CPU和内存压力

      ![image-20220523104418596](Redis.assets/image-20220523104418596.png)

   2. 解决IO压力

4. NoSQL（NoSQL = Not Only sQL )，意即“不仅仅是SQL”，泛指非关系型的数据库。

5. NoSQL不依赖业务逻辑方式存储，而以简单的 key-value模式存储。因此大大的增加了数据库的扩展能力。

6. 特点：

   1. 不遵循Sql标准
   2. 不支持ACID
   3. 远超于Sql的性能

7. 适用场景

   1. 对数据高并发的读写
   2. 海量数据的读写
   3. 对数据高可扩展性的

8. 不适用的支持

   1. 需要事务的支持
   2. 基于sql的结构化查询存储，处理复杂的关系，需要即席查询
   3. 用不着sql和用了sql也不行的情况下，考虑用NoSql

9. NoSql数据库

   1. Memcache
      1. 不支持持久化
      2. 支持类型单一
      3. 一般作为缓存数据库辅助持久化的数据库
   2. Redis
      1. 覆盖了Memcache的绝大部分功能
      2. 支持持久化
      3. 支持多种数据类型
      4. 一般作为缓存数据库辅助持久化的数据库
   3. MongoDB
      1. 文档型数据库
      2. 对value提供了丰富的查询功能
      3. 支持二进制数据及大型对象

# Redis安装

1. 将redis-6.2.7.tar.gz上传到Linux

2. 解压：![image-20220523161236812](Redis.assets/image-20220523161236812.png)

3. 下载gcc：

   ![image-20220523161350136](Redis.assets/image-20220523161350136.png)

   测试安装：

   ![image-20220523161428975](Redis.assets/image-20220523161428975.png)

4. 进入解压后的安装包，编译文件：

   ![image-20220523161534392](Redis.assets/image-20220523161534392.png)

5. 开始安装：

   ![image-20220523161802744](Redis.assets/image-20220523161802744.png)

6. 运行：

   1. 前台运行：

      ![image-20220523161932193](Redis.assets/image-20220523161932193.png)

   2. 后台运行：

      1. 复制redis.conf到/etc下：

         ![image-20220523162051576](Redis.assets/image-20220523162051576.png)

      2. 编辑/etc下的redis.conf文件：`vi redis.conf `，将daemonize no改成yes（/用于搜索）

      3. 进入/usr/local/bin，启动：

         ![image-20220523162319896](Redis.assets/image-20220523162319896.png)

   3. 关闭：

      1. ![image-20220523162425056](Redis.assets/image-20220523162425056.png)
      2. `redis-cli shutdown`

