[TOC]



# 1 认识Linux

1. 特点：

   - 免费使用、自由传播
   - 类Unix操作系统
   - 基于POSIX和UNIX
   - 继承了Unix以网络为核心的设计思想

2. 诞生与发展：

   - **UNIX** **操作系统**、**MINIX** **操作系统**、**GNU** **计划**、**POSIX** **标准**、**Internet** **网络**
   - 基于POSIX和UNIX的多用户、多任务、支持多线程和多CPU的操作系统。
   - UNIX：世界上第一个完善的网络操作系统

3. 组成：

   <img src="Linux.assets/image-20211207113504322.png" alt="image-20211207113504322" style="zoom: 33%;" />

   - Linux内核：
     - 是操作系统的核心
     - 主要模块分为**存储管理**、**CPU和进程管理**、**文件系统**、**设备管理和驱动**、**网络通信**、**系统的初始化**和**系统调用**等几个部分。
   - Shell
     - 系统的**用户界面**，提供了**用户**与**内核**进行交互操作的一种接口。
     - 是一个**命令解释器**。
     - 可以作为编程语言。
   - 文件系统
     - 文件存放在磁盘等存储设备上的组织方法。
     -  **Linux系统能支持多种目前流行的文件系统**，如ext3、ext4、XFS、FAT、VFAT、NTFS和ISO9660等**。** 
   - 应用程序
     - 有一套称为**应用程序**的程序集，它包括**文本编辑器**、**编程语言**、**X Window**、**办公软件**和**Internet工具**等。

4. Linux版本

   - 内核版本：
     - 内核版本号由形如 x1.x2.x3这三组数字组成，例如：3.10.0-327、4.4.3-1、5.3.6-1等等。
   - 发行版本：
     - 基于Linux内核的图形界面，同时配上很多功能强大的应用软件。
     - <img src="Linux.assets/image-20211207114507539.png" alt="image-20211207114507539" style="zoom:50%;" />

5. 获取途径：

   - Red Hat Linux：http://www.redhat.com
   - Fedora Linux： http://www.fedoraproject.org
   - CentOS Linux: http://www.CentOS.org
   - Debian Linux： http://www.debian.org
   - Ubuntu Linux： http://www.Ubuntu.com
   - SuSE Linux： http://www.SuSE.com
   - 红旗Linux：  http://www.redflag-linux.com
   - 中软麒麟：   http://www.cs2c.com.cn
   - 深度操作系统：  https://www.deepin.org
   - 华为云镜像：  http://mirrors.huaweicloud.com
   - 阿里云镜像：  http://mirrors.aliyun.com

6. 基本命令

   1. **halt命令**：关闭或重启系统，执行过程中，终止所有应用和系统进程，将所有数据写入存储介质，最后关闭系统。

   2. **reboot命令**：工作过程与halt几乎一样，但其在关闭系统时会重新启动它。

   3. **poweroff命令**：等价于halt -p，关闭系统时同时关掉电源。

   4. **shutdown命令**：

      - 安全地关闭系统，并在真正执行系统关闭与命令发出之间可以指定一个时间延迟，以供用户做好准备并从容退出。
      - mshutdown [-krhfFc] time [wall mesg]
      - ![image-20211207133012881](Linux.assets/image-20211207133012881.png)
      - 若不带任何选项（比如，-h、-H、-r）执行shutdown，默认切换到运行级1，即单用户模式，而不是关闭系统。
      - 例子：
        -  shutdown -r now  #立刻重新启动
        -  shutdown -h now  #立刻关机
        -  shutdown -k now 'Hey Lets go now.' #发出警告信息，但不真关机
        -  shutdown -h 10:42 "系统将在10:42关闭, 请届时退出" #10:42分关机
        -  shutdown -r +20 '20分钟后将重启系统，请提前退出'  #20分钟后重启系统
        -  shutdown -c  #撤销已下达的shutdown命令
        - shutdown now  #切换至单用户模式

   5. 系统的运行级别：

      | 运行级别 | 描述                                                         |
      | -------- | ------------------------------------------------------------ |
      | 0        | 关闭系统                                                     |
      | 1、s、S  | 单用户模式或系统维护模式                                     |
      | 2        | 多用户模式，没有NFS功能，没有图形界面                        |
      | 3        | 完全多用户模式，没有图形界面（无图形界面系统的默认运行级）   |
      | 4        | 没有使用，用户可自定义                                       |
      | 5        | 完全多用户模式，且支持X-Windows（桌面和工作站系统默认运行级） |
      | 6        | 重新启动                                                     |
      | Q、q     | 重新加载配置文件                                             |

      查询：who -r 或runlevel

      ![image-20211207133958190](Linux.assets/image-20211207133958190.png)

   6. 运行级别切换：

      - 系统内运行着一个叫init的进程，此init在引入systemd软件包之后叫systemd，它负责系统的初始化和运行级别的切换。

      - 命令： init LEVEL 或 telinit LEVEL（LEVEL为数字，7个运行级别中的一个）

      - 例子：

        ​	#init 0 #关机

        ​	 init 6 #重新启动

        ​     telinit 1 #切换到单用户

   7. **在线帮助与资源**（使用说明文档）

      1. man
      2. textinfo
      3. yelp
      4. 其他

