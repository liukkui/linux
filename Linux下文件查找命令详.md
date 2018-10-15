# Linux下文件查找的工具详解：
二者区别

>locate：  
 >1) 非实时查找；  
 2) 依赖于索引，而索引构建非常占用资源，索引的创建是在系统空闲时系统自动进行，可以用updatedb命令更新索引(/var/lib/mlocate/mlocate.db)，极度耗费系统资源；  
 3) 查找速度快；              
 4) 非精准查找(会匹配到文件路径)。  
find :    
1) 实时查找(系统上实时的有就是有没有就是没有)；          
 2) 查找速度慢  
 3) 精准查找 (查找结果绝对符合查找条件才予显示)；  


## ***文件查找之-Locate:***

---查找功能较弱，一般用find进行查找  
在文件系统上查找符合条件的文件  
文件查找：locate, find  
非实时查找(数据库查找)：locate  
实时查找：find  

查询系统上预建的文件索引数据库  
/var/lib/mlocate/mlocate.db  
依赖于事先构建的索引  
索引的构建是在系统较为空闲时自动进行(周期性任务)，管理员手动更新数据库(updatedb)  
索引构建过程需要遍历整个根文件系统，极消耗资源  
工作特点:  
>• 查找速度快  
• 模糊查找  
• 非实时查找  
• 搜索的是文件的全路径，不仅仅是文件名  
• 可能只搜索用户具备读取和执行权限的目录  

locate KEYWORD  
有用的选项  
>-i 不区分大小写的搜索  
-n N 只列举前N个匹配项目  
-r 使用正则表达式  

示例  
搜索名称或路径中带有“conf”的文件  
locate conf  
使用Regex来搜索以“.conf”结尾的文件  
locate -r ‘\.conf$’  

================================================


## ***文件查找之-find***
实时查找工具，通过遍历指定路径完成文件查找  
工作特点：  
>• 查找速度略慢  
• 精确查找  
• 实时查找  
• 可能只搜索用户具备读取和执行权限的目录  
面试：--搜索一个文件，然后进行处理。  

语法：  
>find [OPTION]... [查找路径] [查找条件] [处理动作]  
>>查找路径：指定具体目标路径；默认为当前目录；也可指定路径  
查找条件：指定的查找标准，可以文件名、大小、类型、权限等标准进行；  
默认为找出指定路径下的所有文件  
处理动作：对符合条件的文件做操作，默认输出至屏幕  

1.单独使用find,默认把当前目录所有文件列出来。  
2.递归搜索  

查找条件  
>指搜索层级  
>>-maxdepth level 最大搜索目录深度,指定目录为第1级  
-mindepth level 最小搜索目录深度  
先处理目录内的文件，再处理目录  
-depth --先处理文件，再显示目录  

>根据文件名和inode查找：  
>>-name "文件名称"：支持使用通配符glob *, ?, [], [^]  
-iname "文件名称"：不区分字母大小写  
-inum n 按inode号查找  
-samefile name 相同inode号的文件  
-links n 链接数为n的文件  
-regex “PATTERN”：以PATTERN匹配整个文件路径，而非文件名称  

1.find /data -maxdepth 1 只搜索显示当前目录下的文件，不进去子目录  
2.find /data -maxdepth 2 -mindepth 2  值搜索列出第二层目录下的所有文件  
3.find /data -name "*test*"  搜索包含test字符的文件或目录-需要加""或者''  
  find /data/ -name "*.sh"  搜索是后缀的文件  
4.find /data -samefile /data/f1 索同目录的硬链接文件--搜索的文件要写绝对路径  
5.find /data/ -regex '.*\.sh$' 通过正则表达式搜索--后面要跟完整的路径  


查找条件  
>根据属主、属组查找：  
>>-user USERNAME：查找属主为指定用户(UID)的文件  
-group GRPNAME: 查找属组为指定组(GID)的文件  
-uid UserID：查找属主为指定的UID号的文件  
-gid GroupID：查找属组为指定的GID号的文件  
-nouser：查找没有属主的文件  
-nogroup：查找没有属组的文件  

1.find /data/ -user ceshi -ls  搜索所有者是测试的文件,并列出来  
2.find /data -nouser ---由于用户被删除了，变成没有归属的文件，需要定时清理  


根据文件类型查找：  
-type TYPE:  
>• f: 普通文件  
• d: 目录文件  
• l: 符号链接文件  
• s：套接字文件  
• b: 块设备文件  
• c: 字符设备文件  
• p: 管道文件  
空文件或目录  
-empty  
find /app -type d -empty  

1.find /data -type f/d/l 等  
2.find /dev -type -b  搜索/dev下的块设备  
3.find /data/ -type d -empty 搜索空目录、空文件 -type f  

>组合条件：  
>>与：-a  
或：-o  
非：-not, !  

>德·摩根定律：  
(非 A) 或 (非 B) = 非(A 且 B)==A和B交集之外的  
(非 A) 且 (非 B) = 非(A 或 B)==A和B并集之外的  
示例：  
!A -a !B = !(A -o B)  
!A -o !B = !(A -a B)  
不搜索用prune，剪切掉  

1.find /data/ -name "*.sh" -a -user root 搜索是sh后缀并且所有者是root的文件  
2.find /data/ -name "*.sh" -o -user root 搜索sh后缀或者是所有者是root的文件  
3.find /data/ -name "*.sh" -o -user root -ls 只会列出所有者是root的文件，会把-user root   -ls 当成一个整体，所以要加括号、  
注意：一个命令作用在多个条件时，要加括号，否则命令将默认只作用在相邻的条件上  

4.find /data/ \(-name "*.sh" -o -user root\) -ls 当成一个整体列出，括号要转义  
5.find /data/  ! \(-name "*.sh" -user root\)=== ! -name "*.sh" -o ! -user root  
不是sh后缀的文件或者所有者不是root的文件  
6.find /data/ ! \(-name "*.sh" -o -user root\)=== ! -name "*.sh" -a !-user root  
   不是sh后缀并且所有者不是root的文件  
7.找出/tmp目录下，属主不是root，且文件名不以f开头的文件  
  find /tmp ! \( -user root -o -name "f*" \) -ls 前后空格隔开  
  find /tmp ! -user root -a ! -name "f*"  
8.查找/etc/下，除/etc/sane.d目录的其它所有.conf后缀的文件  
  find /etc -path "/etc/sane.d" -a -prune -o -name "*.conf"   
  --- -path "/etc/sane.d" -a -prune把该目录过滤掉  
9.查找/etc/下，除/etc/sane.d和/etc/fonts两个目录的所有.conf后缀的文件  
  find /etc \( -path "/etc/sane.d" -o -path "/etc/fonts" \) -a -prune -o -name "*.conf"  

>根据文件大小来查找：  
>>-size [+|-]#UNIT  
常用单位：k, M, G，c（byte）  
#UNIT: (#-1, #]  
如：6k 表示(5k,6k]  
-#UNIT：[0,#-1]  
如：-6k 表示[0,5k]  
+#UNIT：(#,∞)  
如：+6k 表示(6k,∞)  

1.find /data -size 10M--表示搜索的是（9M，10M]之间的文件  
2.find /data -size -11M ---表示10M以内的文件 需要减1  
3.find /data -size +5M -size -10M ---表示(5M,10M]之前的文件---实际上是5M-9M  

>根据时间戳：  
>>以“天”为单位  
-atime [+|-]#,  
#: [#,#+1)  
+#: [#+1,∞]  
-#: [0,#)   
-mtime  
-ctime  
以“分钟”为单位  
-amin  
-mmin  
-cmin  

1.find /etc -mtime 10 ---表示10-11天修改过的文件  
2.find /etc -mmin -1 ---最近一分钟修改过的文件  
  [root@centos7 data]#useradd user100  
  [root@centos7 data]#find /etc/ -mmin -1  
  /etc/  
  /etc/group  
  /etc/gshadow  
  /etc/passwd  
  /etc/shadow  
 

### ***根据权限查找：*** 
>-perm [/|-]MODE  
>>MODE: 精确权限匹配  
/MODE：任何一类(u,g,o)对象的权限中只要能一位匹配即可，或关系，+  
从centos7开始淘汰  
-MODE：每一类对象都必须同时拥有指定权限，与关系  
0 表示不关注  
• find -perm 755 会匹配权限模式恰好是755的文件  
• 只要当任意人有写权限时，find -perm +222就会匹配  
• 只有当每个人都有写权限时，find -perm -222才会匹配  
• 只有当其它人（other）有写权限时，find -perm -002才会匹配  

1.find /data -perm 644精确查找权限时644的文件（包括目录）  
2.find /data/ -type f/d -perm 644 -ls 精确查找权限是644的文件或目录  
3.find /data/ -maxdepth 1 -perm /644 查找深度为1，下的只要拥有644其中之一的权限的文件或目录  
4.find /data/ -maxdepth 1 -perm -644 查找深度为1，下的比644权限大的文件或目录，不能比644低  
5.•只要当任意人有写权限时，find -perm +222就会匹配  
  • 只有当每个人都有写权限时，find -perm -222才会匹配  
  • 只有当其它人（other）有写权限时，find -perm -002才会匹配  

>处理动作  
>>-print：默认的处理动作，显示至屏幕--默认行为可以不写  
-ls：类似于对查找到的文件执行“ls -l”命令==ll  
-delete：删除查找到的文件---危险操作！  
-fls file：查找到的所有文件的长格式信息保存至指定文件中  
-ok COMMAND {} \; 对查找到的每个文件执行由COMMAND指定的命令，对于每个文件执行命令之前，都会交互式要求用户确认  
-exec COMMAND {} \; 对查找到的每个文件执行由COMMAND指定的命令  
{}: 用于引用查找到的文件名称自身  
find传递查找到的文件至后面指定的命令时，查找到所有符合条件的文件一次性  
传递给后面的命令  

1.-fls file ==  -ls > file 重定向到文件  
2.find /data/bin -perm -644 -name "*.sh" -ok cp {} /data/bin2/ \;  
  找到文件是sh后缀并且权限时644的文件，复制一份到bin2目录下，但是每次都需要确定是否复制  
  ！！如果复制并改名，是不可以加路径的，会报路径找不到提示错误！！  
3.find /data/bin -perm -644 -name "*.sh" -exec cp {} /data/bin2/ \;  
  找到文件后复制一份到其他路径，不需要一个个确定是否复制，还可以改名  
4.面试题：在/data目录下找到大于10M的文件，并删除  
  find /data/ -size +10M -exec rm {} \;  

### 参数替换xargs
>用法：由于很多命令不支持管道|来传递参数，而日常工作中有这个必要，所以就有了xargs命令  
xargs用于产生某个命令的参数，xargs 可以读入 stdin 的数据，并且以空格符  
或回车符将 stdin 的数据分隔成为arguments  
注意：文件名或者是其他意义的名词内含有空格符的情况  
有些命令不能接受过多参数，命令执行可能会失败，xargs可以解决  

示例：  
ls f* |xargs rm  
find /sbin -perm +700 |ls -l 这个命令是错误的  
find /sbin -perm +7000 | xargs ls –l 查找特殊权限的文件  
find和xargs格式：find | xargs COMMAND  

1.touch {1.50000}命令不支持批量个文件的，（echo可以），所以可以通过xargs传参数给touch.  
  echo f{1.500000} | xargs touch  echo生成的结果，不是作为touch的输入，而是作为touch的参数  
2.find /sbin -perm +7000 | xargs ls –l 查找特殊权限的文件  
3.备份配置文件，添加.orig这个扩展名  
  find -name “*.conf” -exec cp {} {}.orig \;  
4.提示删除存在时间超过３天以上的joe的临时文件  
  find /tmp -ctime +3 -user joe -ok rm {} \;  
5.在主目录中寻找可被其它用户写入的文件并去掉写权限  
  find ~ -perm -002 -exec chmod o-w {} \;  
6.查找/data下的权限为644，后缀为sh的普通文件，增加执行权限  
  find /data –type f -perm 644 -name “*.sh” –exec chmod 755 {} \;  
7.查看/home的目录  
  find /home –type d -ls  

practice:  
1、查找/var目录下属主为root，且属组为mail的所有文件  
  find /var -user root -a -group mail  
2、查找/var目录下不属于root、lp、gdm的所有文件  
  find /var ! \( -user root -o -user lp -o -user gdm \) |wc -l  
  find /var ! -user root -a ! -user lp -a ! -user gdm  
3、查找/var目录下最近一周内其内容修改过，同时属主不为root，也不是postfix的文件  
  find /var -mtime -7 ! -user root -a ! -user postfix  
  find /var -mtime -7 ! \( -user root -o -user postfix \）  
4、查找当前系统上没有属主或属组，且最近一个周内曾被访问过的文件  
  find / -atime -7 -nouser -o -nogroup  
  find / -atime -7 \( -nouser -o -nogroup \)  
5、查找/etc目录下大于1M且类型为普通文件的所有文件  
  find /etc -size +1M -type f  
6、查找/etc目录下所有用户都没有写权限的文件  
  find /etc ! -perm /222  
7、查找/etc目录下至少有一类用户没有执行权限的文件  
  find /etc/ ! -perm -11  
8、查找/etc/init.d目录下，所有用户都有执行权限，且其它用户有写权限的文件  
  find /etc/init.d -perm -111 -a -perm -002  
  find /etc/init.d -perm -113  


要注意的点：  

注意：排除目录查找  
查找q目录下的所有.txt文件，但是跳过子目录e  
```bash
[root@localhost q]# ls
e  w
[root@localhost q]# cd e
[root@localhost e]# ls
e.txt
[root@localhost q]# find . -path './e' -a -prune -o -name *.txt -print
./w/w.txt
./e
--------------------- 

注意：find在查找链接文件时，加/和不加/的区别

[root@localhost /tmp]#ll
total 0
drwxr-xr-x. 2 root root 18 Aug 16 15:59 a
lrwxrwxrwx. 1 root root  1 Aug 16 15:59 b -> a
[root@localhost /tmp]#find b/ -perm -113
b/b.txt
[root@localhost /tmp]#find b -perm -113
b
也就是说：如果目录下有文件写成ls /dir==ls /dir/，而如果像链接文件这样，ls /etc/init.d 不等于 ls /etc/init.d/
--------------------- 

 









































