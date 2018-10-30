# Linux下磁盘存储和文件系统


## 管理分区  
列出块设备  
• lsblk  
创建分区使用：  
>• fdisk 创建MBR分区  
• gdisk 创建GPT分区  
• parted 高级分区操作  
重新设置内存中的内核分区表版本  
• partprobe  

parted命令--可运行在交互环境下，也可以运行在批量执行的环境  
parted的操作都是实时生效的，小心使用  
用法：parted [选项]... [设备 [命令 [参数]...]...]  
>parted /dev/sdb mklabel gpt|msdos  
parted /dev/sdb print  
parted /dev/sdb mkpart primary 1 200 （默认M）  
parted /dev/sdb rm 1  
parted –l 列出分区信息  


## parted工具操作因为是实时的，比较不安全，推荐使用fdisk和gdisk  
>分区工具fdisk(DOS/MBR)和gdisk(GPT)  
gdisk /dev/sdb 类fdisk 的GPT分区工具  
fdisk -l [-u] [device...] 查看分区  
fdisk /dev/sdb 管理分区  
>>子命令：  
p 分区列表  
t 更改分区类型---比如raid类型-fd和LVM类型-8e--swap-82  
n 创建新分区  
d 删除分区  
v 校验分区  
u 转换单位  
w 保存并退出  
q 不保存并退出  

### 同步分区表  
查看内核是否已经识别新的分区  
cat /proc/partations  
centos6通知内核重新读取硬盘分区表  
新增分区用  
partx -a /dev/DEVICE  
kpartx -a /dev/DEVICE -f: force  
删除分区用  
partx -d --nr M-N /dev/DEVICE  
CentOS 5，7: 使用partprobe  
partprobe [/dev/DEVICE]  
 

---------------------------------
----------------------------------

1.实验：分区表的备份和还原  
>删除/dev/sda分区表的后64字节，并恢复.  
dd if=/dev/sda of=dpt bs=1 count=64 skip=446 -备份分区表  
scp dpt 192.168.34.101/data  
提示信息：  
error:no such partition.  
Entering rescue Mode...  
grub rescue  
配置网络ipconfig ens33 192.168.34.103/24  
scp 192.168.34.101/data/dpt . ---拷贝到当前路径下  
dd if=dpt of=/dev/sda bs=1 count=64 seek=446  
sync--把当前修改保存到内存中的数据同步到磁盘上  

2.生成UUID的方法--uuidgen  

3.创建分区前，先parte /dev/sdb1 mklabel msdos|gtp 指定分区类型  

4.cat /proc/partitions 查看系统所有分区=lsblk= ls /dev/sd* = fdisk -l /dev/sdb  
  备注：前面三个查看的是内存中的设备信息，而fdisk -l /dev/sdb则是硬盘上的分区信息数据所以有时会看到这4个命令看的分区表信息不一致，分区后还需要同步到硬盘上  

5.在不关机重启的情况下，新增分区后删除分区，需要把硬盘上分区信息同步到系统内存中  
>partprobe /dev/sdb ---硬盘上分区信息同步到内存中--针对centos5和7  
  partx -a /dev/sdb ---针对centos6 新增分区后，把硬盘上的分区信息同步到内存中  
  partx -d --nr 5 /dev/sdd 删除分区后，同步硬盘分区信息到内存中后面跟删除的分区编号--centos 6  
---------------------------------
----------------------------------

# 创建文件系统  
文件系统是操作系统用于明确存储设备或分区上的文件的方法和数据结构；即在存储设备上组织文件的方法。操作系统中负责管理和存储文件信息的软件结构称为文件管理系统，简称文件系统  

从系统角度来看，文件系统是对文件存储设备的空间进行组织和分配，负责文件存储并对存入的文件进行保护和检索的系统。具体地说，它负责为用户建立文件，存入、读出、修改、转储文件，控制文件的存取，安全控制，日志，压
缩，加密等  

支持的文件系统：ls /lib/modules/`uname –r`/kernel/fs  
>1.没有文件系统，创建的分区只能以二进制的方式存储和访问，文件系统根本特性是把磁盘上的不同数据以文件方式进行管理  
2.日志工作原理：磁盘数据-读入内存修改-保存到文件系统的日志中journel当内存中修改的数据写到磁盘的过程中出故障时，可以将日志中的数据还原到磁盘中，保证磁盘上数据的完整性  
  所以：先写日志--再写数据  

## 创建文件系统  
>mkfs命令：  
>>(1) mkfs.FS_TYPE /dev/DEVICE  
ext4  
xfs  
btrfs  
vfat  
(2) mkfs -t FS_TYPE /dev/DEVICE  
-L 'LABEL' 设定卷标  

## 创建ext文件系统  
>mke2fs：ext系列文件系统专用管理工具  
>>-t {ext2|ext3|ext4} 指定文件系统类型    
-b {1024|2048|4096} 指定块大小---指定后不能更改  
-L ‘LABEL’ 设置卷标--挂载文件系统是不建议使用label,建议使用UUID---命令规则：一般为挂载的目录名 /mnt/sdb1  
-j 相当于 -t ext3  
mkfs.ext3 = mkfs -t ext3 = mke2fs -j = mke2fs -t ext3  
-i # 为数据空间中每多少个字节创建一个inode；不应该小于block大小  
-N # 指定分区中创建多少个inode  
-I 一个inode记录占用的磁盘空间大小，128---4096  
-m # 默认5%,为管理人员预留空间占总空间的百分比  
-O FEATURE[,...] 启用指定特性  
-O ^FEATURE 关闭指定特性  

文件系统标签  
指向设备的另一种方法  
与设备无关  
>blkid：块设备属性信息查看  
blkid [OPTION]... [DEVICE]  
-U UUID 根据指定的UUID来查找对应的设备  
-L LABEL 根据指定的LABEL来查找对应的设备  
e2label：管理ext系列文件系统的LABEL  
e2label DEVICE [LABEL]  
findfs ：查找分区  
findfs [options] LABEL=<label>  
findfs [options] UUID=<uuid>  

### 文件系统检测和修复  
>常发生于死机或者非正常关机之后  
挂载为文件系统标记为“no clean”  
注意：一定不要在挂载状态下修复  
fsck: File System Check  
fsck.FS_TYPE  
fsck -t FS_TYPE  
-p 自动修复错误  
-r 交互式修复错误  
FS_TYPE 一定要与分区上已经文件类型相同  
e2fsck：ext系列文件专用的检测修复工具  
-y 自动回答为yes  
-f 强制修复

## tune2fs  
>tune2fs：重新设定ext系列文件系统可调整参数的值---元数据，属性信息  
>>-l 查看指定文件系统超级块信息；super block  
-L 'LABEL'修改卷标  
-m # 修预留给管理员的空间百分比  
-j 将ext2升级为ext3  
-O 文件系统属性启用或禁用, –O ^has_journal  
-o 调整文件系统的默认挂载选项，–o ^acl---centos6上人为分的分区默认不带acl选项  
-U UUID 修改UUID号  
dumpe2fs：  
块分组管理，32768块  
-h：查看超级块信息，不显示分组信息=tune2fs -l   

步骤：  
>1.parte /dev/sdb1 mklabel msdos|gtp --指定分区类型  
2.fdisk|gdisk /dev/sdb  --硬盘分区  
3.mkfs.ext4 /dev/sdb1 ---单个主分区或者逻辑分区，创建指定文件系统格式  
4.blkid /dev/sdb1 --查看格式化的分区是否成功---查看分区信息，LABEL、UUID、文件系统类型  
5.tune2fs /dev/sdb1 ---查看分区创建的文件系统内带有的详细信息--比如日志功能，acl权限等--只支持ext系列的  

---信息内参数解释：最小块大小4096--决定了在这个系统上给文件分配的最小单位是4K--window下成为簇  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;系统内保留块数量：作用：留给管理员用户---当文件系统出现故障时,做维护使用的空间 一般为5%  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;最大挂载次数和检查间隔--作用：当挂载次数达到限定数或者检查间隔到了后，会对当前文件  &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;系统做检查、修复，保证文件系统的稳定  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;块组：记录每个组中的块数量和属性信息，块组又包括：  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;Super Block超级块--存放元数据-哪些块在这些组内等  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;GPT的分区表信息  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;块位图，节点位图： 标记哪些块或者节点是否被使用了：1 -表示已使用；0 -未使用  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;节点表：记录系统内文件的属性信息和元数据存放的位置  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;Data Blocks：数据存在的真正位置  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;default mount options:挂载选项，在mount时，默认带什么选项--acl  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;可以在挂载前，先加上acl功能：tune2fs -o acl /dev/sdb1 取消用^acl  


ext系列的文件系统在mkfs /dev/sdb1 时如果没有加卷标时，  
后面可以用e2label -L /mnt/sdb1(挂载目录名) /dev/sdb1,不需要重新格式化  

3.查看/etc/fstab中/根来自于哪个分区  
  blkid -U `sed -nr 's@UUID=(.*) / .*@\1@p' /etc/fstab`  

4.在centos6之前版本手动创建的ext分区，默认没有ACL功能，可用une2fs -o acl /dev/sdb1加上  
  安装操作系统时创建的分区是带acl功能的  
  centos7上不管是安装操作系统还是手动创建的分区默认都带acl功能  

---------------------------------
-----------------------------------
---------------------------------

# 挂载mount  

挂载方法：mount DEVICE MOUNT_POINT  
mount：通过查看/etc/mtab文件显示当前已挂载的所有设备  
>mount [-fnrsvw] [-t vfstype] [-o options] device dir  
>>device：指明要挂载的设备；  
(1) 设备文件：例如/dev/sda5  
(2) 卷标：-L 'LABEL', 例如 mount -L 'MYDATA'   
(3) UUID, -U 'UUID'：例如 mount -U '0c50523c-43f1-45e7-85c0-a126711d406e' /mnt/sdb1  
(4) 伪文件系统名称：proc, sysfs, devtmpfs, configfs  
dir：挂载点  
事先存在；建议使用空目录  
进程正在使用中的设备无法被卸载  

>>-t vsftype 指定要挂载的设备上的文件系统类型  
-r readonly，只读挂载  
-w read and write, 读写挂载  
-n 不更新/etc/mtab，mount不可见---不支持centos7  
-a 自动挂载所有支持自动挂载的设备(定义在了/etc/fstab文件中，且挂载选项中有auto功能)  
-L 'LABEL' 以卷标指定挂载设备  
-U 'UUID' 以UUID指定要挂载的设备  
-B, --bind 绑定目录到另一个目录上  
查看内核追踪到的已挂载的所有设备  
cat /proc/mounts  

>-o options：(挂载文件系统的选项)，多个选项使用逗号分隔  
async 异步模式 sync 同步模式,内存更改时，同时写磁盘  
atime/noatime 包含目录和文件 ---访问是，不更新访问时间，提高效率  
diratime/nodiratime 目录的访问时间戳  
auto/noauto 是否支持自动挂载,是否支持-a选项  
exec/noexec 是否支持将文件系统上运行应用程序  
dev/nodev 是否支持在此文件系统上使用设备文件  
suid/nosuid 是否支持suid和sgid权限  
remount 重新挂载  
ro只读 rw读写  
user/nouser 是否允许普通用户挂载此设备，/etc/fstab使用  
acl 启用此文件系统上的acl功能  
loop 使用loop设备---文件挂载  
defaults：相当于rw, suid, dev, exec, auto, nouser, async  

1.一个设备可以同时挂载到多个目录---相当于一个设备有多个路径  
  一个目录可挂载多个设备呢？不可以---最新挂载的设备会生效，最先挂载的会隐藏起来看不到  

2.卸载：需要确认是否有其他用户正在使用此文件系统  
  >方法：umount /dev/sdb1 或者umount /mnt/sdb1 可以卸载设备名或者挂载点  
  lsof /mnt/sdb1  
  fuser -v /mnt/sdb1  
  可以查看正在访问该文件系统的用户和进程  
  fuser -km /dev/sdb1 ---先通知用户再杀死所有进程  
  提示：$? 返回值可以判断该挂载点是否有人在用  

3.mount单独使用：可以看到正在挂载的文件系统状态  
  =cat /proc/mounts  

4.-o remount选项：作用：想要更改某个处在挂载状态的文件系统的权限等信息时，在不想取消挂载的情况下  
                  重新挂载并加上需要更改的选项  
                  eg:mount -o remount，ro /dev/sdb1 不取消挂载更新为只读权限  
                     mount -o remount，acl /dev/sdb1 不取消挂载同时加上acl功能---centos6  
                     ==tune2fs -o acl /dev/sdb1---只是在挂载时加上acl功能，不改变文件系统本身  

5.-o loop：可以把一个大文件格式化创建一个文件系统然后挂载--类似U盘效果-传给其他机器进行挂载  
          centos6挂载时需要加 -o loop  
          centos7不需要  
6. -o nodev/dev  noexec/exec  user/nouser 一般挂载时禁用，保证系统的安全性,普通用户不能挂载  

7.findmnt /data --echo $? 判断某个目录是否为挂载点，然后再挂载  

---------------------------------
-----------------------------------
---------------------------------

## 挂载点和/etc/fstab  
配置文件系统体系  
被mount、fsck和其它程序使用  
系统重启时保留文件系统体系  
可以在设备栏使用文件系统卷标  
使用mount -a 命令挂载/etc/fstab中的所有文件系统  

/etc/fstab每行定义一个要挂载的文件系统  
>1、要挂载的设备或伪文件系统  
设备文件  
LABEL：LABEL=""  
UUID：UUID=""  
伪文件系统名称：proc, sysfs  
2、挂载点  
3、文件系统类型：ext4,xfs,nfs,none  
4、挂载选项：defaults ，acl，bind   
5、转储频率：0：不做备份 1：每天转储 2：每隔一天转储  
6、fsck检查的文件系统的顺序：允许的数字是0, 1, 和2  
0：不自检  
1：首先自检；一般只有rootfs才用  
2：非rootfs使用  

1.在fstab中的添加需要开机挂载的文件系统  
  mount -a 生效,如果在fstab中修改已经挂载的文件系统的权限信息（acl,ro,rw等）  
  则需要重新挂载 mount -o remount /dev/sdb1  

2.文件，目录，光盘，分区挂载到fstab的格式  
```bash
UUID=9612633f-e7f1-4b28-8813-403d209d7abc /                       xfs     defaults        0 0
UUID=0eba9e52-43a4-4c64-89bd-3fb639f0a6c1 /boot                   xfs     defaults        0 0
UUID=06fe63d1-9b57-436f-ad7d-1c01c7a60aee /data                   xfs     defaults        0 0
UUID=48e92f06-6fcb-4a21-8ba0-e7d1c5c1af39 swap                    swap    defaults        0 0
UUID=d84e4bab-88e6-40a9-9d16-c03201c0f3c9 /mnt/sdb1               ext4     defaults       0 3
/boot --目录                              /mnt/boot               none     bind           0 0
/data/ext4file --文件                     /mnt/ext4               ext4     loop           0 0
/dev/sr0  --光盘                          /mnt/cdrom              iso9660  defaults       0 0 
```
3.实验：  
 > 开机启动的信息保存在/var/log/boot.log下  
centos6下：  
  a.把/dev/sdb1的分区表破坏时，重启时会不能进入系统 --设备出问题的情况  
    解决办法：1.fsck -y /dev/sdb1 修复  
             2.修改根/为rw，mount -o remount,rw /  
               a.注释掉/dev/sdb1挂载行  
               b.把最后的检查项3改为0，即开机不检查，即不报错  
  b.fstab中设备的挂载点不存在或错误的情况 --不影响开机启动  
centos7下：  
  和centos6不同：开机检查项即使为0,也无法进入系统  
           解决办法：进入救援模式： vim /mnt/sysimage/etc/fstab注释掉出现错误的文件系统  

---------------------------------
-----------------------------------
---------------------------------

## 交换分区是系统RAM的补充  
基本设置包括：  
• 创建交换分区或者文件  
• 使用mkswap写入特殊签名  
• 在/etc/fstab文件中添加适当的条目  
• 使用swapon -a 激活交换空间  

swapon  
>swapon [OPTION]... [DEVICE]  
>>-a：激活所有的交换分区  
-p PRIORITY：指定优先级  
/etc/fstab:pri=value  
禁用：swapoff [OPTION]... [DEVICE]  

1.RAM<=8G时   RAM >= swap <=2*RAM   
  mount看不到swap分区：swap是模拟内存用，不需要挂载到文件系统上  

### ***2.实验：增加内存条的同时，扩展swap分区大小***
  >前提：swap分区是和内存交换的，所以速度很大程度影响机器性能；所以一般放到固态硬盘上，或者磁盘的外圈sdd1  
  a.创建分区,修改文件系统ID为82  
  b.mkswap /dev/sdd1 --创建文件系统  
  c.添加到/etc/fstab下  
  d.生效：swapon -a   
                -s 当前系统生效的swap分区   
```bash
  UUID=06fe63d1-9b57-436f-ad7d-1c01c7a60aee /data                   xfs     defaults        0 0
  UUID=48e92f06-6fcb-4a21-8ba0-e7d1c5c1af39 swap                    swap    defaults        0 0
  UUID=b908c2d6-85c5-4081-9ed6-21f51603e5e7 swap                   swap     pri=10          0 0
```
3.swap的优先级：cat /etc/swaps  在fstab中pri=10来定义优先级  
 修改优先级需要 swapoff /dev/sdd1 先禁用，再swapon -a 生效

4.如果swap分区是使用文件模拟的情况  
  >添加：  
  a.生成一个2G文件 dd if=/dev/zero of=/data/swapfile bs=2G count=1  
  b.mkswap /data/swapfile  
  c.添加到fstab中，生效swapon -a  
  迁移：如果当前硬盘使用太满，则可迁移swap分区  
  a.swapoff /data/swapfile  
  b. mv /data/swapfile /boot  
  c.修改fstab中的路径  
  d.swapon -a 生效即可    
  删除：不需要时，可删除  
  a.swapoff /data/swapfile 先禁用  
  b.删除fstab中添加的swap分区信息  
  c.rm -rf /data/swapfile或者 fdisk /dev/sdd1 --d删除  

  备注：有时blkid看不到更创建的swapfile,需要 blkid /data/swapfile查看UUID  
  ```bash
  UUID=06fe63d1-9b57-436f-ad7d-1c01c7a60aee /data                   xfs     defaults        0 0
  UUID=48e92f06-6fcb-4a21-8ba0-e7d1c5c1af39 swap                    swap    defaults        0 0
  /data/swapfile                            swap                    swap   defaults        0 0    
```
5.场景：home家目录，由于个人用户占用根/分区空间，容易造成/崩溃，怎么迁移/home到其他分区    
  >fdisk /dev/sda7  
  partprobe  
  mkfs.xfs /dev/sda7  
  mkdir /mnt/home  
  mount /dev/sda7 /mnt/sda7    
  cp -a /home /mnt/home  
  /dev/sda7写入到fstab中，mount -a 生效，开机自动挂载    

---------------------------------
-----------------------------------
---------------------------------

### 使用光盘  
在图形环境下自动启动挂载/run/media/<user>/<label>  
否则就必须被手工挂载  
mount /dev/cdrom /mnt/  
eject命令卸载或弹出磁盘 --可以用eject -t 弹出光驱来定位主机   
创建ISO文件  
cp /dev/cdrom /root/centos7.iso  
mkisofs -r -o /root/etc.iso /etc  
刻录光盘  
wodim –v –eject centos.iso  

---------------------------------
-----------------------------------
---------------------------------
## 1.du和df&dd

>df [OPTION]... [FILE]...    
>>-H 以1000为单位    
-T 文件系统类型    
-h: human-readable    
-i：inodes instead of blocks    
-P: 以Posix兼容的格式输出    
查看某目录总体空间占用状态：    
du [OPTION]... DIR    
-h: human-readable    
-s: summary --max-depth     
du -sh /etc | bigfile 查看某个文件或者目录的大小    


工具dd  
dd命令：convert and copy a file  
用法：  
>dd if=/PATH/FROM/SRC of=/PATH/TO/DEST  
bs=#：block size, 复制单元大小  
count=#：复制多少个bs  
of=file 写到所命名的文件而不是到标准输出  
if=file 从所命名文件读取而不是从标准输入  
bs=size 指定块大小（既是是ibs也是obs)  
ibs=size 一次读size个byte  
obs=size 一次写size个byte  
cbs=size 一次转化size个byte  
skip=blocks 从开头忽略blocks个ibs大小的块  
seek=blocks 从开头忽略blocks个obs大小的块  
count=n 只拷贝n个记录  
conv=conversion[,conversion...] 用指定的参数转换文件  
转换参数:  
ascii 转换 EBCDIC 为 ASCII  
ebcdic 转换 ASCII 为 EBCDIC  
lcase 把大写字符转换为小写字符  
ucase 把小写字符转换为大写字符  
nocreat 不创建输出文件  
noerror 出错时不停止  
notrunc 不截短输出文件  
sync 把每个输入块填充到ibs个字节，不足部分用空(NUL)字符补齐  
Fdatasync 写完成前，物理写入输出文件  
有一个大与2K的二进制文件fileA。现在想从第64个字节位置开始读取，需要读取的大小是128Byts。又有fileB, 想把上面读取到的128Bytes写到第32个字节开始的位置，替换128Bytes，实现如下    
dd if=fileA of=fileB bs=1 count=128 skip=63 seek=31 conv=notrunc    

---------------------------------
-----------------------------------
---------------------------------
# RAID  

>RAID-0: 读写性能提升，无容错能力；>=2；可用容量：磁盘个数总容量  
RAID-1: 读写性能提示，有容错性，镜像备份功能，>=2；可用容量：N/2 --只能防止硬件损坏，不能防止数据删除  
RAID-4: 读写性能提升，有容错性:n-1/n；>=3 ;可用容量：磁盘个数-1 -单独一块磁盘放校验位数据  
RAID-5: 读写性能提升, 有容错性：n-1/n;>=3 ;可用容量：磁盘个数-1 -每块磁盘都放校验位   
        一般采用空闲硬盘技术，在raid5加一块spare硬盘，加上一块spare硬盘，可以损坏2块，降级使用  
RAID-6: 2个重校验位，>=4,有容错性：n-2/n  
RAID-10: 先做raid1-raid0；>=4;可用容量：N/2；容错性：2/3-可接受  
RAID-01: 先做raid0-raid1;>=4;可用容量：N/2--容错性：1/3可接受  
RAID-50: 读、写提升冗余能力；>=6;有空间利用率：(n-2)/n  



>RAID-0：连续以位或字节为单位分割数据，并行读/写于多个磁盘上，因此具有很高的数据传输率，但它没有数据冗余，因此并不能算是真正的RAID结构。  
      RAID-0只是单纯地提高性能，并没有为数据的可靠性提供保证，而且其中的一个磁盘失效将影响到所有数据。因此，RAID-0不能应用于数据安全性要求高的场合。

>RAID-1：它是通过磁盘数据镜像实现数据冗余，在成对的独立磁盘上产生互为备份的数据。当原始数据繁忙时，可直接从镜像拷贝中读取数据，因此RAID-1可以提高读取性能。
      RAID1是磁盘阵列中单位成本最高的，但提供了很高的数据安全性和可用性。当一个磁盘失效时，系统可以自动切换到镜像磁盘上读写,而不需要重组失效的数据。简单来说就是：镜象结构，类似于备份模式，一个数据被复制到两块硬盘上。

>RAID10:高可靠性与高效磁盘结构一个带区结构加一个镜象结构，因为两种结构各有优缺点，因此可以相互补充。主要用于容量不大，但要求速度和差错控制的数据库中。

>RAID5：分布式奇偶校验的独立磁盘结构，它的奇偶校验码存在于所有磁盘上，任何一个硬盘损坏，都可以根据其它硬盘上的校验位来重建损坏的数据。支持一块盘掉线后仍然正常运行。
  
模拟raid0:  
>每块硬盘不需要单独创建文件系统 -成员大小相同  
>>a.拿分区来模拟需要先指定分区类型为 -fd linux raid auto  
b.mdadm -C -a yes /dev/md0 -l 0 -n 2 -c 16M /dev/sd{b,c}1  -创建raid并指定raid类型和成员个数和成员-块大小
c.创建完成后，可通过mdadm -D /dev/md0查看具体信息  
d.mkfs.ext4 /dev/md0 -创建文件系统  
e.添加到fstab中，挂载  

模拟raid5:
不同之处在于：
 mdadm -C -a yes /dev/md0 -l 5 -n 3 -c 16M  -x 1 /dev/sd{b1,c1,d1,e1} -x 指定一个做热备盘

禁用raid，然后清楚分区上的raid信息  
 >mdadm -C /dev/md0 -禁用raid  
 mdadm -A /dev/md0 -启用raid  
 mdadm --zero-superblock /dev/sdb1 -清楚分区上的raid信息  

---------------------------------
----------------------------------

# 逻辑卷管理器（LVM）  
允许对卷进行方便操作的抽象层，包括重新设定文件系统的大小  
允许在多个物理设备间重新组织文件系统  
>• 将设备指定为物理卷  
• 用一个或者多个物理卷来创建一个卷组  
• 物理卷是用固定大小的物理区域（Physical Extent，PE）来定义的  
• 在物理卷上创建的逻辑卷  是由物理区域（PE）组成  
• 可以在逻辑卷上创建文件系统  
 
 LVM: Logical Volume Manager， Version: 2  
dm: device mapper：将一个或多个底层块设备组织成一个逻辑设备的模块  
设备名：/dev/dm-#  
软链接：  
/dev/mapper/VG_NAME-LV_NAME  
/dev/mapper/vol0-root  
/dev/VG_NAME/LV_NAME  
/dev/vol0/root  

LVM可以弹性的更改LVM的容量  
通过交换PE来进行资料的转换，将原来LV内的PE转移到其他的设备中以降低LV的容量，或将其他设备中的PE加到LV中以加大容量  
 
pv管理工具  
>pvs：简要pv信息显示 --显示当前系统的所有物理卷  
pvdisplay  
创建pv  
pvcreate /dev/DEVICE  
删除pv  
pvremove /dev/DEVICE  


vg管理工具  
>显示卷组  
vgs  
vgdisplay  
创建卷组  
vgcreate [-s #[kKmMgGtTpPeE]] VolumeGroupName  
PhysicalDevicePath [PhysicalDevicePath...]  
管理卷组  
vgextend VolumeGroupName PhysicalDevicePath [PhysicalDevicePath...]  
vgreduce VolumeGroupName PhysicalDevicePath [PhysicalDevicePath...]  
删除卷组  
先做pvmove，再做vgremove  

lv管理工具  
>lvs | Lvdisplay  
创建逻辑卷  
lvcreate常用选项：  
-p|--permission {r|rw}]设置创建的逻辑卷权限 --一般做快照时加上权限  
-n|--name LogicalVolumeName] -逻辑卷的名  
-l|--extents LogicalExtentsNumber[%{VG|FREE|ORIGIN}] -指定PE -size数量  
   -L|--size LogicalVolumeSize[bBsSkKmMgGtTpPeE]} 以M,G大小来创建  
-s|--snapshot} OriginalLogicalVolume[Path] -标记为逻辑卷快照  
lvcreate -L #[mMgGtT] -n NAME VolumeGroup  
lvcreate -l 60%VG -n mylv testvg  
lvcreate -l 100%FREE -n yourlv testvg  
删除逻辑卷  
lvremove /dev/VG_NAME/LV_NAME  
重设文件系统大小  
fsadm [options] resize device [new_size[BKMGTEP]]  
resize2fs [-f] [-F] [-M] [-P] [-p] device [new_size]  
xfs_growfs /mountpoint   
  

扩展和缩减逻辑卷 
>扩展逻辑卷：  
>>lvextend -L [+]#[mMgGtT] /dev/VG_NAME/LV_NAME  
resize2fs /dev/VG_NAME/LV_NAME  
lvresize -r -l +100%FREE /dev/VG_NAME/LV_NAME  
缩减逻辑卷：  
umount /dev/VG_NAME/LV_NAME  
e2fsck -f /dev/VG_NAME/LV_NAME 
resize2fs /dev/VG_NAME/LV_NAME #[mMgGtT]   
lvreduce -L [-]#[mMgGtT] /dev/VG_NAME/LV_NAME  
mount

1.raid要求每个成员大小相同，LVM则不需要 -逻辑卷内部支持raid镜像功能

2.逻辑卷空间来自于卷组--卷组空间来自于多块硬盘
多块块设备（硬盘或分区）--组成物理卷（pvcreat，把多块硬盘进行标记为逻辑卷来用）--组成卷组（vgcreat）--生成逻辑卷（lvcreat）

## 3.创建逻辑卷的步骤：  
 >1.格式化至少两块硬盘或分区--改分区格式为 --8e --硬盘不需要改8e分区需要改  
 2.pvs | pvdisplay -查看是否有物理卷  
 3.pvcreate /dev/sd{c1,d}  -生成物理卷 -然后再pvs查看是否生成物理卷  
 4.vgs | vgdisplsy -查看是否有卷组  
 5.vgcreate  -s 16M vg0 /dev/sd{c1,d} -组成卷组 -再pvs查看-可看到物理卷属于什么卷组  
   其中 -s 16M 为pe -size -给逻辑卷分配空间的最小单位  
 6.lvs | Lvdisplay -查看是否有逻辑卷  
 7.lvcreate -n lv_test -L 3G vg0 --创建逻辑卷-指定设备名和大小  
 8.mkfs.ext4 /dev/vg0/lv_test  -创建文件系统  
 9.mount /dev/vg0/lv_test /mnt/test -挂载  

## 扩展逻辑卷步骤：  
 >1.确保卷组有空间  
 2.lvextend -L +2G | +100%FREE /dev/vg0/lv_test -可写加多少或者全部使用  
 3.df看到的是空间上的文件系统的大小 -虽然加了空间但是新增的空间没有文件系统  
 4.resize2fs /dev/vg0/lv_test -在线扩展逻辑卷-命令只适用于ext系列  
  备注：xfs系列的扩展用 xfs_growfs + 挂载点，而resize2fs + 设备  
       代替2,4步骤，不管是ext还是xfs支持一步扩展  
       lvextend -r -L +2G /dev/vg0/lv_test
  
## 如果卷组没空间，则需扩展卷组  
  >1.pvcreate /dev/sdd -创建物理卷  
  2.vgextend vg0 /dev/sdd -加入已有卷组  
  
## 缩减逻辑卷步骤： -ext支持缩减 -xfs不支持缩减  
 >1.umount /mnt/test -不支持在线缩减, -需要取消挂载   
 2.e2fsck -f /dev/vg0/lv_test -先检查文件系统完整性  
 3.resize2fs /dev/vg0/lv_test 10G -先缩减文件系统到指定大小  
 4.lvreduce -L 10G /dev/vg0/lv_test -缩减逻辑卷大小到指定大小  
 5.mount /dev/vg0/lv_test /mnt/test -重新挂载  

## 跨主机迁移卷组  
 源计算机上  
>1 在旧系统中，umount所有卷组上的逻辑卷  
2 禁用卷组  
vgchange –a n vg0   
lvdisplay  
3 导出卷组  
vgexport vg0  
pvscan  
vgdisplay  
拆下旧硬盘  
在目标计算机上  
4 在新系统中安装旧硬盘，并导入卷组：vgimport vg0  
5 vgchange –ay vg0 启用  
6 mount所有卷组上的逻辑卷  

## 迁移逻辑卷步骤：  
>思路：把多块物理卷的上的数据迁移到一个可插拔的硬盘上  
 >>1.umount /mnt/test  
   pvdisplay -迁移前查看要迁移的物理卷在卷组中是否还要空间, -否则迁移失败
 2.pvmove /dev/sda6 -把不能插拔硬盘上的分区数据迁移到卷组其他位置-首先确保要有足够的空间     
 3.vgreduce vg0 /dev/sda6 --从卷组删除  
 4.先判断对方主机上是否有同名的卷组 -先vgrename vg0 vg1 改卷组名  
 5.vgchange -an vg1 -禁用卷组  
 6.vgexport vg1 -导出卷组  
 7.插入到另外一台主机：vgdisplay 可看到加入的卷组， -或者pvscan  
 8.vgimport vg1 -导入逻辑卷  
 9.vgchange -ay vg1 -激活vg1上的所有逻辑卷  
 10.mount /dev/vg1/lv_test /mnt/test  


## 为现有逻辑卷创建快照  

>1.快照是特殊的逻辑卷，它是在生成快照时存在的逻辑卷的准确拷贝，最大为原始LVM的大小  
2.只有在它们和原来的逻辑卷不同时才会消耗空间，原始逻辑卷的15%～20%，lvextend放大快照 --有数据更改原文件才会迁移到快照中  
3.快照就是将当时的系统信息记录下来，若将来有任何数据改动了，则原始数据会被移动到快照区，没有改动的区域则由快照区和文件系统共享  
4.由于快照区与原本的LV共用很多PE的区块，因此快照与被快照的LV必须在同一个VG中.系统恢复的时候的文件数量不能高于快照区的实际容量  
5.快照不能代替备份，作用：如果在原来的逻辑卷上做增删改查，数据太多时，备份太慢，可以考虑做备份

## 1.创建快照，并挂载：  
 > centos6：  
  a.先vgdisplay，查看同一卷组中，是否空间 --最大为原LVM大小-最小15%  
  b.lvcreate -n lv_test_snap -s -p r -L 500M /dev/vg0/lv_test  
     创建快照指定大小并设置只读权限-也可以创建完后，挂载的时候 -o ro选项  
  c.创建完快照后，lvdisplay会显示多了一项：  
    LV Creation host, time centos6.localdomain, 2018-10-20 07:02:45 +0800  
    LV snapshot status  active destination for lv_tes  
    blkid可以看到逻辑卷快照的UUID等信息  
  d.mount /dev/vg0/lv_test_snap /mnt/test_snap 把快照挂载  
    理论上，挂载后可以看到快照里的数据的 -人为提示快照做成功了实际上是不应该看到的  

  备注：centos7上，做完快照，快照的的UUID会和原来的逻辑卷UUID一样，所以挂载不了
        可以通过mount -o nouuid /dev/vg0/lv_test_snap /mnt/test_snap 

>2.修改原来逻辑卷内的数据，然后还原：  
 >> a.把逻辑卷和快照都取消挂载  
  b.lvconvert --merge /dev/vg0/lv_test_snap 合并逻辑卷和快照  
  c.快照合并后，自动删除  
3.删除快照  
  umount /mnt/test_snap  
  lvremove /dev/vg0/lv_test_snap  
  --直接删除逻辑卷时，会自动先删除快照  
 
## 逻辑卷的删除：  
  >a.umount /mnt/lv_test -取消挂载  
  b.lvremove /dev/vg0/lv_test -删除逻辑卷  
  c.vgremove  vg0 -删除卷组  
  d.pvremove /dev/sdc1 /dev/sdd -删除物理卷  


1、创建一个2G的文件系统，块大小为2048byte，预留1%可用空间,文件系统  
ext4，卷标为TEST，要求此分区开机后自动挂载至/test目录，且默认有acl挂载选项  

mkfs.ext4 -b 2048 -m 1 -L TEST /dev/sdc1 

  
