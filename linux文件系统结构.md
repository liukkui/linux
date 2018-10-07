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


## 6.文件的三个时间戳和通配符
access time：访问时间，atime，读取文件内容
一般为了提升系统性能可以屏蔽
modify time: 修改时间, mtime，改变文件内容（数据）
change time: 改变时间, ctime，Metedata元数据发生改变
查看文件的3个时间命令：stat+file

通配符：
* 匹配零个或多个字符
? 匹配任何单个字符
~ 当前用户家目录
~test 用户test家目录
~+ 当前工作目录
~- 前一个工作目录
[0-9] 匹配数字范围
[a-z]：字母
[A-Z]：字母
[test] 匹配列表中的任何的一个字符
[^test] 匹配列表中的所有字符以外的字符
预定义的字符类：man 7 glob
[:digit:]：任意数字，相当于0-9
[:lower:]：任意小写字母
[:upper:]: 任意大写字母
[:alpha:]: 任意大小写字母
[:alnum:]：任意数字或字母
[:blank:]：水平空白字符
[:space:]：水平或垂直空白字符
[:punct:]：标点符号
[:print:]：可打印字符
[:cntrl:]：控制（非打印）字符
[:graph:]：图形字符
[:xdigit:]：十六进制字符

# ls:列出当前目录的内容或指定目录
用法：ls [options] [files_or_dirs] 
示例: 
ls -a 包含隐藏文件 
ls -l 显示额外的信息 
ls -R 目录递归通过 
ls -ld 目录和符号链接信息 
ls -1 文件分行显示 
ls –S 按从大到小排序 
ls –t 按mtime排序 
ls –u 配合-t选项，显示并按atime从新到旧排序 
ls –U 按目录存放顺序显示 
ls –X 按文件后缀排序

范例：
1.显示/var目录下所有以l开头，以一个小写字母结尾，且中间出现至少一位数字的文件或目录。
[root@centos7 ~]#ls /var/l*[0-9]*[[:lower:]]
2.显示/etc目录下以任意一位数字开头，且以非数字结尾的文件或目录。
[root@centos7 ~]#ls /etc/[0-9]*[^0-9]
3.只显示/root下的隐藏文件和目录。 只显示/etc下的非隐藏目录
[root@centos7 ~]#ls -d /root/.* ls /etc/[^.]*/ -d
ls: cannot access ls: No such file or directory
/etc/abrt/               /etc/gnupg/           /etc/pki/             /etc/speech-dispatcher/
/etc/alsa/               /etc/groff/           /etc/plymouth/       
4.显示/etc/目录下以非字母开头，后面跟了一个字母及其他任意长度任意字符的文件或目录。
[root@centos7 ~]#ls /etc/[^[:alpha:]][a-zA-Z]*



# Shell
在shell中可执行的命令有两类 
内部命令：由shell自带的，而且通过某命令形式提供 
help 显示所有内部命令列表 
enable cmd 启用内部命令 
enable –n cmd 禁用内部命令 
enable –n 查看所有禁用的内部命令 
外部命令：在文件系统路径下有对应的可执行程序文件 
查看路径：which -a |--skip-alias ; whereis
区别指定的命令是内部或外部命令 type COMMAND


# Hash缓存表 
系统初始hash表为空，当外部命令执行时，默认会从PATH路径下寻找该命 令，找到后会将这条命令的路径记录到hash表中，当再次使用该命令时，shell解 释器首先会查看hash表，存在将执行之，如果不存在，将会去PATH路径下寻找。 利用hash缓存表可大大提高命令的调用速率 
hash常见用法 
hash 显示
hash缓存 
hash –l 显示hash缓存，可作为输入使用 
hash –p path name 将命令全路径path起别名为name 
hash –t name 打印缓存中name的路径 
hash –d name 清除name缓存 hash –r 清除缓存

显示当前shell进程所有可用的命令别名 
# alias 
定义别名NAME，其相当于执行命令VALUE alias NAME='VALUE' 
用法: unalias unalias [-a] name [name ...] 
-a 取消所有别名
如果别名同原命令同名，如果要执行原命令，可使用 
\ALIASNAME 
“ALIASNAME” 
‘ALIASNAME’ 
command ALIASNAME 
/path/command

在命令行中定义的别名，仅对当前shell进程有效 
如果想永久有效，要定义在配置文件中 
仅对当前用户：~/.bashrc 
对所有用户有效：/etc/bashrc

linux系统中命令的工作原理：alias（查看是否为别名）--- 内部（是否为内部命令）---hash表（记录外部命令的路径）---$PATH ---命令找不到

缓存cache：将刚用硬盘的数据放在内存中，下次用此数据，不需要从硬盘找，直接内存取出hash，可以提升系统性能。


# echo命令 
功能：显示字符 
语法：echo [-neE][字符串] 
说明：echo会将输入的字符串送往标准输出。输出的字符串间以空白字符隔开, 并在最后加上换行号 
选项：  
-E （默认）不支持 \ 解释功能 
-n 不自动换行 
-e 启用 \ 字符的解释功能
启用命令选项-e，若字符串中出现以下字符，则特别加以处理，而不会将它当成 一般文字输出 
\a 发出警告声 
\b 退格键 
\c 最后不加上换行符号 
\n 换行且光标移至行首 
\r 回车，即光标移至行首，但不换行 
\t 插入tab 
\\ 插入
\字符 
\0nnn 插入nnn（八进制）所代表的ASCII字符 
echo -e '\033[43;31;5mmagedu\033[0m' 
\xHH插入HH（十六进制）所代表的ASCII数字（man 7 ascii）

''，""
用于用户把带有空格的字符串赋值给变量事的分界符；
如果没有单引号或双引号，shell会把空格后的字符串解释为命令；
区别：
   单引号告诉shell忽略所有特殊字符，而双引号忽略大多数，但不包括$、\、`。
   ''弱引用，可以实现变量和命令的替换。 在双引号中可以使用变量，用$COMMAND。
   ""强引用，不完成变量替换 在单引号中不能使用任何变量和命令。
``（反引号ESC下的按键）起着命令替换的作用。命令替换是指shell能够将一个命令的标准输出插在一个命令行中任何位置。
$()：把一个命令的输出打印给另一个命令的参数
{}：打印重复字符串的简化形式
[]的 用法
例如：
输出$PATH和主机名hostname
[root@centos7 data]#echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@centos7 data]#echo 'echo $PATH'
echo $PATH
[root@centos7 data]#echo "echo $PATH"             -只显示带$的变量信息
echo /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@centos7 data]#echo `echo $PATH`            -输出echo $PATH命令
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@centos7 data]#echo $(echo $PATH)              $()=``
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@centos7 data]#echo "my hostname is `hostname`" -显示反引号的信息
my hostname is centos7.localdomain


# date&clock
Linux的两种时钟
系统时钟：由Linux内核通过CPU的工作频率进行的
硬件时钟：主板
相关命令
date 显示和设置系统时间
date +%s
date -d @1509536033
hwclock，clock: 显示硬件时钟
-s, --hctosys 以硬件时钟为准，校正系统时钟
-w, --systohc 以系统时钟为准，校正硬件时钟
时区：/etc/localtime
显示日历：cal –y 

eg:
显示当前时间，格式：2018-09-23 10:20:30
[root@centos7 ~]#date '+%F %R'
2018-09-23 21:45
[root@centos7 ~]#date +'%F %R'
2018-09-23 21:45
[root@centos7 ~]#date '+%F %R:%S'
2018-09-23 21:45:55

显示前天是星期几
[root@centos7 ~]#date -d '-2 day' +%A
Friday

# shutdown poweroff halt init 0 关机区别
关机：halt, poweroff
重启：reboot
-f: 强制，不调用shutdown
-p: 切断电源
关机或重启：shutdown
shutdown [OPTION]... [TIME] [MESSAGE]
-r: reboot
-h: halt
-c：cancel
TIME：无指定，默认相当于+1（CentOS7）
now: 立刻,相当于+0
+m: 相对时间表示法，几分钟之后；例如 +3
hh:mm: 绝对时间表示，指明具体时间
eg:
今天20:00自动关机，并提示用户
shutdown -h 20:00 "wanshang 8dian guanji"



