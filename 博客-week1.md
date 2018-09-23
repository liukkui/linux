# Linux文件系统结构&基本命令

## 1.文件系统
>文件和目录被组织成一个单根倒置树结构  
文件系统从根目录下开始，用“/”表示  
根文件系统(rootfs)：root filesystem  
文件名称区分大小写  
以.开头的文件为隐藏文件  
路径分隔的 /  
文件包括两类数据:
1. 元数据：metadata
2. 数据：data  

>文件系统分层结构：LSB Linux Standard Base  
&ensp;&ensp;是一个在Linux基金会结构下对Linux发行版的联合项目,使Linux操作系统符合软件系统架构,或文件系统架构标准的规范及标准。LSB基于POSIX,统一UNIX规范及其他开放标准,共在某些领域扩展它们。
FHS: (Filesystem Hierarchy Standard)  
&ensp;&ensp;文件系统层次结构标准（Filesystem Hierarchy Standard，FHS）定义了Linux操作系统中的主要目录及目录内容。 在FHS中，所有的文件和目录都出现在根目录"/"下，即使他们存储在不同的物理设备中。

## 2.文件系统结构
>/boot：引导文件存放目录，内核文件(vmlinuz)、引导加载器(bootloader,grub)都存放于此目录   
/bin：供所有用户使用的基本命令；不能关联至独立分区，OS启动即会用到的 程序   
/sbin：超级用户使用的命令目录（root）管理类的基本命令；不能关联至独立分区，OS启动即会用到的程序   
/lib：启动时程序依赖的基本共享库文件以及内核模块文件(/lib/modules)  
/lib64：专用于x86_64系统上的辅助共享库文件存放位置   
/etc：配置文件目录 eg: /etc/rc.d 启动的配置文件和脚本  
/home/USERNAME：普通用户家目录   
/root：管理员的家目录   
/media：便携式移动设备挂载点  
/mnt：临时文件系统挂载点  
/dev：设备文件及特殊文件存储位置 b: block device，随机访问 c: character device，线性访问 如硬盘：/dev/sda dev/sdb /dev/sr0  
/opt：第三方应用程序的安装位置   
/srv：系统上运行的服务用到的数据   
/tmp：临时文件存储位置(重启清空文件夹)  
/usr: universal shared, read-only data bin: 保证系统拥有完整功能而提供的应用程序
>>sbin: lib：32位使用   
>lib64：只存在64位系统  
>include: C程序的头文件(header files)   
>share：结构化独立的数据，例如doc, man等   
>local：第三方应用程序的安装位置 bin, sbin, lib, lib64, etc, share  

>/var variable data files （某些大文件的溢出区，比方说各种服务的日志文件）  
>>cache: 应用程序缓存数据目录   
>lib: 应用程序状态信息数据   
>local：专用于为/usr/local下的应用程序存储可变数据；   
>lock: 锁文件   
>log: 日志目录及文件   
>opt: 专用于为/opt下的应用程序存储可变数据；   
>run: 运行中的进程相关数据,通常用于存储进程pid文件  
>spool: 应用程序数据池   
>tmp: 保存系统两次重启之间产生的临时数据   

/proc: 用于输出内核与进程信息相关的虚拟文件系统   
/sys：用于输出当前系统上硬件设备相关信息虚拟文件系统   
/selinux: security enhanced Linux，selinux相关的安全策略等信息的存储位置

## 3.Linux上的应用程序的组成部分
二进制程序：/bin, /sbin, /usr/bin, /usr/sbin, /usr/local/bin, /usr/local/sbin  
库文件：/lib, /lib64, /usr/lib, /usr/lib64, /usr/local/lib, /usr/local/lib64  
配置文件：/etc, /etc/DIRECTORY, /usr/local/etc  
帮助文件：/usr/share/man, /usr/share/doc, /usr/local/share/man,
/usr/local/share/doc
## 4.Linux下的文件类型
-：普通文件  
d: 目录文件  
b: 块设备  
c: 字符设备  
l: 符号链接文件：又区分硬链接和软连接  
p: 管道文件pipe  
s: 套接字文件socket  
## 5.文件命令规则
文件名最长255个字节  
包括路径在内文件名称最长4095个字节  
蓝色-->目录 绿色-->可执行文件 红色-->压缩文件 浅蓝色-->链接文
件 灰色-->其他文件  
除了斜杠和NUL,所有字符都有效.但使用特殊字符的目录名和文件不推荐使用，
有些字符需要用引号来引用它们  
标准Linux文件系统（如ext4），文件名称大小写敏感
例如：MAIL, Mail, mail, mAiL  
eg：创建/删除一个‘-a‘文件夹  
w1:touch -- -a/rm -- -a  
w2:用绝对路径和相对路径的表示方法  
在/data路径下：   touch ./-a | rm ./-a  
相对路径表示方法：touch /data/-a | rm /data –a  
取基名：basename   
取目录名：dirname  
eg: 
```bash
[root@centos7 ~]#basename /etc/sysconfig/network-scripts/
network-scripts
[root@centos7 ~]#dirname /etc/sysconfig/network-scripts/
/etc/sysconfig


