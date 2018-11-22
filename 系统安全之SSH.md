# 加密和安全(二)

    内容
        ssh服务 -重点(sshd_config配置)
            实验：本地端口，远程端口，动态端口转发原理
        openssh：实现ssh服务的开源软件
        dropbear服务  -小型的实现ssh服务的开源软件
        AIDE -系统安全检测工具
        PSSH -基于ssh-key验证的小型管理工具

## SSH
基于网络的大型服务都有C/S结构，服务端&客户端
如SSH服务，电脑上通过xsell和SecureCRT连接linux，电脑端为客户端，linux为服务器端22端口

    为什么要用ssh协议？
        因为如telnet，rlogin，但都是不安全的，ssh中间的传输过程是加密的

### ssh协议的特点和如何配置
    ssh: secure shell, protocol, tcp/22, 安全的远程登录
    实现ssh协议的软件：
        OpenSSH: ssh协议的开源实现，CentOS默认安装
        dropbear：第三方实现ssh协议开源软件，二选一即可
    SSH协议版本：目前用V2版本
        v1: 基于CRC-32做MAC，不安全；容易受到man-in-middle中间人攻击
        v2：双方主机协议选择安全的MAC方式
            基于DH算法做密钥交换，基于RSA或DSA实现身份认证        
    两种方式的用户登录认证：
    基于password:用户名密码方式，适合给用户使用，不适合程序
                可以用expect写一个脚本模拟人工输入来实现自动化
    基于key：不需输入用户名和密码，适合管理多台主机，实现自动化的基础

## OpenSSH软件
    主要有三个相关包：
        openssh         服务器&客户端都需要的安装包
        openssh-clients     客户端需要的
        openssh-server      服务器端需要的
        默认三个包机器都是安装的 rpm -qa "openssh*"
    工具：
    基于C/S结构
    客户端连接使用Client: ssh, scp, sftp，slogin
            Windows客户端：
                xshell, putty, securecrt, sshsecureshellclient
    服务器端：Server: 表现为sshd服务
    在服务器上表现为：ps aux |grep "sshd" 
        管理sshd服务：
        centos7:systemctl start|status|stop|restart sshd
            systemctl is-enabled sshd 确定开机是否启动
        centos6:service sshd start|stop|status|restart 
            chkconfig --list sshd 确定开机是否启动

# SSH的加密过程
![sshd通讯过程]()
可通过wireshark抓包体现
![抓包ssh]()


那在上述的通讯过程中，是基于已经得到对方的公钥，才能验证信息，那么这个公钥是如何事先得到的呢？
可参考下面的首次连接时的公钥交换
![首次公钥交换]()


在上面的第一个阶段，如何确认B给A的公钥一定是正确的？是否具有中间人攻击？

        第一次连接时，A-->B，底层会把B的公钥进行哈希算法传给A，然后B会把自己的公钥进行哈希   运算，得到的值进行比对，如果是则安全.
        同时，A,B会同时把对方的公钥拷贝到自己的机器上，上次连接时，通过公钥的值进行比对，确保下次连接是安全。
第一次公钥交换后，已经连接的主机IP和公钥放在各自家目录/root/.ssh/known_hosts中
A,B自己的公钥私钥放在/etc/ssh下，下面会介绍/ssh目录下的ssh配置文件

# ssh客户端
    ssh, 客户端配置文件：/etc/ssh/ssh_config
    ssh, 服务器配置文件：/etc/ssh/sshd_config

    ssh, 配置文件：/etc/ssh/ssh_config
    Host PATTERN
    StrictHostKeyChecking no 首次登录不显示检查提示
    格式：ssh [user@]host [COMMAND]
          ssh [-l user] host [COMMAND]
    常见选项
    -p port：远程服务器监听的端口    ssh 192.168.34.100 -p 1080
    -b：指定连接的源IP              对方机器有多个IP地址，指定某个IP连接
    -v：调试模式                    可以显示详细过程，有助于排错
    -C：压缩方式
    -X：支持x11转发            下文详细介绍X11通讯过程和实际作用
    -t：强制伪tty分配         很实用的工具，跨主机跨路由连接多台主机
    ssh -t remoteserver1 ssh remoteserver2

比较实用的功能：ssh -t|-X ip 选项
ssh -t ip选项作用
    场景：A-->B-->C-->D,AD在不同网段不同路由，如果通过ssh连过去？
    方式：A  ssh -t IPB ssh -t IPC ssh IPD 
            最终连接的IP不需要加-t选项
    如：跨主机连接192。168.34.4主机
    ssh -t 192.168.34.2 ssh -t 192.168.34.3 ssh 192.168.34.4

ssh -X ip 选项作用
1.当想运行ssh连接过去的服务器上的图形工具时，不加-X选项是不能运行的，
    可以通过加-X选项连接，然后再运行图形工具
2.还可以通过Xmanager工具包中的Xstart进行连接，可以直接显示服务器桌面
    进行比如oracle数据图形安装界面，退出时选择system-log out，下图是具体步骤
![xstart]()

### SSH基于key验证方式 -用于自动化运维基础
![基于key的原理]() 


在家目录下/root/.ssh/authorized_keys记录对方公钥信息，即实现key验证

    基于key认证
    基于密钥的认证：
    (1) 在客户端生成密钥对
        ssh-keygen -t rsa [-P ''] [-f “~/.ssh/id_rsa"]
    (2) 把公钥文件传输至远程服务器对应用户的家目录
        ssh-copy-id [-i [identity_file]] [user@]host
    (3) 测试
    (4) 在SecureCRT或Xshell实现基于key验证
        在SecureCRT工具—>创建公钥—>生成Identity.pub文件
        转化为openssh兼容格式（适合SecureCRT，Xshell不需要转化格式），并复制到
        需登录主机上相应文件authorized_keys中,注意权限必须为600，在需登录的ssh
        主机上执行：
        ssh-keygen -i -f Identity.pub >> .ssh/authorized_keys
    (5)重设私钥口令：
            ssh-keygen –p
    (6)验证代理（authentication agent）保密解密后的密钥
        • 这样口令就只需要输入一次
        • 在GNOME中，代理被自动提供给root用户
        • 否则运行ssh-agent bash
    (7)钥匙通过命令添加给代理
        ssh-add

#### 如何实现基于key的免用户名密码登录方式？
![key验证原理]()

    步骤：
        ssh-keygen --help
        1.ssh-keygen 默认都在/root/.ssh/下  | -P "" 不对私钥加口令
        交互式生成公钥私对，默认就行，也可以对生成的私钥进行对称加密
        id_rsa(私钥600权限)  id_rsa.pub(公钥644权限)
        推荐使用命令模式，因为通常在脚本里实现
        ssh-keygen -t rsa -P "" -f /root/.ssh/id_rsa 
        
        2.将生成的公钥拷贝到对方的家目录/root/.ssh/下
        通过ssh内部的copy命令
            ssh-copy-id -i /root/.ssh/id_rsa.pub 192.168.34.101
                默认放到对方/root/.ssh下生成authorized_keys文件
        
        备注：如果是scp拷贝过去，需要该名称成authorized_keys
            或者scp -p /root/.ssh/id_rsa.pub 192.168.34.101：/root/.ssh/
###作用：由于是免密码登录的，可以批量在多台机器上创建用户等其他操作
    ssh 192.168.34.101 useradd haha
    ssh 192.168.34.101 userdel -r  haha
    当然可以写一个脚本，在每一台机器上都创建一个haha用户
        touch iplist.txt
        while read ip;do
                ssh $ip useradd haha
        done < iplist.txt 

####基于key验证方式作用：
    可以直接通过命令批量执行一些操作如：
        ssh 192.168.34.101 getent passwd
        可以直接查看对方主机上的用户信息

        1.对比一下写法，要加双引号，才能执行
            ssh 192.168.34.101 echo > /root/.ssh/authorized_keys
            ssh 192.168.34.101 "echo > /root/.ssh/authorized_keys"
        2.扩展：由于服务器一般都把ssh默认端口22改成其他的,如9527，如何将公钥拷贝
            ssh-copy-id -i /root/.ssh/id_rsa.pub "root@192.168.34.101 -p 9527"
            ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.34.101 -p 9527
        只能通过上面加双引号的写法实现拷贝，当成是一个整体


### 因为当前主机可以连接所有的机器,那么私钥的安全性很重要，所以建议对私钥设置对称秘钥
    ssh-keygen -p 默认对id_rsa私钥进行加密，此时私钥即被加密
    再通过当前主机管理其他机器时，都需要输入私钥的密码
    此时可以通过ssh的代理程序来托管私钥的密码，不用每次都输入密码
        运行ssh-agent bash，ssh-add 输入密码即可
        但是退出重新登录，代理需要重新设定

## xshell&securtCRT如何通过key验证登录
    以xshell为例:

## 扩展1.如何把之前管理过的100台机器的key删除
    ip=192.168.34.
    for((i=1;i<=100;i++));do
        if [ -f /root/.ssh/authorized_keys ];then
            ssh $ip$i "echo > /root/.ssh/authorized_keys"
        fi                                                                
    done
## 扩展2.给100台主机都基于key验证-脚本
iplist.sh先取出要基础key验证的网内主机IP

    ip=192.168.34.
    for ((i=1;i<=254;i++));do
            {
           if ping -c1 -w1 $ip$i &> /dev/null;then
                   echo $ip$i >> /data/iplist.txt
           fi
           }&
    done
    wait

copykey.sh

    user=test
    password=test123
    ssh-keygen -t rsa -P "" -f /root/.ssh/id_rsa
    while read ip;do
    expect <<EOF
    set timeout 10
    spawn ssh-copy-id -i /root/.ssh/id_rsa.pub $user@$ip
    expect {
    "yes/no" { send "yes\n";exp_continue }
    "password" { send "$password\n" }
    }
    expect eof
    EOF
    done < /data/iplist.txt 

## 扩展3.如何为了集群中的主机之间通信方便，互相基于key验证
    1.在一台机器上生成一对公钥私钥
        ssh-keygen -t rsa -P "" -f /root/.ssh/id_rsa
    2.ssh 当前主机ip，从而生成authorized_keys文件
    3.把当前主机上的.ssh目录复制到其他机器上
        scp -rp /root/.ssh 192.168.34.$i:/root/
    即集群内的所有机器都使用一套公钥私钥


## scp命令(使用的是ssh协议)
    scp命令：使用的ssh协议
        scp [options] SRC... DEST/
    两种方式：
        scp [options] [user@]host:/sourcefile /destpath
        scp [options] /sourcefile [user@]host:/destpath
    常用选项：
        -C 压缩数据流
        -r 递归复制
        -p 保持原文件的属性信息
        -q 静默模式
        -P PORT 指明remote host的监听的端口
    也可以使用跨主机拷贝文件的效果，类似ssh -t跨主机连接主机
        scp f1.txt 192.168.34.2 192.168.34.3 192.168.34.4:/data
    扩展：在A主机上，将B主机文件拷贝到C主机上
        scp 192.168.34.B:/etc/fstab 192.168.34.C:/data/

# rsync命令：sync是同步的意思,和拷贝是有区别的
####windows里很多对比文件夹不同，再拷贝都是基于同步的原理
    基于ssh和rsh服务实现高效率的远程系统之间复制文件
    使用安全的shell连接做为传输方式
        • rsync –av /etc server1:/tmp 复制目录和目录下文件
        • rsync –av /etc/ server1:/tmp 只复制目录下文件
        比scp更快，只复制不同的文件
    选项：
        -n 模拟复制过程
        -v 显示详细过程
        -r 递归复制目录树
        -p 保留权限
        -t 保留时间戳
        -g 保留组信息
        -o 保留所有者信息
        -l 将软链接文件本身进行复制（默认）
        -L 将软链接文件指向的文件复制
        -a 存档，相当于–rlptgoD，但不保留ACL（-A）和SELinux属性（-X）
## 对比scp&rsync的区别
    1.区别1：以拷贝目录为例：
        scp不区别目录后的/，默认就是拷贝这个目录，而rsync不同
        rsync -r /data/bin 192.168.34.103:/data/lianxi
            不带/是将/bin整个目录拷贝
        rsync -r /data/bin/ 192.168.34.103:/data/lianxi
            带/是将/bin下的文件拷贝，不拷贝目录
    2.如果目录中的文件更新了，scp&rsync的区别
        scp -r /data/bin/  192.168.34.A:/data
        默认会把全部文件再拷贝一次，占用系统资源
        rsync -r /data/bin/  192.168.34.A:/data
        只会把对应目录下变化的文件进行拷贝更新，没变化的不操作



==============================================================================================================================================================

#SSH端口的转发功能，也就是ssh隧道（又包括本地端口转发，远程端口，动态转发）

#### 应用场景：SSH协议是加密的，但是生产中很多服务都是不加密的，很不安全，但是可以让其他服务跑在ssh协议之上，将其他服务协议封装一层ssh协议，在网络中传输的即是使用ssh封装过的其他协议流量，把ssh协议内的数据当成是其他服务自身，这种方式叫做ssh隧道(端口转发)功能

#####类似于http跑在ssl/tls之上，http封装tls加密协议，同理如telnet跑在ssh之上，开机ssh端口转发(隧道)功能

### 怎么去使用ssh的隧道功能，实用场景下怎么去使用呢？



#SSH本地端口转发实验原理：
    本地转发：
     -L localport:remotehost:remotehostport sshserver
    选项：
        -f 后台启用
        -N 不打开远程shell，处于等待状态
        -g 启用网关功能
    示例：ssh -L 9527:192.168.34.101：23 -Nf 192.168.34.105后台执行
        关闭隧道：killall ssh

下文以不安全的telnet服务为例，封装ssh协议进行传输，达到加密安全传输效果
![SSH本地端口转发]()

实验1：以上图为例centos7为公司内部的主机，centos6-min为内部网络ssh服务器，
以centos6为内部网络telnet服务器，还需要自己在外网的一台笔记本

    centos6上安装开启telnet步骤，在centos6上telnet是非独立服务，需要被xinetd开启的
        步骤：
            1.yum install telnet 安装客户端
            1.yum install telnet-server,安装服务端,同时默认安装xinetd服务
            2.chkconfig telnet on
            3.service xinetd start；然后ss -ntl:telnet23端口即被打开了
    centos7上安装开启telnet步骤
        步骤：
            1.yum install telnet
            2.systemctl start telnet-socket
    备注：telnet协议连接，登录是不能使用管理员用户的，必须使用普通用户
### 实验前提及步骤：
    centos7模拟主机，centos6-min为模拟中间的ssh服务器，centos6模拟telnet服务端
    在centos6上iptables -A INPUT -s 192.168.34.101 -j REJECT拒绝centos7的直接连接

    步骤：
    1.外网的笔记本先通过公网teamviwer连接到公司内部的主机7，
    2.连接上去之后，所有操作都是在centos7主机上执行：
        ssh -L 9527:192.168.34.101：23 -Nf 192.168.34.105   -Nf后台执行
        意思是：当前主机开机9527端口，通过192.168.34.105当跳板，连接到192.168.34.101：23的telnet服务,搭建一个桥梁，在后台执行，killall ssh关闭后台隧道
    3.可以在三台机器上通过ss -nt看到：
        ss -nt查看7和6-min之间有ssh连接状态，6-min和6之间还没有telnet流量，
        7上ss -ntl可以看到 9527端口是打开并监听的
    4.7通过telnet连接自己的9527端口时，默认会连到6上去了，实现了跳板连接
        执行telnet 127.0.0.1 9527后，直接提示输入6的用户和密码，达到端口转发功能
        7上端口状态：
        6-min上端口状态
        6上端口状态

![ss连接状态]()


#### 实验总结：通过上面的实验结果可以看出7主机确实telnet通过6-min当跳板连接到6上了，但是本地端口转发很有局限性，一般中间会有防火墙隔断，而且7主机必须是公司内部网的一台机器，如果在家或者不在公司如何去连接到6的telnet服务器，还必须先把自己的笔记本连到7的主机上去，但是主机7并不是自己的公司内部电脑，没有权限去连接，而且会有防火墙，那么如何解决这个问题？那就需要用到远程端口转发来解决。


==============================================================================================================================================================


# SSH远程端口转发：
    远程转发:
     -R sshserverport:remotehost:remotehostport sshserver
    示例：
    ssh –R 9527:telnetsrv:23 –N sshsrv

## 需求场景：不在公司时,如果需要连到telnet的服务器上，如何实现安全连接？(区别实验1)
![SSH远程端口转发]()

### 实验前提及步骤：
    以上图为例centos7为外网笔记本，centos6-min为内部网络ssh服务器，
    centos6为内部网络telnet服务器，实现跨防火墙，且不借助笔记本先连接到内网机器.
    在6上iptables -A INPUT -s 192.168.34.101 -j REJECT模拟拒绝centos7的直接连接
    
    步骤：
        1.全部操作在centos6-min上，即内网的ssh服务器主动连接笔记本
            ssh -R 9527:192.168.34.101:23 -Nf 192.168.34.103
        意思是：-R，是在centos7(103)笔记本上开启9527端口,把自己(ssh客户端)当跳板，连接到centos6(192.168.34.101：23)的telnet服务,搭建一个桥梁，在后台执行，killall ssh关闭后台隧道，centos7上自动断开与centos6的telnet连接.

        2.然后再笔记本上centos7上(192.168.34.103)，执行
            telnet 127.0.0.1 9527
            即可连接到centos6上，实现了在自己的笔记本上直接操作
        3.各个主机ss -nt连接状态图如下
![ss-nt连接状态]()


### 实验结果：对比实验1可以发现实验2可以直接跨过防火墙进行外网连接，进而实现端口转发功能，避免了需要先连到公司内网机器，而且避免在防火墙上开启一个22端口才能连接的危险操作，实际情况下，SSH的远程端口转发更有安全性。


### 实验3：如果在左侧还有一台主机和笔记本是一个网段的，也需要连接telnet服务器，如何在centos6-min上开机9527端口？
    1.可以在centos6-min上执行
        ssh -R 9527:192.168.34.101:23 -gNf 192.168.34.103
        开启192.168.34.103(centos7)上的网关功能，即所有机器都可以通过9527端口连接
    2.但是有个前提：centos6-min的ssh_config配置文件需要把Gateway port no 一行注释删掉，或改成Gateway port yes
    3.其他主机执行
        telnet 192.168.34.7(笔记本) 9527
    其他主机连接笔记本的9527端口也可以直接连接到centos6(telnet服务器)上

==============================================================================================================================================================


# SSH动态端口转发原理
    动态端口转发：
    当用firefox访问internet时，本机的1080端口做为代理服务器，firefox的访问
    请求被转发到sshserver上，由sshserver替之访问internet
    ssh -D 1080 root@sshserver
     在本机firefox设置代理socket proxy:127.0.0.1:1080
     curl --socks5 127.0.0.1:1080 http://www.qq.com

## 实验：fq实现浏览国外学术网站
![浏览学术网站原理]()

## 实验前提及步骤：
    以上图为例centos7为国内机器，centos6-min为购买的国外虚拟ssh服务器
    centos6为国外学术网站
    步骤：
        1.在centos7(192.168.34.103)上建立指向虚拟服务器(centos6-min)的SSH连接
            ssh -D 1080 192.168.34.105 -fN
        2.然后打开firefox或者google浏览器，网络代理设置里Socks项,把代理挤进去
            socks项配置： 127.0.0.1 1080
        也可以通过命令行方式配置代理：
            curl --socks5 192.168.34.105:1080
        然后本机可以实现科学上网了
        如果局域网内其他机器也要连接，其他winows电脑代理地址配置centos7服务器的就行了
        socks项：192.168.34.103 1080,直接通过centos7去访问，把7的IP设置成网关


# SSH服务端重要配置文件之(sshd_config文件)
    服务器端：sshd, 配置文件: /etc/ssh/sshd_config
    sshd_config文件重要项：
        Port   建议修改默认ssh端口为不常用的端口如：22022
        ListenAddress ip   本机有公网和内网地址,可以设置只绑定在内网地址，只允许内网连接
            listenaddress 192.168.34.7
        SyslogFacility AUTHPRIV   把ssh登录消息记录在/var/log/secure下
            可以分析secure日志，把失败登录的IP取出来
        LoginGraceTime 2m       登录时，等待密码输入的等待时长，超过2分钟，自动关闭
        PermitRootLogin yes     默认允许root用户登录，买的云服务默认不允许root登录
            可以改成 PermitRootLogin no
        StrictModes yes             检查
        MaxAuthTries 6              最大的尝试连接次
        MaxSessions 10      一个连接允许连接会话，如xshell连接一次，最多允许克隆10次会话
        PubkeyAuthentication yes  默认支持公钥验证
        PermitEmptyPasswords no   禁止空口令登录
        PasswordAuthentication yes    默认运行登录口令验证
            可改为：PasswordAuthentication no 不允许口令验证，只支持key验证
        GatewayPorts no
        ClientAliveInterval：单位:秒 设置多长时间不操作自动断开连接
        ClientAliveCountMax：默认3 设置检查次数
        UseDNS yes                  会将IP反向解析成名字
            一般要改成UseDNS no
        GSSAPIAuthentication yes 提高速度可改为no
        MaxStartups  未认证连接最大值，默认值10
        Banner /path/file     登录提示文件，等同于/etc/motd文件
        限制可登录用户的办法：
            AllowUsers user1 user2 user3   只允许哪些用户连接
            DenyUsers      用户黑名单
            AllowGroups    允许哪个组的用户连接
            DenyGroups     组黑名单

##### 为了ssh的登录安全，可以对下面项进行设置
    建议使用非默认端口
    禁止使用protocol version 1
    限制可登录用户
    设定空闲会话超时时长
    利用防火墙设置ssh访问策略
    仅监听特定的IP地址
    基于口令认证时，使用强密码策略
        tr -dc A-Za-z0-9_ < /dev/urandom | head -c 30| xargs
    使用基于密钥的认证
    禁止使用空密码
    禁止root用户直接登录
    限制ssh的访问频度和并发在线数
    经常分析日志


#PSSH
### 基于ssh协议的key验证的工具PSSH(在key验证的基础上使用更加方便)
#### pssh适用于主机数不多的情况下，如果主机比较多还是使用专业的管理软件ansible,puppet
#### PSSH语法
    pssh是一个python编写可以在多台服务器上执行命令的工具，也可实现文件复制
    选项如下：
        --version：查看版本
        -h：主机文件列表，内容格式”[user@]host[:port]”
        -H：主机字符串，内容格式”[user@]host[:port]”
        -A：手动输入密码模式
        -i：每个服务器内部处理信息输出
        -l：登录使用的用户名
        -p：并发的线程数【可选】
        -o：输出的文件目录【可选】
        -e：错误输入文件【可选】
        -t：TIMEOUT 超时时间设置，0无限制【可选】
        -O：SSH的选项
        -P：打印出服务器返回信息
        -v：详细模式

## 使用示例：
    1.如果没有基于key验证：需要使用-A，-H -I,选项,输入IP，用户名和密码
        pssh -H 192.168.34.2 -A -i hostname

    2.批量创建用户nginx，先将ip存放在文本中，iplist.txt,代替了脚本实现
        pssh -h iplist.txt -i useradd nginx

    3.将查到root用户信息保存到/data/pssh目录下,生成IP对应的文件列表
        pssh -h iplist.txt -o /data/pssh -i getent passwd root

    4.批量删除iplist.txt中的/data下数据
        pssh -h iplist.txt -i 'rm -rf /data/*' 此处需要加单引号引起来，双引号都不行

    5.批量显示iplist.txt主机名
        pssh -h iplist.txt -i 'echo $HOSTNAME' 此处需要加单引号引起来，双引号都不行

    6.批量显示iplist.txt下的selinux是否关闭了,如果没关闭，则发送命令批量关闭
        显示：pssh -h iplist.txt -i '/etc/selinux/config'
        关闭：pssh -h iplist.txt -i 'sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/sysconfig/selinux'
    7.批量安装软件
        pssh -h iplist.txt -i  'yum install pssh -y'

    8.批量解压keepalived软件
        pssh -h iplist.txt -i 'tar -zxvf keepalived-1.2.19.tar.gz'

## PSCP.PSSH命令：pssh的批量复制命令
    PSCP语法：
    pscp.pssh功能是将本地文件批量复制到远程主机
    pscp [-vAr] [-h hosts_file] [-H [user@]host[:port]] [-l user] [-p par] [-o
    outdir] [-e errdir] [-t timeout] [-O options] [-x args] [-X arg] local remote
    Pscp-pssh选项
        -v 显示复制过程
        -r 递归复制目录
###pscp.pssh使用示例：
    因为 pssh不支持把当前主机上的脚本，在iplist.txt中执行，所以需要先拷贝
        1.把当前的test.sh批量复制到其他主机iplist.txt的/data目录下
            pscp.pssh -h iplist.txt test.sh /data/
        2.批量执行test.sh脚本
            pssh -h iplist.txt -i '/data/test.sh'
        3.把本地目录批量复制到多台主机iplist.txt的/data目录下
            pscp.pssh -h iplist.txt -r /data/test/ /data/

## PSLURP命令：把多台主机的文件全部拉取(复制)到当前主机上
    pslurp功能是将远程主机的文件批量复制到本地
    pslurp [-vAr] [-h hosts_file] [-H [user@]host[:port]] [-l user] [-p par][-o
    outdir] [-e errdir] [-t timeout] [-O options] [-x args] [-X arg] [-L localdir]
    remote local（本地名）
    Pslurp选项
        -L 指定从远程主机下载到本机的存储的目录，local是下载到本地后的名称
        -r 递归复制目录
### pslurp使用示例：
    1.把多台主机的日志文件复制到当前主机并改名为log.bak
        pslurp -h iplist.txt -L /data/var/ /var/log/messages log.bak
        备注：把各个主机/var/log/messages日志，全部复制到/data/var/下并改名log.bak
    
    可以查看/data/var/的树结构，是以IP名为目录下
        [root@node7-1 data]#tree /data/var/
        /data/var/
        ├── 192.168.34.101
        │   └── log.bak
        └── 192.168.34.105
            └── log.bak
### pnuke    并行在远程主机杀进程
    1.批量杀死httpd进程
        pnuke -h  iplist.txt  httpd
### prsync   使用rsync协议从本地计算机同步到远程主机
    1.将当前主机的/data/test目录同步到多台主机的/tmp目录下
         prsync -h iplist.txt -l root -a -r  /data/test /tmp/

#### PSSH在主机数量不多的时候使用非常方便，更多功能，日后遇到时，再更新





#AIDE
    作用：
        入侵检测工具，，主要用途是检查文件的完整性，审计计算机上的那些文件被更改过了。
    检查过程：
        AIDE能够构造一个指定文件的数据库，它使用aide.conf作为其配置文件。AIDE数据库
    能够保存文件的各种属性，包括：权限(permission)、索引节点序号(inode number)、
    所属用户(user)、所属用户组(group)、文件大小、最后修改时间(mtime)、创建时间
    (ctime)、最后访问时间(atime)、增加的大小以及连接数。AIDE还能够使用下列算法：
    sha1、md5、rmd160、tiger，以密文形式建立每个文件的校验码或散列号.
        这个数据库不应该保存那些经常变动的文件信息，例如：日志文件、邮件、/proc文件
    系统、用户起始目录以及临时目录

### AIDE的安装和使用
    安装
        yum install aide
    选项：
        -i, --init       初始化aide数据库
        -C, --check       检查现有数据属性和数据库记录的是否一样
        -u, --update      Check and update the database non-interactively
          --compare     Compare two databases
        -v, --version     版本信息以及配置文件路径

    Miscellaneous:
      -D, --config-check    Test the configuration file
      -v, --version     Show version of AIDE and compilation options
      -h, --help        Show this help message

###需要先配置监控规则
    系统中可以定义监控的选项：
        # These are the default rules.
        #
        #p:      permissions
        #i:      inode:
        #n:      number of links
        #u:      user
        #g:      group
        #s:      size
        #b:      block count
        #m:      mtime
        #a:      atime
        #c:      ctime
        #S:      check for growing size
        #acl:           Access Control Lists
        #selinux        SELinux security context
        #xattrs:        Extended file attributes
        #md5:    md5 checksum
        #sha1:   sha1 checksum
        #sha256:        sha256 checksum
        #sha512:        sha512 checksum
        #rmd160: rmd160 checksum
        #tiger:  tiger checksum

        #haval:  haval checksum (MHASH only)
        #gost:   gost checksum (MHASH only)
        #crc32:  crc32 checksum (MHASH only)
        #whirlpool:     whirlpool checksum (MHASH only)

    1.比如先定义要监控哪些属性的集合
        TEST=p+u+g+s+md5
    2.定义好监控属性集合后，可以自定义监控目录或文件
        如监控/data下除了f2的全部文件
        /data/ TEST
        !/data/f2
    3.定义完监控规则后，收集要监控数据的属性信息，存到aide数据库中
        aide --init 
        会保存/var/lib/aide.db.new.gz中
    4.然后比较当前系统中监控的目录属性信息是否发生变化
        aide -C








    


