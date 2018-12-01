#DNS服务
#####实现DNS主流服务器端软件BIND(著名的Inter系统协会组织：研发的DHCP和DNS技术)

    C/S结构：服务端和客户端，实现DNS的软件有很多，最多的是BIND DNS
    1.名字解析  dns是做什么的？
    2.DNS服务工作原理   --->
    3.实现主服务器  --->也叫正向解析，搭建主DNS服务
    4.实现从(备)服务器  --->实现DNS主从复制，搭建从/备DNS服务
    5.实现反向解析区域 --->反向解析，搭建DNS反向解析服务器
    6.实现子域     --->父域和子域，搭建子域
    7.实现转发     ---> DNS的转发功能：搭建DNS的局部转发和全局转发
    7.实现只能DNS --->CDN内容分发网络，搭建CDN分发
    8.编译安装    --->编译安装BIND
    9.DNS排错
    10.搭建互联网架构的DNS网络综合实验 --->搭建根域，顶级域，二级域

###写在前面：
####看完本章应该理解一个常识：
    1.IP地址是否能够访问和主机有没有连互联网有关和DNS无关
    2.在浏览器上访问域名不通，和本机有没有连互联网没有明确的关系，即使在脸上互联网的情
      况下，也会不通，因为配置的DNS的有关，DNS是专门把域名解析成IP的，访问不了域名很大
      程度上是因为DNS服务器出现了故障。
    3.linux主机没有DNS缓存的功能（linux下的DNS服务器是有缓存的），windows主机是有缓存功能的
    4.DNS只负责解析，通不通是网络的事情，就能看出网络通不通和DNS是否能解析是两码事
        [root@node6-1 data]#ping blog.haha.com
        PING blog.haha.com (7.7.7.7) 56(84) bytes of data.
       从这个执行结果可以看出，域名blog.haha.com是已经解析成对应的7.7.7.7IP地址
        当前的DNS解析是没有问题的，只是网络连接不正常
###DNS排错
    #dig A example.com
        ; <<>> DiG 9.9.4-RedHat-9.9.4-14.el7 <<>> A example.com
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30523
        ...
        SERVFAIL:The nameserver encountered a problem while processing the query.
        • 可使用dig +trace排错，可能是网络和防火墙导致
    NXDOMAIN：The queried name does not exist in the zone.
        • 可能是CNAME对应的A记录不存在导致
    REFUSED：The nameserver refused the client's DNS request due to policy
    restrictions.
        • 可能是DNS策略导致
    NOERROR不代表没有问题，也可以是过时的记录
    查看是否为权威记录，flags:aa标记判断
    被删除的记录仍能返回结果，可能是因为*记录存在
    如：*.example.com． IN A 172.25.254.254
    注意“.”的使用
    避免CNAME指向CNAME记录，可能产生回环
        test.example.com. IN CNAME lab.example.com.
        lab.example.com. IN CNAME test.example.com.
    正确配置PTR记录，许多服务依赖PTR，如sshd,MTA
    正确配置轮询round-robin记录
####DNS是做什么的？
    1.当主机不多时，通过IP地址访问是合适的
    2.但是当在稍微大一点的网络或者互联网上进行访问时，通过IP地址访问是不现实的，
        所以一般都是通过名字(FQDN)来访问的，因此就要把名字转换成真正的IP地址，进而进行访问而这个把名字解析成IP的功能就是DNS的名字解析。

####DNS服务
    DNS:Domain Name Service 应用层协议
        C/S结构,53/udp, 53/tcp
            tcp/53端口管理同步功能；udp/53端口管理查询功能
    BIND：Bekerley Internat Name Domain
        ISC （www.isc.org）
    本地名称解析配置文件：hosts
        linux下：/etc/hosts
        windows下：C：/system32/drivers/etc/hosts

####FQDN&DNS域名
    FQDN:
        FQDN：主机名/别名+域名，如：www.baidu.com
        域名：又可通过.来分割多个部分：即下面所说的DNS域名(根域，一级域名，二级域名等)
    根域. --->只是一个点
    一级域名：也叫顶级域名
            com, edu, mil, gov, net, org, int,arpa
            三类：组织域、国家域(.cn, .ca, .hk, .tw)、反向域
    二级域名
    三级域名
    最多127级域名
    
    DNS域名总结：
        NS服务器实际上记录的是主机和IP地址的对应关系。2.下级域DNS和IP地址
    如：
        1.根域的DNS服务器下记录的是顶级域和他对应的DNS数据库服务器
        2.顶级域的DNS服务器记录的是二级域名和他对应的DNS数据库服务器
        3.二级域名的DNS服务器记录的是FQDN和他真正的IP地址，
            当然也包其子域和他对应的DNS数据库服务器
            即二级域名下是包括真正的主机和他子域的数据库服务器
        4.以此类推

####DNS解析
    DNS查询类型：
        递归查询：客户端向DNS查询，负责到底的方式
        迭代查询：只负责告知查询的下一级dns位置，不负责查询具体的对应关系的方式
    名称服务器：域内负责解析本域内的名称的主机
    根服务器：13组服务器(在后面的实验中会有具体体现)
    解析类型：
        FQDN --> IP  -将域名解析成IP地址
        IP --> FQDN  -将IP地址解析成域名
    注意：正反向解析是两个不同的名称空间，是两棵不同的解析树


#####实现DNS功能的主流软件--->BIND
可以yum install -y bind安装
也可以进行编译安装：[编译安装bind]()

    BIND的安装配置：
        dns服务程序包:bind
            tcp/53端口管理同步功能；udp/53端口管理查询功能
    服务名&程序名：named
    程序包：yum list all bind*  ---涉及的重要文件  
        /etc/named.conf  --->BIND主要的配置文件
        /etc/named.rfc1912.zones --->添加自定义域和数据库的文件
        /usr/lib/systemd/system/named.service  --->服务名称
        /usr/sbin/named        --->bind主程序
        /var/log/named.log    --->DNS的日志文件(通过rndc工具开启)
        /var/named/named.ca 安装时即带有互联网上著名的13个根服务器
        /var/named       --->记录名字和IP对应关系的数据库文件
    相关的文件
        bind-libs：相关库
        bind-utils:客户端 管理dns的工具包：dig,host,nslookup
        bind-chroot: /var/named/chroot/
        相关的文件
        bind-libs：相关库
        bind-utils:客户端 管理dns的工具包：dig,host,nslookup
        bind-chroot: /var/named/chroot/
            作用：，互联网上利用dns的漏洞namd账号，从而控制整个linux系统，为了降低
            DNS被攻击产生的影响，从而会把整个DNS的所有文件搬到chroot目录下，
            即使DNS，被攻击了，也只会影响当前的chrrot目录，不会影响整个linux系统

###相关命令：dig、rndc、host、nslookup(windows可以通过此命令查询)
    测试命令dig
        dig [-t type] name [@SERVER] [query options]
        dig只用于测试dns系统，不会查询hosts文件进行解析
        查询选项：
            +[no]trace：跟踪解析过程 : dig +trace magedu.com
            +[no]recurse：进行递归解析
        测试反向解析：
            dig -x IP = dig –t ptr reverseip.in-addr.arpa
                dig -x 192.168.34.103
        模拟区域传送：
            dig -t axfr ZONE_NAME @SERVER 从DNS服务器上获取域信息
            dig -t axfr magedu.com @10.10.10.11 
            dig –t axfr 100.1.10.in-addr.arpa @172.16.1.1
            dig -t NS . @114.114.114.114  查询NS资源记录
            dig -t NS . @a.root-servers.net 
            dig -t MX haha.com  查看是否搭建自己的邮件服务器
####rndc命令:来自于bind安装包
    rndc：可以实现DNS的多种控制
        rndc --> rndc (953/tcp)
    rndc COMMAND
    COMMAND:
        reload: 重载主配置文件和区域解析库文件
            每次修改配置文件执行rndc reload
        reload zonename: 重载区域解析库文件
        retransfer zonename: 手动启动区域传送，而不管序列号是否增加
        notify zonename: 重新对区域传送发通知
        reconfig: 重载主配置文件
        querylog: 开启或关闭查询日志文件/var/log/message
            rndc querylog 启用日志功能
        trace: 递增debug一个级别
        trace LEVEL: 指定使用的级别
        notrace：将调试级别设置为 0
        flush：清空DNS服务器的所有缓存记录，做实验时记得要每次清空DNS缓存信息
        rndc status可以看出日志是关闭的

=============================================================================
================================================

#####搭建一个只缓存服务器：通过实验1和实验2来解析原理(第一类DNS服务器)，只提供DNS缓存功能，本身没有配置区域信息，将查询的域名通过根域取查询(DNS转发的一种形式)

####实验1：在centos7上搭建一个DNS服务器，然后centos6把DNS配置为centos7的IP地址
    步骤：
    centos7上：IP:192.168.34.103
        1.安装bind： yum -y install bind
        2.启动服务： systemctl start named
    centos6上：关闭桥接和NAT网卡
        1.在ifcfg-eth0配置文件添加
            DNS1=192.168.34.107
        2.service restart network  重启网络
        3.cat /etc/resolv.conf  可以看到生效的DNS只有192.168.34.103
        4.ping www.baidu.com   -测试ping百度的域名：不通
        5.ping 180.97.33.107(事先知道的百度的一个地址)，结果是可以的
        6.dig www.baidu.com 通过DNS的专业工具，测试是否能解析IP：不可以
从下图可以看出：1.不能解析出百度的IP；2.但是可以ping通百度的IP地址，3.dig 查询提示DNS服务不可达
    可以得出结论，网通，DNS解析错误，也会出现不能上网的提示
![ping百度域名不通IP通.png](https://upload-images.jianshu.io/upload_images/8783576-52478a72bb3db55d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




####结论：从上面的实验可以理解在日常工作，有时会发现直接访问www.baidu.com是不通的，但是输入IP正常访问，qq也能正常使用；这就说明了DNS不能解析百度的地址，但网络是正常的，出现的问题有可能是工作区的DNS服务出现了问题。

=============================================================================
=============================================================================


###实验2：基于实验1的基础:实现centos6能够ping百度，只需要修改配置文件即可
    实验1中centos6ping不通的原因：
        因为Centos7上搭建的DNS服务，将DNS53端口的绑定在了本机(127.0.0.1)上，只对本机提供DNS服务，因此centos6上肯定是不能解析出百度的IP的.
        listen-on port 53 { 127.0.0.1; }将DNS服务的端口只绑定在本地IP上了。

    修改配置文件，让centos6能够上网：
        为了可以让centos6也可以解析出百度的IP地址，就需要修改centos7的DNS配置文件，将
        /etc/named.conf的listen-on port 53 { 127.0.0.1; }改成localhost或者注释掉
        重启DNS服务,即所有配置本机DNS的主机都可以通过本机进行域名解析 
        systemctl restart named 或者 rndc reload

    备注：
        即使centos7上的DNS只能对自己提供服务，在ifcfg-ens33的配置文件中，也一定要把
        DNS写成：DNS1=127.0.0.1，因为53的端口在上图中是绑定在127.0.0.1上的.



named-checkconf可以检查named.conf配置文件是否有语法错误

然后：在centos6上，执行ping www.baidu.com则可以ping通，说明centos7上的DNS解析是正确的。
那我们来想一下这个解析过程：
        6(配置的DNS指向7(192.168.34.103))-->7的DNS收到后-->发现本机的DNS区域数据库里没有www.baidu.com域名--->然后7通过询问根服务器-->一级一级的DNS数据库查找,获取到www.baidu.com域名对应的IP，从而再告诉6百度的域名真正对应的IP地址，然后解析出来，ping通了

![修改7上DNS配置文件使6上能够解析百度.png](https://upload-images.jianshu.io/upload_images/8783576-fb104d8d5c2da03f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####总结：实验1，实验2，只是搭建实现了一个只缓存服务器：即在本机的DNS服务器没有的域名，则通过去询问根，来获取所要解析的域名.实现基本的DNS缓存功能。



=============================================================================
=============================================================================


####实验3：Centos7上搭建一个实现解析xxxxxx.haha.com的权威服务器，也叫主DNS服务器(正向的DNS服务器)(第二类DNS服务器)

    步骤：
        1.修改vim /etc/named.rfc1912.zones文件，添加自己的要维护哪个域的信息
            域的类型(主/备)和对应的区域数据库文件
            例如：
                zone "haha.com" {
                    type master;
                    file "haha.com.zone";
                };
        2.进入/var/named/目录下，创建 haha.com.zone数据库文件
            要注意该文件是640权限，所属组是named
            -rw-r----- 1 root  named  230 Nov 25 20:41 haha.com.zone
        3.编辑haha.com.zone的文件内容编辑
        4.检查区域数据库文件格式的语法和conf配置文件的语法
            named-checkconf可以检查named.conf配置文件是否有语法错误
            named-checkzone haha.com /var/named/haha.com.zone
                此命令是检查对应的域和他的数据库文件是不是有语法错误
        5.重启DNS服务：
            可以用systemctl restart named (建议重启服务)
            或者rndc reload:重新加载DNS配置文件
![Centos7上搭建正向主DNS服务器（一）.png](https://upload-images.jianshu.io/upload_images/8783576-0d5392da858232aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![基于centos7正向主DNS的Centos6测试访问.png](https://upload-images.jianshu.io/upload_images/8783576-490fc3cf8cf33abc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#####要搭建DNS服务器(权威服务器)就需要创建区域数据库文件，下面介绍区域数据库的格式写法
#####对于区域数据库文件来说必须要有的三条记录
            SOA记录 (有且只有一条)
            NS记录： 当前区域的某DNS服务器的名字
                    一个区域可以有多个NS记录
            一条解析DNS服务本身的A记录：
                服务名对应的IP(即哪些服务器作为DNS服务器的IP地址)
        

######区域数据库文件内的格式和各项信息详解:
    a.文件可以copy -a /var/named/named.localhost当模板(保留属性)，在模板上进行修改
    b.也可以手写区域数据库文件，但是要修改属性信息：
        -rw-r----- 1 root  named  220 Nov 27 17:03 haha.com.zone
        所属组named；权限640

    下面介绍文件具体格式和给选哪个代表的意义：
        1.每一行都是由资源记录组成
        2.而各种资源记录都由5项组成组成 (有些项可省略-除了第一条记录)
        3.资源记录格式：、
            由：name [TTL] IN  rr_type（记录类型） value（IP）--->5项组成

            @/name：代表本域，代表本区域数据库文件管理的是哪个域，也可以写成                 @=haha.com.
            TTL：time to live 这条记录是否缓存到客户端/服务端的有效时长
                    (继承上面的有效期)
            IN：inter 固定格式(可不写)
            rr_type：资源记录的类型，SOA必须放在整个资源记录的第一条，有且只有一个，           叫启始授权记录
           value（IP）： 
                SOA记录比较特殊：后面一串都为第一条资源记录的的值，也叫键值对，
                而A，AAAA，NS等记录，没有这一串，只有IP地址

####资源记录的类型(SOA、NS、A、AAAA、PTR、MX)
    区域解析库：由众多RR组成：
        资源记录：Resource Record, RR
        记录类型：A, AAAA, PTR, SOA, NS, CNAME, MX
    SOA：Start Of Authority，起始授权记录；一个区域解析库有且仅能有一个
    SOA记录，必须位于解析库的第一条记录
    A：internet Address，作用，FQDN --> IP
    AAAA：FQDN --> IPv6
    PTR：PoinTeR，IP --> FQDN
    NS：Name Server，专用于标明当前区域的DNS服务器
    CNAME ： Canonical Name，别名记录
    MX：Mail eXchanger，邮件交换器
    TXT：对域名进行标识和说明的一种方式，一般做验证记录时会使用此项，如：
    SPF（反垃圾邮件）记录，https验证等

#####SOA记录书写格式
        下面是SOA记录的独特书写格式：
        SOA记录的值包括：(区域数据库必须有的，有且只有一个，且只有第一条资源记录会有)
            SOA记录的各项信息：
                @ rname.invalid. (
                                                0       ; serial
                                                1D      ; refresh
                                                1H      ; retry
                                                1W      ; expire
                                                3H )    ; minimum
        SOA记录包括三大项： 
            第一项：@：要表示出谁是维护haha.com.域的主DNS服务的名(也叫IP)：DNS服务器的名称
                    @如果换成dns1,则一定需要在下面通过A记录把dns1解析成IP
                            dns1 A 192.168.34.103
            第二项：rname.invalid.：表示这个DNS服务器管理员的邮箱
                    格式；admin.haha.com. 或者admin. (此处的.代表=@=haha.com.资源记录的第一项信息)
            第三项：实现当有从/备服务器时，同步主从DNS服务器的策略信息
                0       ; serial  -通过数据库的序列号，触发主DNS的推和备服务器的拉操作
                1D      ; refresh -备DNS周期性主动拉取主DNS数据的时间间隔，可以自定义
                1H      ; retry -备服务器拉取失败时，尝试再去拉取信息的时间间隔
                1W      ; expire -备DNS拉取信息失败后，一周后，备DNS服务的信息就将过期，不能再通过备DNS进行查询了
                3H )    ; minimum：通过当前DNS服务器查询时，当没有这个信息时，会把
                    这条查询的DNS记录保存到缓存DNS中，再次查询时，则通过缓存DNS告知，
                    而不是再次询问当前的DNS服务器，避免了此DNS服务器的压力，次数代表的是保存到缓存DNS的时长。

#####NS(name server)记录：
    1.区域数据库必须有的记录
    2.写上当前haha.com.域到底有几个DNS服务器，和分别都是谁
    3.NS就是体现当前区域数据库都是哪些DNS服务器，对于haha.com.域来说都有哪些DNS服务器
        对于本域来说，谁是我的DNS服务器
            主DNS，从/备DNS，第三个DNS
        可以省略@ 等，继承上一条资源记录，把不同资源类型改成NS就行了
        @ IN NS dns1
        dns1 192.168.34.103

#####A记录：把fqdn解析成ipv4地址
    问：在互联网上还有可能域名输入错误时，
        1.不输入(www.)baidu.com，jd.com,时也可以正确跳转到百度网页 
        2.域名输入错误wwww.baidu.com;xxxx.jd.com也可以正确跳转，如何实现的？ 
        3.批量生成很多重复的域名对应的IP

    主机FQDN对应的IPV4地址
    或者SOA记录中的dns1,把DNS服务器的本机地址/或者备DNS服务器的地址解析出来
    避免用户写错名称时给错误答案，可通过泛域名解析进行解析至某特定地址(后面有示例)    
    name: 某主机的FQDN，例如www.magedu.com.
    value: 主机名对应主机的IP地址
    例如：
        dns1 A 192.168.34.103
        $GENERATE 1-254 HOST$ A 1.2.3.$
        *.magedu.com. IN A 5.5.5.5 （泛域名解析）
![特殊的DNS解析设置.png](https://upload-images.jianshu.io/upload_images/8783576-e9eef41f17a4e0d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



######AAAA记录：主机FQDN对应的IPV6地址
    把fqdn解析成IPv6地址

######CNAME记录:别名技术
    别名记录:CNAME，为什么要用到别名记录？
        小网站一般都是A记录，大网站一般都是别名记录，因为有些网站访问量大
            需要用到CDN网络商进行加速
    例如：
        给www起个别名：
        www CNAME websrv   先定义成别名
        websrv A 6.6.6.6   再将别名解析成IP
        websrv A 8.8.8.8   一个别名记录也可以解析成多个IP
        websrv A 9.9.9.    当然一个别名记录也可以解析成多个IP
    作用：
        别名一般都是一些CDN服务商：内容分发网络的服务商
        作用：有些网站访问量比较大，为了能够加速访问速度,如提供给海外游戏网站，电商网站等）通过第三方的服务商通过CDN进行加速访问。
    下文会介绍什么是CDN？         
            将加速的网站推送到距离用户最近的DNS服务器上去，从而达到加速目的）
    下文介绍分发网络和如何搭建？
            （CDN服务商一般会在全国甚至全世界都有自己的DNS服务器）

#####PTR：反向解析记录
    将IP地址解析成 --> FQDN域名
    后面有实验

#####MX记录
    邮件资源记录
        如邮箱地址：test@haha.com
    邮件收发的原理：(下图有示例)
        1.事先配置一个smtp邮件服务器的地址 先到达此机器
            如： @ MX 10 mailsrvs
                mailsrvs A 10.10.10.10
        2.不在一个邮件服务器上：则会查我们配置的(haha.com)区域数据库上MX记录，如果有
            则传到对应的IP地址,进而再转发给test用户;没有MX地址，或者不存在的邮件地址，则会提示错误：邮件不可达，这就是发邮件是否能收到的原理.
    通过dig -t mx baidu.com命令可以查看是否搭建自己的邮件服务器
![MX资源记录.png](https://upload-images.jianshu.io/upload_images/8783576-78f256d4125cc3ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


=============================================================================
=============================================================================

###搭建反向解析DNS
##实验4：Centos7上搭建一个实现反向解析haha.com.域的权威服务器(PTR资源记录)
    搭建反向解析DNS服务器
    步骤：
        1. vim /etc/named.conf 和正向解析的一样就行：注释掉两行
        2. vim /etc/named.rfc1912.zones 再添加反向解析的区域数据库
            要写网段到解析名字的数据库文件，格式如下
            zone "34.168.192.in-addr.arpa" {
                    type master;
                    file "192.168.34.zone";                               
            };
        3.创建反向解析数据库文件：192.168.34.zone
            vim /var/named/192.168.34.zone,
        4.named-checkconf检查反向解析数据语法和格式有无错误
            named-checkzone 34.168.192.in-addr.arpa /var/named/192.168.34.zone
![Centos7搭建DNS反向解析(一).png](https://upload-images.jianshu.io/upload_images/8783576-b94787231f7666e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Centos7搭建DNS反向解析(二).png](https://upload-images.jianshu.io/upload_images/8783576-f04202c480cc672f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



=============================================================================
=============================================================================

###搭建主从DNS服务器
####实验5：搭建一个从/备DNS服务器(第三类DNS服务器)
######以之前的Centos7为主DNS服务器，在Centos7-min上搭建从DNS服务
    步骤：
        从服务器：centos7-min上：
            1.yum -y install bind
            2.vim /etc/named.conf
                注释掉绑定端口那两行
                添加不允许其他主机同步区域数据库文件的代码
                allow-transfer {none;};
            3.vim /etc/named.rfc1912.zones 添加从域的信息
        主DNS服务器上：
            1.vim /etc/named.conf
                添加只允许备的IP进行数据库文件传输控制代码
                allow-transfer {备DNS的IP;};
![搭建主从DNS服务.png](https://upload-images.jianshu.io/upload_images/8783576-d888fe8197fdca0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####DNS主从复制功能测试：
    1.修改主DNS的serial,执行rndc reload，重新加载主DNS服务，主DNS数据会自动
        把数据同步到备DNS/var/named/slaves/下
    2.删除备DNS的slave/下的备份文件，重启
        备DNS systemctl restart named，备份文件又会自动生成
    3.删除备DNS的slave/下的备份文件，关闭主DNS服务器的tcp53端口，
    重启主DNS服务，发现备DNS的数据没有同步说明TCP53端口是管理同步功能的（主从复制的）
    4.关闭主DNS的UDP53，（提供查询的），必须开放的


=============================================================================
=============================================================================

###搭建子域
###实验6：搭建haha.com.的子域xixi.haha.com.
    方式一：可以在之前的centos7主DNS上再搭建xixi.haha.com子域，把父域子域放在一台主机
        步骤：
        1.vim /etc/named.rfc1912.zones，在父域haha.com域配置后面加上
                xixi.haha.com的子域信息
            zone "xixi.haha.com" {
            type master;
            file "xixi.haha.com.zone";                                         
            };
        2.vim xixi.haha.com.zone，创建子域区域数据库文件
            $TTL 1D
            @       IN SOA  dns1 admin. ( 403   1D 1H  1W  3H )  
                    NS      dns1
            dns1    A      192.168.34.103
            blog.xixi.haha.com. A 7.7.7.7
            www    CNAME    websrv
            websrv  A      33.33.33.33     

    方式二：在另外一台服务器上搭建xixi.haha.com子域，然后再主DNS：haha.com的区域数据库文件中把子域的域名和IP进行解析进行了
        在centos6-mini2搭建xixi.haha.com.子域，主DNS还是用原来的centos7
        centos6上步骤：
            1.yum -y install bind
            2.vim /etc/named.rfc1912.zones
                zone "xixi.haha.com" {
                    type master;
                    file "xixi.haha.com.zone";
                };
            3.vim xixi.haha.com.zone
                $TTL 1D
                @       IN SOA  dns1 admin. (
                                                        403     ; serial
                                                        1D      ; refresh
                                                        1H      ; retry
                                                        1W      ; expire
                                                        3H )    ; minimum
                        NS      dns1
                dns1    A      192.168.34.103
                blog.xixi.haha.com. A 7.7.8.8
                www    CNAME    websrv
                websrv  A      33.33.9.9
            4.centos7上的区域数据库里加上xixi.haha.com.子域的NS记录和A解析的IP
![基于主DNS搭建xixi.haha.com子域.png](https://upload-images.jianshu.io/upload_images/8783576-cbb8636b3718bad0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


=============================================================================
=============================================================================

###实验7：搭建DNS转发服务器(不是转发到根域上，而是转发设定的DNS服务器上)
    转发服务器：分两种转发
        only:本机DNS上没有，转发给指定的DNS服务器上，如果指定的那台也没有，则转发的那台主机DNS去向互联网上去寻找，而不是本机去互联网上寻找
        first:本机DNS没有，先转到另外一台指定的DNS，如果另外一台DNS也没有，本机去通过互联网的根上获取域名
    注意：被转发的服务器需要能够为请求者做递归，否则转发请求不予进行
        (1) 全局转发: 对非本机所负责解析区域的请求，全转发给指定的服务器
                Options {
                    forward first|only;
                    forwarders { ip;};
                };
        (2) 特定区域转发：仅转发对特定的区域的请求，比全局转发优先级高
            zone "ZONE_NAME" IN {
                type forward;
                forward first|only;
                forwarders { ip;};
            };
        注意：
                关闭dnssec功能：
                dnssec-enable no;
                dnssec-validation no;
![DNS的转发功能.png](https://upload-images.jianshu.io/upload_images/8783576-b139a2fef43c2d60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



=============================================================================
=============================================================================

##智能DNS之CDN(Content Delivery Network)
###实验8：搭建CDN分发网络
######bind中ACL;
    bind有四个内置的acl:
        none: 没有一个主机
        any: 任意主机
        localhost: 本机
        localnet: 本机的IP同掩码运算后得到的网络地址
        还可以自定义acl
    注意：只能先定义，后使用；因此一般定义在配置文件中，处于options的前面

######还需要用到view技术
    view:视图：实现智能DNS：
    一个bind服务器可定义多个view，每个view中可定义一个或多个zone
    每个view用来匹配一组客户端
    多个view内可能需要对同一个区域进行解析，但使用不同的区域解析库文件
    实现场景：用户在某个区域内，就把当前区域内(或者离得最近)的服务器上的
                网页信息返回给用户?
######搭建步骤：
        1.先生成各个地区的区域数据库文件，然后修改各自的websrv解析的各自的IP地址
            复制/etc/named.rfc1912.zones该名成3个地区的区域数据库文件
            haha.com.zone.bj  haha.com.zone.sh haha.com.zone.cd
                192.168.34.0   172.18.0.0       其他的是成都
        2.再自定义acl列表
            /etc/named.conf下添加acl列表信息
        3.在/etc/named.conf中通过view技术把acl列表和各地区的区域数据库建立对应关系
![CDN网络之创建各地区数据库文件.png](https://upload-images.jianshu.io/upload_images/8783576-b419469c103655ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![CDN网络之修改区域数据库文件.png](https://upload-images.jianshu.io/upload_images/8783576-9016efc4365040f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![CDN网络之view绑定acl和区域数据库文件.png](https://upload-images.jianshu.io/upload_images/8783576-7c63d2a1bb897ea5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![CDN网络之测试是否成功.png](https://upload-images.jianshu.io/upload_images/8783576-fa90fa0692e733fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)









#####DNS相关备注
    根域服务器文件删除了如何恢复？
        可以通过执行dig命令得到根服务器的配置文件
        dig @114.114.114.114 > named.ca  只能生成13个根服务器的名字但是不能解析出根服务器对应的地址
           dig @a.root-server.net. > named.ca 通过随便一个
        根服务器生成解析的IP导入到name.ca中就可以了

    客户端软件
    通过key验证来实验控制named 
    /etc/named.root.key  /etc/named.iscdlv.key 
    记录了管理服务和客户端实验通讯的key
###综合实验：搭建一个互联网DNS(模拟根域、顶级域、二级域等)
######实验功能：模拟互联网，客户端发请求-->电信DNS服务器-->根-->com域-->主备haha.com.域-->www.haha.com网站
![搭建互联网DNS原理图.png](https://upload-images.jianshu.io/upload_images/8783576-7e558f9ddb77b2ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


    实验前提：根据原理图，需要7台服务器进行实验
        1.需要访问www.haha.com网站的主机(192.168.34.106)
            DNS地址需要配置工作区DNS的IP
        2.模拟工作区的DNS服务器(或者是模拟小区内的DNS服务器)(192.168.34.101)
            搭建DNS服务，并将默认的互联网根IP，改成模拟的根DNS的IP
        3.模拟根的DNS服务器(192.168.34.103)
            搭建根.域DNS服务
        4.模拟com.的顶级域的DNS服务器(192.168.34.107)
            搭建com.域DNS服务
        5.模拟haha.com.域的主DNS服务器(192.168.34.105)
            搭建haha.com.域主DNS服务，实现DNS主从复制
        6.模拟haha.com.域的备DNS服务器(192.168.34.104)
            搭建haha.com.域备DNS服务，实现DNS主从复制
        7.haha.com.域下的www.haha.com的网站(192.168.34.109)
            只需要搭建httpd服务
    备注：所有服务器关闭防火墙和selinux

下图是每台主机的具体配置图：倒叙排列
#####搭建www.haha.com的网站
![综合实验-主机7.png](https://upload-images.jianshu.io/upload_images/8783576-830852eb767707f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####搭建haha.com.域的主DNS服务器
![综合实验-搭建主DNS（二）.png](https://upload-images.jianshu.io/upload_images/8783576-a16c52eafff1c01e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![综合实验-搭建主DNS.png](https://upload-images.jianshu.io/upload_images/8783576-05095033842e0f36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####搭建haha.com.域的备DNS服务器
![综合实验-搭建备DNS.png](https://upload-images.jianshu.io/upload_images/8783576-c4105441499b4500.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####搭建com.的顶级域的DNS服务器
![综合实验-搭建com顶级域.png](https://upload-images.jianshu.io/upload_images/8783576-9ef5321bd0431f41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####搭建根的DNS服务器
![综合实验-搭建模拟根域.png](https://upload-images.jianshu.io/upload_images/8783576-f2add43c03dba5d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####搭建工作区的DNS服务器
![综合实验-搭建工作区的DNS.png](https://upload-images.jianshu.io/upload_images/8783576-7149b9db4a47a3f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####测试访问
![综合实验-主机测试是否成功.png](https://upload-images.jianshu.io/upload_images/8783576-82d3ffeb0ad27862.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
