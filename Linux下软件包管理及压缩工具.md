## ***<font color=#00ffff>Linux下软件包软件包管理之rpm、yum、编译安装&解压缩工具介绍</font>***

<font color=#FF0000>压缩、解压缩及归档工具 </font>

## compress/uncompress  
>compress [-dfvcVr] [-b maxbits] [file ...]  
>>-d: 解压缩，相当于uncompress  
-c: 结果输出至标准输出,不删除原文件  
-v: 显示详情  
zcat file.Z >file  

## gzip/gunzip  
>gzip [OPTION]... FILE ...  
>>-d：解压缩，相当于gunzip  
-c：结果输出至标准输出，保留原文件不改变  
-#：1-9，指定压缩比，值越大压缩比越大  
zcat：不显式解压缩的前提下查看文本文件内容  

cat messages | gzip > m.gz   
作用：把大量的执行结果，进行压缩  
数据库备份：通过数据库的命令把数据库的内容提取出来并压缩，达到备份的效果。  
tree /etc | gzip tree.gz 把显示树的结果保存并压缩.  
 
## bzip2/bunzip2/bzcat  
>bzip2 [OPTION]... FILE ...  
>>-k：keep, 保留原文件  
-d：解压缩  
-#：1-9，压缩比，默认为9  
bzcat：不显式解压缩的前提下查看文本文件内容  

## xz/unxz/xzcat  
>xz [OPTION]... FILE ...  
>>-k: keep, 保留原文件  
-d：解压缩  
-#：1-9，压缩比，默认为6  
unxz file.xz 解压缩  
xzcat: 不显式解压缩的前提下查看文本文件内容  

<font color=#FF0000>compress、gzip、bzip2、xz只能针对文件进行压缩，不能针对目录</font>

针对目录需要先打包，再进行压缩：zip、tar  

## zip/unzip  打包压缩  
>zip –r /backup/sysconfig /etc/sysconfig/  
>解包解压缩  
unzip sysconfig.zip    
cat /var/log/messages | zip messages -  
unzip -p message > message  

# <font color=#FF0000>tar工具 </font>
## tar（Tape ARchive，磁带归档的缩写）  
>tar [OPTION]...  
>>(1) 创建归档  
tar -cpvf /PATH/FILE.tar FILE.../creat/files/verson/保留权限  
  tar cvf etc.tar /data/etc 对data下的etc目录进行打包  
(2) 追加文件至归档： 注：不支持对压缩文件追加  
tar -r -f /PATH/FILE.tar FILE...  
  tar -rf usr.tar f100 把f100追加到usr.tar包中  
(3) 查看归档文件中的文件列表  
tar -tf /PATH/FILE.tar  
(4) 展开归档    
tar -x -v -f /PATH/FILE.tar  
tar -x -v -f /PATH/FILE.tar -C /PATH/ --支持解压到其他路径  
(5) 结合压缩工具实现：归档并压缩  
-j: bzip2, -z: gzip, -J: xz  
支持打包时并进行压缩  
tar Jcvf bin.tar.xz /data/bin/   
tar jcvf bin.tar.bz2 /data/bin/  
tar zcvf bin.tar.gz /data/bin/  

>-exclude 排除文件  
tar zcvf /root/a3.tgz --exclude=/app/host1 --exclude=/app/host2 /app  
-T选项指定输入文件,-X选项指定包含要排除的文件列表  
tar zcvf mybackup.tgz -T /root/includefilelist -X /root/excludefilelist  

>splist:：分割一个文件为多个文件  
分割大的 tar 文件为多份小文件  
split –b 2M  tar-file-name name --默认生成的是以a..z为后缀  
split –b Size –d tar-file-name name -- -d 是生成以数字为后缀的文件  
合并：  
cat name* > mybackup.tar.gz  


# <font color=#FF0000>cpio  </font>
 功能：复制文件从或到归档  
 cpio命令是通过重定向的方式将文件进行打包备份，还原恢复的工具，它可以解压以“.cpio”或者“.tar结尾的文件  
 cpio [选项] > 文件名或者设备名  
 cpio [选项] < 文件名或者设备名  
 选项  
>-o 将文件拷贝打包成文件或者将文件输出到设备上  
-O filename 输出到指定的归档文件名  
-A 向已存在的归档文件中追加文件  
-i 解包，将打包文件解压或将设备上的备份还原到系统  
-I filename 对指定的归档文件名解压  
-t 预览，查看文件内容或者输出到设备上的文件内容  
-F filename 使用指定的文件名替代标准输入或输出  
-d 解包生成目录，在cpio还原时，自动的建立目录  
-v 显示打包过程中的文件名称  




各版本的安装工具及包名后缀  

>Redhat, SUSE, Debian,Ubuntu,Centos  
Redhat:  
Redhat Package Manager  
PRM is Package Manager  
Debian、Ubuntu: dpb文件，dpkg包管理器  
rpm、Centos：rpm文件，rpm包管理器  

各版本解决依赖包管理工具：  
前端工具：yum(rpm), apt-get(deb)、zypper(suse)、dnf(Fedora)  
后端工具：RPM, dpt   

库文件---软件运行所依赖的库：  

>查看二进制程序所依赖的库文件：  
ldd /PATH/TO/BINARY_FILE  
eg: which rpm --- ldd /usr/bin/rpm  
管理及查看本机装载的库文件：  
ldconfig 加载库文件  
/sbin/ldconfig -p: 显示本机已经缓存的所有可用库文件名及文件路径  
映射关系  
配置文件：/etc/ld.so.conf, /etc/ld.so.conf.d/*.conf  
缓存文件：/etc/ld.so.cache  


    

# RPM:  
>包文件组成 (每个包独有)  
RPM包内的文件  
RPM的元数据，如名称，版本，依赖性，描述等  
安装或卸载时运行的脚本  

>数据库(公共)：/var/lib/rpm-记录系统内已经安装软件的各种信息-important  
程序包名称及版本  
依赖关系  
功能说明  
包安装后生成的各文件路径及校验码信息  
	
安装、查询、卸载、升级、校验、数据库的重建、验正数据包等工作；  

CentOS系统上使用rpm命令管理程序包：  
安装、卸载、升级、查询、校验、数据库维护  
安装：  
>rpm {-i|--install} [install-options] PACKAGE_FILE…  
>>-v: verbose显示详细过程  
-vv:更详细的过程  
-h: 以#显示程序包管理执行进度  
rpm -ivh PACKAGE_FILE ...  

>[install-options]  
>>--test: 测试安装，但不真正执行安装，即dry run模式  
--nodeps：忽略依赖关系--安装也无法使用  
--replacepkgs | replacefiles  
--nosignature: 不检查来源合法性  
--nodigest：不检查包完整性  
--noscripts：不执行程序包脚本  
%pre: 安装前脚本 --nopre  
%post: 安装后脚本 --nopost  
%preun: 卸载前脚本 --nopreun  
%postun: 卸载后脚本 --nopostun  

1.--replacepkgs:删除/bin/tree,  
相当于只删除tree包里的一个文件，不代表是卸载，可以用replacepkgs,覆盖安装包、  
！但是如果删除的是/usr/bin/rpm程序本身，则需要从安装包里先解压出文件，在复制到rpm原来路劲  
例：下面的eg:3.   
2.--replacefiles:以pkgs1:f1,f2 pkgs2：f2,f3，两个包都包含f2文件，在安装pkgs1时会提示已有f2文件，报  有冲突，可以用replacefiles安装pkgs1时，强制覆盖已存在f2文件

包升级  
>--force:强制安装---适用于安装，升级，但不适用于强制卸载.  
相当于--replacepkgs 强制覆盖安装  

<font color=#FF0000>注意：</font>  
(1) 不要对内核做升级操作；Linux支持多内核版本并存，因此，对直接安装新版本内核  卸载时，要指定版本号  
(2) 如果原程序包的配置文件安装后曾被修改，升级时，新版本的提供的同一个配置文件并不会直接覆盖老版本的配置文件，而把新版本的文件重命名  (FILENAME.rpmnew)后保留

## 包查询  
>rpm {-q|--query} [select-options] [query-options][select-options]  
>>-a: 所有包   
-f: 查看指定的文件由哪个程序包安装生成  
-p rpmfile：针对尚未安装的程序包文件做查询操作  
--whatprovides CAPABILITY：查询指定的CAPABILITY由哪个包所提供  
--whatrequires CAPABILITY：查询指定的CAPABILITY被哪个包所依赖  
rpm2cpio 包文件|cpio –itv 预览包内文件  
rpm2cpio 包文件|cpio –id “*.conf” 释放包内文件  

>[query-options]  
>>--changelog：查询rpm包的changelog  
-c: 查询程序的配置文件  
-d: 查询程序的文档  
-i: information  
-l: 查看指定的程序包安装后生成的所有文件  
--scripts：程序包自带的脚本  
--provides: 列出指定程序包所提供的CAPABILITY  
-R: 查询指定的程序包所依赖的CAPABILITY  

常用查询用法：  
-qi PACKAGE, -qf FILE, -qc PACKAGE, -ql PACKAGE, -qd PACKAGE  
-qpi PACKAGE_FILE, -qpl PACKAGE_FILE, ...  
-qa   
  
1.rpm -qa列出所有的包 rpm -qa | grep "tree"过滤是有某个包。  
2.rpm -ql tree 这样是查看已经安装的软件的软件列表  
>/usr/bin/tree  
/usr/share/doc/tree-1.6.0  
/usr/share/doc/tree-1.6.0/LICENSE  
/usr/share/doc/tree-1.6.0/README  
/usr/share/man/man1/tree.1.gz 

默认安装是按照预定义的规则，在安装时，将各个文件拷贝到相应的路劲下  
也可以指定安装路劲：  
--root的选项：默认安装是以根/为安装点的  
[root@centos7 data]#rpm -ivh /run/media/root/CentOS\ 7\   x86_64/Packages/tree-1.6.0-10.el7.x86_64.rpm --root=/data/  
则以/data为安装点将上面的包内所有问价复制到/data目录下.  

3.查询某个软件来源于哪个包--后面跟的是文件  
rpm -qf /path/file  
[root@centos7 data]#rpm -qf /usr/bin/cat  
coreutils-8.22-21.el7.x86_64  
如果删除了/usr/bin/cat  
可以把对应的包通过rpm2cpio解开包  
rpm2cpio coreutils-8.22-21.el7.x86_64.rpm | cpio -id ./usr/bin/cat  
然后把解开的文件移动到原来的目录，不过这样可能会丢失原来文件的权限属性  
(注意：当前路劲.和只解压丢失的单个文件，不是列表中的所有。)  
--可以强制重新安装一遍--replacepkgs--force.  

4.rpm -qpl /path/*.rpm     
  是在未安装某个包之前，先查看这个包都有什么文件，后面跟的是完整的包名rpm  -ql binry.file，
  而这样是查看已经安装的软件，都有哪些文件-后面跟的是二进制的命令路径  
5.rpm -qi pkgs：显示已安装的软件的安装信息.  
```bash
[root@centos7 data]#rpm -qi tree
Name        : tree
Version     : 1.6.0
Release     : 10.el7
Architecture: x86_64
Install Date: Tue 16 Oct 2018 11:21:59 AM CST
Group       : Applications/File
Size        : 89505
License     : GPLv2+
Signature   : RSA/SHA256, Fri 04 Jul 2014 01:36:46 PM CST, Key ID 24c6a8a7f4a80eb5
```
6.rpm -qpi pkgs_file 显示安装包的信息 

7.面试题：查询某个命令来源于哪个包  
>which command  
rpm -qf /path/command 

8.如果删除了rm -f /usr/bin/rpm  
  >可以通过救援模式：通过救援模式的rpm,在/mnt/sysimage/下重新安装  
  rpm -ivh /run/media/root/CentOS\ 7\ x86_64/Packages/rpm-4.11.3-32.el7.x86_64.rpm   --root=/mnt/sysimage/ 

9.删除/lib64/libc.so.6，并恢复  
  救援模式下：  
            1.cd /mnt/sysimage/lib64  
              ln -s libc-2.17.so libc.so.6 (本身就是个软链接，重新创建即可)  
            2.cp /lib64/libc.so.6 /mnt/sysimage/lib64   
              复制救援模式/下的lib64/libc.so.6到操作系统原来的路径下.  

## 包卸载：  
>rpm {-e|--erase} [--allmatches] [--nodeps] [--noscripts] [--notriggers]
[--test] PACKAGE_NAME ...  

包校验  
>rpm {-V|--verify} [select-options] [verify-options]  
>>S file Size differs  
M Mode differs (includes permissions and file type)  
5 digest (formerly MD5 sum) differs  
D Device major/minor number mismatch  
L readLi nk(2) path mismatch  
U User ownership differs  
G Group ownership differs  
T mTime differs  
P capabilities differ  

包来源合法性验正及完整性验证  
完整性验证：SHA256  
来源合法性验证：RSA  
公钥加密  
对称加密：加密、解密使用同一密钥  
非对称加密：密钥是成对儿的  
public key: 公钥，公开所有人  
secret key: 私钥, 不能公开  
导入所需要公钥  
rpm -K|checksig rpmfile 检查包的完整性和签名  
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7  
CentOS 7发行版光盘提供：RPM-GPG-KEY-CentOS-7  
rpm -qa “gpg-pubkey*”  
 

# <font color=#FF0000>yum</font>
CentOS: yum, dnf  
YUM: Yellowdog Update Modifier，rpm的前端程序，可解决软件包相关依  
赖性，可在多个库之间定位软件包，up2date的替代工具  
yum repository: yum repo，存储了众多rpm包，以及包的相关的元数据  
文件（放置于特定目录repodata下）  
>文件服务器：  
http://  
https://  
ftp://  
file://  

## yum配置文件  
 yum客户端配置文件：  
/etc/yum.conf：为所有仓库提供公共配置  
/etc/yum.repos.d/*.repo：为仓库的指向提供配置  
仓库指向的定义：  
```bash
[repositoryID]  
name=Some name for this repository  
baseurl=url://path/to/repository/  
enabled={1|0}  
gpgcheck={1|0}  
gpgkey=URL  
enablegroups={1|0}  
failovermethod={roundrobin|priority}  
roundrobin：意为随机挑选，默认值  
priority:按顺序访问  
cost= 默认为1000  
```
$releasever: 当前OS的发行版的主版本号  
$basearch：基础平台；i386, x86_64  


yum-config-manager  
生成172.16.0.1_cobbler_ks_mirror_CentOS-X-x86_64_.repo  
yum-config-manager --add-repo= http://172.16.0.1/cobbler/ks_mirror/7/  
yum-config-manager --disable "仓库名" 禁用仓库  
yum-config-manager --enable "仓库名" 启用仓库  


## ***yum命令***  
yum命令的用法：  
yum [options] [command] [package ...]  
显示仓库列表：  
yum repolist [all|enabled|disabled]  
list: 列表   
>支持glob -通配符  
all  
available：可用的，仓库中有但尚未安装的  
installed: 已经安装的  
updates: 可用的升级  
clean: 清理缓存  
	[ packages | headers | metadata | dbcache | all ]  
	
repolist: 显示repo列表及其简要信息,生成  
>all  
enabled： 默认  
disabled  
install: 安装  
yum install PACKAGE_NAME  
update: 升级  
update_to: 升级为指定版本  

remove|erase：卸载  
>info:   
>provides| whatprovides: 查看指定的文件或特性是由哪个包安装生成的;   
	
>groupinfo 包组信息   
grouplist 包组列表  
groupinstall 安装包组 yum groupinstall "Development Tools" 后面加双“号！  
groupremove 删除包组  
groupupdate 升级包  
```bash
yum grouplist installed | grep "development tools" --查看包组是否安装  
[root@centos7 data]#yum grouplist installed | grep "Development Tools"  
Development Tools  
```
yum list installed | grep "java" ---查看单个软件是否安装  
```bash
[root@centos7 data]#yum list installed | grep "^java"  
java-1.8.0-openjdk.x86_64               1:1.8.0.161-2.b14.el7          @anaconda  
java-1.8.0-openjdk-headless.x86_64      1:1.8.0.161-2.b14.el7          @anaconda  
javapackages-tools.noarch               3.4.1-11.el7                   @anaconda  
```
yum search *---搜索  
```bash
[root@centos7 data]#yum search apr  
Loaded plugins: fastestmirror, langpacks  
Loading mirror speeds from cached hostfile  
apr-devel.i686 : APR library development kit  
apr-devel.x86_64 : APR library development kit   
```
1.安装lsb_release命令：查看内核版本  
默认从光盘里安装，会找不到对应的包  
>在没有此命令的时候，执行会报命令不存在，可以通过一个方法：yum provides */lsb_release  
来显示有哪些包提供这个功能  
```bash
[root@centos7 data]#yum provides */lsb_release  
Loaded plugins: fastestmirror, langpacks  
Loading mirror speeds from cached hostfile  
redhat-lsb-core-4.1-27.el7.centos.1.i686 : LSB Core module support  
Repo        : epel
Matched from:
Filename    : /usr/bin/lsb_release

redhat-lsb-core-4.1-27.el7.centos.1.x86_64 : LSB Core module support
Repo        : cdrom
Matched from:
Filename    : /usr/bin/lsb_release

redhat-lsb-core-4.1-27.el7.centos.1.x86_64 : LSB Core module support
Repo        : epel
Matched from:
Filename    : /usr/bin/lsb_release

然后通过yum install redhat-lsb-core-4.1-27.el7.centos.1.x86_64来安装.
```

2.创建自己的第三方包装yum仓库  
>1.mkdir /data/repodb  
2.将第三方的安装到copy到该目录  
3.creatrepo /data/repodb 创建repodata,repodata目录下会自动生成元数据.   
4.修改yum源路径为当前目录即可.  


搭建一个基于HTTP协议的网络yum仓库  
3.建立网络yum服务器  
功能：centos7下搭建，同时提供给centos6和7使用.  
>yum install httpd  
rpm -ql httpd | grep service---httpd服务启动的文件  
systmctl start http.service--启动httpd服务  
mkdir -pv /var/www/html/centos/{6,7}/os/x86_64 ---html下创建6,7的目录  
mount /dev/sr0 /var/www/html/centos/7/os/x86_64   
mount /dev/sr1 /var/www/html/centos/6/os/x86_64将6,7光盘挂载到html下在6,和7分别配置yum客户端  


# <font color=#FF0000>编译安装</font>
    
C语言源代码编译安装三步骤：  
>1、./configure  
(1) 通过选项传递参数，指定启用特性、安装路径等；执行时会参考用户的  
指定以及Makefile.in文件生成Makefile  
(2) 检查依赖到的外部环境，如依赖的软件包  
2、make 根据Makefile文件，构建应用程序  
3、make install 复制文件到相应路径  
开发工具：  
autoconf: 生成configure脚本  
automake：生成Makefile.in  
注意：安装前查看INSTALL，README  
1../configure -help 安装信息功能设置项，默认安装路径等  
2.install 文件，提示安装方法  
3.README.软件的作用  
  
## 安装后的配置：  
>(1) 二进制程序目录导入至PATH环境变量中  
编辑文件/etc/profile.d/NAME.sh  
export PATH=/PATH/TO/BIN:$PATH  
(2) 导入库文件路径  
编辑/etc/ld.so.conf.d/NAME.conf  
添加新的库文件所在目录至此文件中  
让系统重新生成缓存：  
ldconfig [-v]  
(3) 导入头文件  
基于链接的方式实现：  
ln -sv  
(4) 导入帮助手册  
编辑/etc/man.config|man_db.conf文件  
添加一个MANPATH 


源码编译安装httpd-2.4.25.tar.bz2  
>1 yum groupinstall "development tools"  
yum install apr-devel apr-util-devel pcre-devel openssl-devel  
2 useradd -r -u 80 -d /data/www/ -s /sbin/nologin httpd  
3 tar xf httpd-2.4.25.tar.bz2   
cd cd httpd-2.4.25/  
4 cat README  
cat INSTALL  
5 ./configure --help  
 ./configure --prefix=/app/httpd --sysconfdir=/etc/httpd24  --enable-ssl --disable-status  
6 make && make install   
7 PATH变量  
echo 'PATH=/app/httpd/bin:$PATH' > /etc/profile.d/httpd.sh  
. /etc/profile.d/httpd.sh  
8 apachectl start  
>>卸载编译安装的包：  
1.删除安装时指定的路劲  
2.删除安装时创建的用户  
3.卸载安装时的依赖的包  
4.删除修改的PATH变量  
5.删除自定义的man手册路径  