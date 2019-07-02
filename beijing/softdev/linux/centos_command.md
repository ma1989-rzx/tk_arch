### 01-nmap
     参考网址：
           http://www.cnblogs.com/st-leslie/p/5115280.html	
           http://www.2cto.com/article/201203/125686.html 
     (1) 主机发现（Host Discovery）
	  -F：扫描100个最有可能开放的端口   
          -v 获取扫描的信息   
          -sT：采用的是TCP扫描 不写也是可以的，默认采用的就是TCP扫描
	  例子要求：获取http://nmap.org 的主机是否开启
		nmap -F -sT -v nmap.org	
		状态	详细的参数说明
		 Open	 端口开启，数据有到达主机，有程序在端口上监控
		 Closed	 端口关闭，数据有到达主机，没有程序在端口上监控
		 Filtered	 数据没有到达主机，返回的结果为空，数据被防火墙或者是IDS过滤
		 UnFiltered	 数据有到达主机，但是不能识别端口的当前状态
		 Open|Filtered	 端口没有返回值，主要发生在UDP、IP、FIN、NULL和Xmas扫描中
		 Closed|Filtered 只发生在IP ID idle扫描

    (2) 端口扫描（Port Scanning）
	nmap -F -sT -v -n 172.16.31.1-225
	1、TCP扫描（-sT）
	   这是一种最为普通的扫描方法，这种扫描方法的特点是：扫描的速度快，准确性高，
           对操作者没有权限上的要求， 但是容易被防火墙和IDS(防入侵系统)发现
	   运行的原理：通过建立TCP的三次握手连接来进行信息的传递
		① Client端发送SYN；
		② Server端返回SYN/ACK，表明端口开放；
		③ Client端返回ACK，表明连接已建立；
		④ Client端主动断开连接。		
	2、SYN扫描（-sS）
	    这是一种秘密的扫描方式之一，因为在SYN扫描中Client端和Server端没有形成3次握手，
            所以没有建立一个正常的TCP连接，因此不被防火墙和日志所记录，一般不会再目标主机上
            留下任何的痕迹，但是这种扫描是需要root权限（对于windows用户来说，是没有root权
            限这个概念的，root权限是linux的最高权限，对应windows的管理员权限）
	3、NULL扫描
	    NULL扫描是一种反向的扫描方法，通过发送一个没有任何标志位的数据包给服务器，然后
            等待服务器的返回内容。这种扫描的方法比前面提及的扫描方法要隐蔽很多，但是这种方
            法的准确度也是较低的， 主要的用途是用来判断操作系统是否为windows，因为windows
            不遵守RFC 793标准，不论端口是开启还是关闭的都返回RST包
	4、FIN扫描
	    FIN扫描的原理与NULL扫描的原理基本上是一样的在这里就不重复了		 
	5、ACK扫描
	    ACK扫描的原理是发送一个ACK包给目标主机，不论目标主机的端口是否开启，都会返回相应的RST包，
            通过判断RST包中的TTL来判断端口是否开启
	    TTL值小于64端口开启，大于64端口关闭		  
	6、自定义TCP扫描包的参数为（--scanflags）
	    例如：定制一个包含ACK扫描和SYN扫描的安装包
	    命令：nmap --scanflags ACKSYN nmap.org
	    -PS 端口列表用,隔开[tcp80 syn 扫描]
	    -PA 端口列表用,隔开[ack扫描](PS+PA测试状态包过滤防火墙)【默认扫描端口1-1024】
	    -PU 端口列表用,隔开[udp高端口扫描 穿越只过滤tcp的防火墙]
	    其他的常见命令
	       输出命令
		-oN 文件名 输出普通文件
		-oX 文件名 输出xml文件
	       错误调试：
		--log-errors 输出错误日志
		--packet-trace 获取从当前主机到目标主机的所有节点
		  备注：nmap ip地址/域名 支持CIDR.(连续的ip用-连接)【空选项主机存活、SYN端口】

	nmap可以检测出服务器开放的端口，也可以对端口进行分析及安全
	使用nmap来检测一下它的开放端口以及安全方面的情况，服务器IP  219.129.216.156
	    检测服务器1-1000的端口：
		# nmap -v -A 219.129.216.156
	    检测全部的端口，默认是TCP的：
		# nmap -v -A 219.129.216.156 -p  1-65535
	扫描例子参考：http://blog.csdn.net/findmyself_for_world/article/details/50521810
	扫描指定IP段开放的端口示例：
                nmap -sS -p 443 -oG - 172.30.154.0/24 | grep open | awk '{print $2}'	
	路由跟踪: nmap -v -A  39.106.57.186	监视到达39.106.57.186服务器所经过的路由器	
	masscan用法
	    参考地址： 
                https://blog.csdn.net/hackerie/article/details/78651725    
                http://www.4hou.com/tools/8251.html	   		   
	     CIDR地址扫描：（Classless Inter-Domain Routing，无类域间路由选择）
                    masscan 172.30.154.0/24 -p80
	     IPv4地址扫描： masscan 10.0.0.1-10.0.0.233 -p80
		扫描262142个IP，用时3分钟左右。 443： 29416 、80：76551
                # 从文件中读取CIDR地址，以2000毫秒速率扫描443端口，并将其写入到文件
		       masscan -iL conf/ip.txt -p443 --rate=2000 -oJ tmp/tempResult 
                # 扫描CIDR地址172.30.154.0/24，以2000毫秒速率扫描80或443端口并将其写入到文件 
		      masscan 172.30.154.0/24 -p80,443 --rate=2000 -oJ tmp/tmpResult_test    
			文件命名方式:  10-24_80.txt
                # 速度提高到每秒1000万，扫描所有443端口的互联网		
		      masscan 0.0.0.0/0 -p443 -rate 10000000
                # 速度提高到每秒1000万，扫描所有端口的互联网  			 
		      masscan 0.0.0.0/0 -p0-65535 -rate 10000000	
     10000,000个数据包/秒，扫描整个互联网（减去排除）每个端口大约10个小时，
     100,000个数据包/秒，占用带宽平均在30kb左右，峰值84kb
     1000,000个数据包/秒，占用带宽平均在40kb左右，峰值90kb，大约45个小时(两天)， 
     
     **备注**：在阿里弹性云服务器上	1核1G内存1M带宽， 以速率rate=2000扫描80端口
	rate=2000	网络带宽出网稳定在100KB/s  最高145KB/s   总共1MB=1024KB
	rate*6 = 12000	速率可调整到12000值。之前配置为1000000 按这估计得500M带宽；					
     参数：
	‐‐resume paused.conf   恢复扫描
	-oX filename：输出到filename的XML。
	-oG filename：输出到filename在的grepable格式。
	-oJ filename：输出到filename在JSON格式。
	-iL filename：从文件读取输入。
	‐‐exclude filename：在命令行中排除网络。
	‐‐excludefile filename：从文件中排除网络。
	-S：欺骗源IP。
	-v interface：详细输出。
	-vv interface：非常冗长的输出。
	-e interface：使用指定的接口。

	4277075968主机(去除私网地址段)  ，用时多久
        使用tcpdump捕获所有网络流量，监视的流量包使用wireshark打开，网卡通过ifconfig获取		
	tcpdump -i eno16777736 -w /opt/text.pcap	

### 02-查看Linux的CPU、内存、版本、硬盘信息
    (1) 查看CPU信息: #cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c			
    (2) 查看内存:#free -g |grep "Mem" | awk '{print $2}'		
    (3) 查看cpu是32位还是64位
	   # getconf LONG_BIT
	   # echo $HOSTTYPE
	   # uname -a
    (4) 查看当前linux的版本:#cat /etc/issue | grep Linux				
    (5) 查看内核版本
	  # uname -r
	  # uname -a
	  # lsb_release -a   Ubuntu系统查看版本      
          # cat /etc/redhat-release    CentOS系统查看版本
    (6) 查看硬盘:#fdisk -l
### 03-iptables命令(防火墙  [CentOS 7]farewall)
	基本操作
		service iptables [status|start|restart|stop|save]
	(1) 添加数据规则
		iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
	
	CentOS7使用 farewall作为防火墙
		查看farewall
			firewall-cmd --state			# not running 
		搜索farewall服务
			systemctl list-unit-files|grep firewalld.service   
		关闭farewall
			systemctl stop firewalld.service

### 04-rpm命令
	概念:RPM软件包的管理工具。rpm原本是Red Hat Linux发行版专门用来管理Linux各项套件的程序，由于它遵循GPL规则且功能强大方便，
	     因而广受欢迎。逐渐受到其他发行版的采用。RPM套件管理方式的出现，让Linux易于安装，升级，间接提升了Linux的适用度。
	选项：
		-a：查询所有套件； 
		-b<完成阶段><套件档>+或-t <完成阶段><套件档>+：设置包装套件的完成阶段，并指定套件档的文件名称； 
		-c：只列出组态配置文件，本参数需配合"-l"参数使用； 
		-d：只列出文本文件，本参数需配合"-l"参数使用； 
		-e<套件档>或--erase<套件档>：删除指定的套件； 
		-f<文件>+：查询拥有指定文件的套件； 
		-h或--hash：套件安装时列出标记； 
		-i：显示套件的相关信息； 
		-i<套件档>或--install<套件档>：安装指定的套件档； 
		-l：显示套件的文件列表； 
		-p<套件档>+：查询指定的RPM套件档； 
		-q：使用询问模式，当遇到任何问题时，rpm指令会先询问用户； 
		-R：显示套件的关联性信息； 
		-s：显示文件状态，本参数需配合"-l"参数使用； 
		-U<套件档>或--upgrade<套件档>：升级指定的套件档； 
		-v：显示指令执行过程； 
		-vv：详细显示指令执行过程，便于排错。
	(1) 查询
		#rpm -qa | grep your-package
	(2) 安装
		# rpm -ivh your-package.rpm
	(3) 卸载rpm软件包
		# rpm -e your-package.rpm
	
	例如：卸载CentOS7自带的Java
		# 查询Java
			rpm -qa | grep java
		# 卸载以java-开头的装包
			rpm -e --nodeps java-xxxx

### 05-chkconfig命令
	概念：检查、设置系统的各种服务。这是Red Hat公司遵循GPL规则所开发的程序，它可查询操作系统在每一个执行等级中会执行哪些系统服务，
	      其中包括各类常驻服务。谨记chkconfig不是立即自动禁止或激活一个服务，它只是简单的改变了符号连接。
	参考网址：http://man.linuxde.net/chkconfig
	设置开机启动
		chkconfig mysql on

### 06-systemctl命令(CentOS7有)
	概念：系统服务管理器新指令，它实际上将 service 和 chkconfig 这两个命令组合到一起。
	参考网址：http://man.linuxde.net/systemctl
	示例：
		systemctl enable httpd.service	#使某服务自动启动
		systemctl disable httpd.service #使某服务不自动启动
		systemctl status httpd.service  #检查服务状态
		systemctl start|stop|restart httpd.service   #启动|停止|重启某服务

### 07-crontab指令(定时任务)
	概念：定时执行任务命令
	参数：
		*　　*　　*　　*　　*　　command 
		分　时　日　月　周　命令 
	示例：
		crontab  -e 编辑crontab服务文件
		crontab  -l 查看该用户下的crontab服务是否创建成功
		ps -ax | grep cron 查看服务是否已经运行
	
	vim /etc/crontab 与crontab -e 编辑任务的区别？
		【使用范围】
		修改/etc/crontab这种方法只有root用户能用，这种方法更加方便与直接直接给其他用户设置计划任务，而且还可以指定执行shell等等，
		crontab -e这种所有用户都可以使用，普通用户也只能为自己设置计划任务。然后自动写入/var/spool/cron/usename
	
	CentOS: service crond restart 重启crond

### 08-showmount命令
	概念：查看挂载信息
	示例：
		showmount -e 192.168.40.128

### 09-mount命令(挂载磁盘)
	概念：用于加载文件系统到指定的加载点。此命令的最常用于挂载cdrom，使我们可以访问cdrom中的数据，因为你将光盘插入cdrom中，
	      Linux并不会自动挂载，必须使用Linux mount命令来手动完成挂载。
	选项：
		-V：显示程序版本； 
		-l：显示已加载的文件系统列表； 
		-h：显示帮助信息并退出； 
		-v：冗长模式，输出指令执行的详细信息； 
		-n：加载没有写入文件“/etc/mtab”中的文件系统；
		-r：将文件系统加载为只读模式； 
		-a：加载文件“/etc/fstab”中描述的所有文件系统。
	示例：
		mount -t nfs 192.168.248.208:/home/nfs /home/nfs  #mount挂载服务器端的目录/home/nfs到客户端某个目录

### 10，Linux查看某一端口占用情况：lsof -i:port
	114DNS服务地址（国内第一个、全球第三个开放DNS地址）：114.114.114.114
	Google的域名服务器：8.8.8.8
	poc:Proof of Concept的缩写,意思是为观点提供证据，证明漏洞的样本。

	查看文件的命令有：cat、more、less、head、sed和tail等。
	例： 
	查看文件的前5行：  head -5 test.log 
	查看文件的后2行： tail -2 test.log  或 tail -n 2 test.log
	查看文件中间一段：sed -n '5,10p' test.log

### 11，mktemp:可以轻松在/temp文件夹中创建一个唯一的临时文件
	mktemp:可以轻松在/temp文件夹中创建一个唯一的临时文件
	-t : 在临时目录下创建临时文件
	-d : 在临时目录下创建临时目录

	tee:向文件中写数据，像管道的T管道一样
	默认重新写入会覆盖原先数据
	-a : 向文件添加数据

	捕获信号
	trap:指定能够通过shell脚本监控和拦截Linux信号。如果脚本收到在trap命令中列出的信号，将保护该信号不被shell处理，并在本地处理它。
	
	以后台模式运行shell脚本，只要在命令后加一个&符号

	nohup:不使用控制台的情况下运行脚本		nohup ./test1 &
	nohup.out:进程与任何终端断开后，自动将STDOUT和STDERR消息重定向nohup.out文件

	脚本使用$$变量显示Linux系统分配给脚本的PID

	jobs： 查看当前作业
	带+(加号)的作业被视为默认作业
	-(减号)的作业是在处理完当前默认作业之后将成为默认作业的作业
	没有指定作业编号，它是任何作业控制命令引用的作业

	无论shell运行多好个作业，某一时间点，只能有一个带有加号的作业，和一个带有减号的作业

	bg:以后台模式启动的作业
	fg:以前台模式启动作业

	调度优先级(整数)：-20(最高优先级)~20(最低优先级)，默认情况下，bash shell 启动所有优先级为0的进程,(优先级越高越先执行)
	nice:更改作业优先级
	renice:更改在系统中运行的命令优先级
		限制：
			只能对拥有的进程使用renice命令。
			只能使用renice命令将进程调至更低的优先级。
			根用户可以使用renice命令将任何进程调至任何优先级
		如果要完全控制运行进程，则必须以根账户登录
	
	batch:安排脚本在系统负载（使用率）低时运行

	cron:可定期运行的作业，在后台运行，从特殊表格(cron 表格)中查询需要调度运行的作业
		crontab ：创建cron表格  -l 参数

	anacron:使用时间戳确定调度的作业是否在正确的时间间隔运行


	w/who:查看已登录用户信息

	pkill -kill -t pts/2  :踢出pts/2终端的用户

	sudo shutdown -h now

	netstat -tunlp | grep 3306  查看3306端口被占用情况
	lsof -i:3306		    查看3306端口被占用情况
	
	netstat -at | grep 2181
	netstat -nat  查看所有端口

	od -d elasticsearch.properties 
	od -c elasticsearch.properties 已ASC码形式查看文件
	od -b elasticsearch.properties 

	注意：（UTF-8 + BOM） 创建的.txt文件会在文件的开头加上一个看不见的字符编码。

### 12，awk: 是一个强大的文本分析工具， 简单的说，awk就是把文件逐行读入，以空格为默认分隔符将每行切片。
	有3个版本：awk、nawk、gawk，未作特别说明，一般指gawk
	awk: Alfred Aho、Peter Weinberger、Brian Kernighan 姓氏首字母
	样式扫描和处理语言
	使用方法：awk '{pattern + action}'{filenames}
		pttern:表示AWK在数据中查找的内容
		action:在找到匹配内容时所执行的一系列命令
	调用方式：
		1，命令行方式
		2，shell脚本方式
		3，将所有的awk命令插入一个单独的文件，然后调用
	参考地址：
		http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html
		https://www.cnblogs.com/emanlee/p/3327576.html

	批量杀死相同的进程：ps -ef | grep phantomjs |grep -v grep | awk '{print "kill -9 " $2}' | sh

	筛选过滤：
		cat result.txt | grep '-' | awk -F '\t' '$4<0 && $4>-1000 { print $1"\t"$2"\t"$3"\t"$4}' > minScore.txt
		awk -F '\t' '$4 > -1000 && $4 < 0' result.txt  >  minScore.txt
		cat site.txt | awk -F "\r" '{print "UPDATE `ct_site_finder` SET  site_templet =1,confidence_level=100 where domain=\""$1"\";" }' > rs.txt

### 13，初始登录，设置系统根用户名密码
	sudo passwd root  #输入两次重置的密码

### 14，Centos更改默认启动桌面（或命令行）模式
	vim /etc/inittab
		# Default runlevel. The runlevels used are:
		#   0 - halt (Do NOT set initdefault to this)
		#   1 - Single user mode
		#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
		#   3 - Full multiuser mode(命令行模式)
		#   4 - unused
		#   5 - X11	(图形模式)
		#   6 - reboot (Do NOT set initdefault to this)
		id:3:initdefault:
	将“id:5:initdefault:” 中的5修改为3，就会从图形模式转变为命令行模式，减少对内存的占用。

	CentOS 7配置命令行启动
		cat /etc/inittab	查看配置文件
		systemctl get-default //获取当前系统启动模式
		systemctl set-default graphical.target	由命令行模式更改为图形界面模式
		systemctl set-default multi-user.target	由图形界面模式更改为命令行模式

### 15，Linux设置系统启动时间为
	参考地址：https://www.cnblogs.com/vagabond/p/6808315.html
	1，tzselect (选择时区)
		5) Asia
		9) China   
		1) Beijing Time
		Yes
	2，修改本地 localtime 文件
		rm -rf /etc/localtime
		ln -s /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
	
	3，ntpdate & ntpd
		yum -y install ntpdate ntp
		同步在线网络s
		ntpdate  xxxx
		 
	   crontab: 每十分钟调整一次时间;
		*/10 * * * * ntpdate time.nist.gov   #域名或IP
		ntpdate -u s2c.time.edu.cn	时间同步
		配合crond来保证时间一致
			crontab  -e -u root  #编辑root用户crontab配置，添加下列情况
			0 1 * * * /usr/sbin/ntpdate s2c.time.edu.cn
			crontab  -l -u root	 #查看是否成功

### 16，Top查看系统监控统计情况
	参考地址：http://blog.csdn.net/dxl342/article/details/53507673

### 17，bc 计算器
	https://jingyan.baidu.com/article/1e5468f974a17a484861b770.html

### 18，wc命令
	语法：wc [选项] 文件…
	说明：该命令统计给定文件中的行数、字数、字节数。如果没有给出文件名，则从标准输入读取。wc同时也给出所有指定文件的总统计数。字是由空格字符区分开的最大字符串。
	该命令各选项含义如下：
	    -c 统计字节数。
	　　-l 统计行数。
	　　-w 统计字数。
	输出列的顺序和数目不受选项的顺序和数目的影响。总是按下述顺序显示并且每项最多一列。
		行数、字数、字节数、文件名
	如果命令行中没有文件名，则输出中不出现文件名。

### 19，网络带宽监视命令
	nethogs:显示每个进程所使用的带宽，并对列表排序，将耗用带宽最多的进程排在最上面。【可行】
			https://www.cnblogs.com/hanyifeng/p/5439520.html
			yum install nethogs -y 
		查看某网卡占用的流量
			nethogs eno16777736	#eno16777736为网卡名，可以用过ifconfig获取
			
	vnstat：
		默认只会显示自守护进程运行以来所传输的数据总量。
		-l 实时监控带宽使用情况
		yum install vnstat 

		 https://www.91yun.co/archives/3363

		 step01: 创建监控数据库：vnstat -u -i eth0
		 step02: 启动服务并设置开机启动
				service vnstat start
				chkconfig vnstat on
		 step03: 流量查看命令
			 vnstat -d ：看每天的流量统计命令
			 vnstat -m ：看每月的流量统计命令
	
	vmstat: 系统的CPU,内存,IO的使用情况
		vmstat 2   : 两秒采集一次服务器状态
### 20，war包解压命令
	jar -xvf xxx.war  : 将xx.war包解压到当前目录
### 21，用户操作
	查看所有用户	cat /etc/passwd
	查看所有用户组	cat /etc/group
	删除用户组	groupdel uuser
	创建用户uuser			useradd uuser  (CentOS用此命令创建用户后，/home目录会会自动出现一个该用户的目录)
	给已创建的用户uuser设置密码	passwd	uuser  (注意：用户必须通过此命令方式设置密码后，ssh才能登陆)
	删除用户			userdel uuser
	删除用户/home/uuser所在目录	rm -rf /home/uuser
### 22，硬链接和软连接区别
	硬链接创建只能针对file的  inode号相同，文件名不同，事实上指向的是同一区块数据，修改文件a，b的内容也会被修改。
	软链接创建可以针对dir和file ，创建后inode不同，新链接是新文件，会自动解析成原文件路径，原文件删除，指向地址的链接就会无效，
		ln -s XX-SNAPSNOT.jar xx.jar

### 23，linux上执行脚本时出现$'\r':command not found异常，经发现是脚本文件是dos格式，需要转换为unix格式？
	第一种方式：dos2unix
		yum -y install dos2unix 
		dos2unix load.sh 
	第二种方式：Vim
		vim dos.txt
		:set fileformat=unix	或 :set ff=unix
		:w

		删除^M的方法： 
			:%s//r//g 
### 24，查看当前系统发布版本命令
	$ cat /etc/os-release 
### 25，拷贝文件时带进度条命令
	pv 
### 26，nohup与&的作用
	使用&后台运行程序： ./a.out& 后台运行程序	
		结果会输出到终端
		使用Ctrl + C发送SIGINT信号，程序免疫
		关闭session发送SIGHUP信号，程序关闭
	使用nohup运行程序：nohup ./a.out 
		结果默认会输出到nohup.out
		使用Ctrl + C发送SIGINT信号，程序关闭
		关闭session发送SIGHUP信号，程序免疫
	平日线上经常使用nohup和&配合来启动程序：用nohup./a.out &运行程序
		同时免疫SIGINT和SIGHUP信号
	同时，还有一个最佳实践：
		不要将信息输出到终端标准输出，标准错误输出，而要用日志组件将信息记录到日志里

### 27，SIGINT、SIGQUIT、 SIGTERM、SIGSTOP区别
	参考地址：https://www.cnblogs.com/dzhs/p/5388092.html
	1) SIGINT
		程序终止(interrupt)信号, 在用户键入INTR字符(通常是Ctrl-C)时发出，用于通知前台进程组终止进程。
	2) SIGQUIT
		和SIGINT类似, 但由QUIT字符(通常是Ctrl-\)来控制. 进程在因收到SIGQUIT退出时会产生core文件, 在这个意义上类似于一个程序错误信号。
	3) SIGTERM
		程序结束(terminate)信号, 与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出，shell命令kill缺省产生这个信号。如果进程终止不了，我们才会尝试SIGKILL。
	4) SIGSTOP
		停止(stopped)进程的执行. 注意它和terminate以及interrupt的区别:该进程还未结束, 只是暂停执行. 本信号不能被阻塞, 处理或忽略.

### 28，yum命令：
	yum remove PackageKit	#解除yum包安装被占用

### 29，查看DNS域名是否生效命令(linux/win 都可使用)
	nslookup 域名   显示IP
	cat /etc/hosts		#查看hosts文件
	cat /etc/resolv.conf	#查看服务器的DNS方法

	yum  provides  */nslookup    就可以找到提供nslookup命令的软件包了
	linux下提供nslookup命令的软件就是 bind-utils
	yum install -y bind-utils


### 30，Centos7查看网关命令
	traceroute baidu.com
	IP段含义：https://blog.csdn.net/gatieme/article/details/50989257

### 31，uptime/top 查看负载均衡命令
	load average: 0.09, 0.05, 0.01
　　	三个数分别代表不同时间段的系统平均负载（一分钟、五 分钟、以及十五分钟），它们的数字当然是越小越好。数字越高，说明服务器的负载越大，这也可能是服务器出现某种问题的信号。
	参考地址：http://blog.chinaunix.net/uid-687654-id-2075858.html
		https://blog.csdn.net/lj1314ailj/article/details/73550097
		https://blog.csdn.net/csCrazybing/article/details/79160496
	如果占用cpu较高，可以cat /proc/pid/stack看看堆栈
	进程上下文频繁切换导致load average过高：http://www.361way.com/linux-context-switch/5131.html

	
	linux下用top命令查看cpu利用率超过100%
		top命令显示的是占用的cpu总数
		8cpu时top下cpu利用率最大可以到达800%。
		如果你的进程利用了多个cpu，那么top命令显示的是多个cpu占用率的总和。
		所以top命令下查看到的cpu利用率是可能超过100%的。


### 32，CentOS rz/sz命令安装方式
	yum install -y lrzsz
	ps: rz/sz 通过Zmodem协议传输数据,速度大概为10KB/s，适合中小文件。

### 33，pstree 查看进程列表


### 34，linux内存交换命令
	检查内存时，主要查看used项
	swapoff -a  关闭交换区
	swapon -a  打开交换区
	
	sync	强制被改变的内容立刻写入磁盘，更新超块信息，以防止释放，sync命令则可用来强制将内存缓冲区中的数据立即写入磁盘中。 

### 35，java.net.SocketException: Too many open files 系统并发量太大超过linux设置的默认值1024,引起的问题。
	(1) 查看系统设置
		ulimit -a
		通过以上命令，我们可以看到open files 的最大数为1024
	(2) 解决方法
		修改可以打开最大文件描述符的数量
		ulimit -n 4096	
		ulimit -n 4096 命令只能临时的改变open files  的值，当重新登陆后又会恢复，所以需要永久设置open files 的值。
		修改/etc/security/limits.conf文件改变
		* soft nofile 65535
		* hard nofile 65535
	(3) lsof -p [进程ID] 可以看到某ID的打开文件状况。

	Linux每个进程的开启的最大线程为1000  

### 36，linux中,查看某个进程打开的文件数?
	lsof -p [进程ID] | wc -l

### 37，host命令
	host （选项） （参数）	主机： 指定要查询信息的主机信息
	-a : 显示详细的DNS信息	
	第一种方法：是用resolv.conf中定义的DNS服务器查出百度主机的IP。
		host -a www.baidu.com
	第二种方法：是用谷歌的DNS（8.8.8.8）来查百度主机的IP。
		host -a www.baidu.com  8.8.8.8

### 38，hostnamectl 是在 centos7 中新增加的命令，它是用来修改主机名称的，centos7 修改主机名称会比以往容易许多
	status                 显示当前主机名设置
	set-hostname NAME      设置系统主机名
	set-icon-name NAME     为主机设置icon名
	set-chassis NAME       设置主机平台类型名

	静态的（static）、瞬态的（transient）、和灵活的（pretty）。
	静态主机名也称为内核主机名，是系统在启动时从/etc/hostname内自动初始化的主机名。
	瞬态主机名是在系统运行时临时分配的主机名。
	灵活主机名则允许使用特殊字符的主机名。

	修改主机名： echo "node242" > /etc/hostname

### 39，离线安装RPM包方法
	step01: 找一台相同版本的虚拟机服务器
	step02: 使用rpm -e xx卸载相应安装应用
	step03: 使用rpm命令下载应用安装依赖包
		rpm -y install xxx --downloadonly --downloaddir  /opt/temp
	step03: 执行后，在/opt/temp目录下找到需要安装的rpm包。

### 40，文件解压命令
	*.Z           #    compress程序压缩产生的文件(现在很少使用)
	*.gz          //    gzip程序压缩产生的文件
	*.bz2         //    bzip2程序压缩产生的文件
	*.zip　　　　　//　　 zip压缩文件
	*.rar　　　　　//　　 rar压缩文件
	*.7z　　　　　　//　　7-zip压缩文件
	*.tar         //    tar程序打包产生的文件
	*.tar.gz      //    由tar程序打包并由gzip程序压缩产生的文件
	*.tar.bz2     //    由tar程序打包并由bzip2程序压缩产生的文件

	# *.tar.bz2结尾文件
		tar -jxvf  *.tar.bz2  
	# *.tar.bz结尾文件
		tar -zxvf  *.tar.gz
	# *.tar结尾文件
		tar -xvf  *.tar
	#目录压缩命令
		tar -zcvf  xxx.tar.gz  xxxx
	
	#phantomjs 命令快照截图  (yum install bitmap-fonts bitmap-fonts-cjk)
		./bin/phantomjs  ../examples/rasterize.js https://www.baidu.com/   baidu.com.png
	# 查看phantomjs版本
		phantomjs -v
		
	gzip命令
		-c　　　　　　　//将输出写至标准输出，并保持原文件不变
		-d　　　　　　　//进行解压操作
		-v　　　　　　　//输出压缩/解压的文件名和压缩比等信息
		-digit　　　　 //digit部分为数字(1-9)，代表压缩速度，digit越小，则压缩速度越快，但压缩效果越差，digit越大，则压缩速度越慢，压缩效果越好。默认为6.
		
		注意，使用 gzip 指令压缩/解压文件均会使得源文件消失，即源文件会被直接解压/压缩而不保留备份。若想要保留原文件可以使用 -c 参数结合数据流重定向操作(见下例)。
		gzip exp1.txt exp2.txt　　　　 //分别将exp1.txt和exp2.txt压缩，且不保留原文件。注意对于多个文件参数是将多个文件分别进行压缩，而不是压缩在一起。参考下文 tar 指令。
		gzip -dv exp1.gz　　　　　　 //将exp1.gz解压，并显示压缩比等信息。
		gzip -cd exp1.gz > exp.1　　  //将exp1.gz解压的结果放置在文件exp.1中，并且原压缩文件exp1.gz不会消失

### 41，screen命令 命令行终端切换
	screen -S xxx
	screen -ls
	
	screen -S yourname -> 新建一个叫yourname的session
	screen -ls -> 列出当前所有的session
	screen -r yourname -> 回到yourname这个session
	screen -d yourname -> 远程detach某个session(关闭session)
	screen -d -r yourname -> 结束当前session并回到yourname这个session

### 42，sed 命令
	https://www.cnblogs.com/ctaixw/p/5860221.html
	sed命令可以进行字符串的批量替换操作(https://www.cnblogs.com/coffy/p/5607913.html)
	sed -i "s/oldstring/newstring/g" `grep oldstring -rl path`	#递归查询path目录文件中包含oldstring的文件，将文件中oldstring替换为newstring
	例如：sed -i "s/6/sk/g" `grep 6 -rl /home/work/test`	# 递归搜索/home/work/test文件中包含6的行，将每行中的6替换为sk
	      sed -i "s/6/sk/g" `grep 6 -rl /home/work/test/*.sh`	# 递归搜索/home/work/test/下*.sh的文件中包含6的行，将每行中的6替换为sk

	      sed -ie 's/gamesmart.com/baidu.com/g' test9.txt    #递归将 test9.txt文件中gamesmart.com字符全部替换为baidu.com
	      参考地址：https://www.cnblogs.com/ginvip/p/6376049.html
	      (1) 清除以空格开头的文本行
			sed -ie 's/^[ ]*//' testsed.txt		或者	sed -ie 's/^[[:space:]]*//' testsed.txt
	      (2) 删除文本中空行和空格组成的行及#号注释的行 
			grep -Eiv "^#|^$" testsed.txt
	      (3) 删除所有行的首数字
			sed 's/^[0-9][0-9]*//g' testesed.txt
	
### 43，cut命令
	https://blog.csdn.net/weixin_42255666/article/details/81302747#%C2%A0paste
	https://www.cnblogs.com/dong008259/archive/2011/12/09/2282679.html
	cut命令用来显示行中的指定部分，删除文件中指定字段。
	cut -d: -f1,7 /etc/passwd	按":"分割符进行拆分，输出第1列和第7列值
	cut -d\" -f4 /opt/testdata/result	特殊字符"使用反斜杠打印
	cut -d ' ' -f6 tmp/test		使用空格进行分割使用反斜杠打印
	masscan -iL conf/ip.txt -p80 --rate=10000 --output-format | cut -d ' ' -f6 > tmp/p80.txt
	公网IP的443端口扫描(0107 10:49:49开始, 0108 13:46 70%)

### 44，rsync文件同步命令
	-a：归档模式，表示以递归方式传输文件，并保持所有属性，它等同于-rlptgoD。-a选项后面可以跟一个--no-OPTION，表示关闭-rlptgoD中的某一个，比如-a--no-l等同于-rptgoD；
	-v：表示打印一下信息，比如文件列表、文件数量等；
	-r：表示以递归方式处理子目录，它主要是针对目录，如果单独传一个文件不需要加-r选项，但是传输目录时必须加；
	--delete：表示删除DEST中SRC没有的文件；
	--execlude=PATTERN：表示指定排除不需要传输的文件，等号后面跟文件名，可以是万用字符模式（如*.txt）；
	-P：表示在同步的过程中可以看到同步的过程状态，比如同步的文件传输速度等；

	rsync -av /etc/passwd /tmp/1.txt   #把/etc/passwd同步到/tmp目录下，并改名为1.txt
	rsync -av /etc/passwd 192.168.30.129:/tmp/1.txt 
	rsync -av --delete /opt/test /tmp/   #删除/tmp/目录下不包括在/opt/test中的文件，

### 45，split命令
	将大文件分割成小文件，如下，有15G大小的domain_20190109.txt文件，切割命令如下
	split  -b 500M  domain_tt_  domain_20190109.txt
	切分后会以domain_tt_aa、domain_tt_ab……domain_tt_az……文件存放，然后再压缩

### 46，fdisk 磁盘分区命令及格式化(Ubuntu)
	参考地址：https://blog.csdn.net/hejiamian/article/details/52031910
		https://www.cnblogs.com/tssc/p/9184895.html
	fdisk -lu	显示硬盘及所属分区情况
	fdisk /dev/sdb	对硬盘进行分区
	在Command (m for help)提示符后面输入n，执行 add a new partition 指令给硬盘增加一个新分区
	出现Command action时，输入e，指定分区为扩展分区（extended）。出现Partition number(1-4)时，
	输入1表示只分一个区。后续指定起启柱面（cylinder）号完成分区。
	在Command (m for help)提示符后面输入p，显示分区表。
	在Command (m for help)提示符后面输入w，保存分区表。
	 fdisk -lu	系统已经识别了硬盘 /dev/sdb 的分区。

	 硬盘格式化
	 mkfs -t ext4 /dev/sdb	显示硬盘及所属分区情况。说明：-t ext4 表示将分区格式化成ext4文件系统类型。
	 注意：在格式 化完成后系统有如下提示：This filesystem will be automatically checked every 28 mounts or180 days, whichever comes first. Use tune2fs -c or -i to override.
### 47，grep命令
	grep -E '3345|3346|3357' test.txt  # 按行搜索test.txt文件中包含3345、3346或3357的数据

### 48，文件或文件夹判断
	(1) 文件夹不存在则创建
		if [ ! -d "/data/" ];then
		mkdir /data
		else
		echo "文件夹已经存在"
		fi
	(2) 文件存在则删除
		if [ ! -f "/data/filename" ];then
		echo "文件不存在"
		else
		rm -f /data/filename
		fi
	(3) 文件比较符
		-e 判断对象是否存在
		-d 判断对象是否存在，并且为目录
		-f 判断对象是否存在，并且为常规文件
		-L 判断对象是否存在，并且为符号链接
		-h 判断对象是否存在，并且为软链接
		-s 判断对象是否存在，并且长度不为0
		-r 判断对象是否存在，并且可读
		-w 判断对象是否存在，并且可写
		-x 判断对象是否存在，并且可执行
		-O 判断对象是否存在，并且属于当前用户
		-G 判断对象是否存在，并且属于当前用户组
		-nt 判断file1是否比file2新  [ "/data/file1" -nt "/data/file2" ]
		-ot 判断file1是否比file2旧  [ "/data/file1" -ot "/data/file2" ]

### 49，ssh登录主机后，记录连接主机的地址会保存在.ssh目录下的known_hosts文件中，该文件自动生成。

### 50，sar与ethtool命名
	sar -n DEV 3  #每3秒查看一下当前网口实时传输速率
	ethtool	eth0	#显示网卡带宽
	netstat -ntop | grep 8086	#查看8086端口被占用情况
	
	sar -n DEV 3  #每3秒打印3条网卡统计信息，查看一下当前网口实时传输速率，通过此命令，可统计当前服务器发送和传输的平均流量大小，示例如下：
		04:12:56 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
		04:12:59 PM      eth0     87.54   2094.61      5.60    112.95      0.00      0.00      0.00
		04:12:59 PM        lo      1.01      1.01      0.06      0.06      0.00      0.00      0.00
		按住Ctrl+C退出时：
		Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
		Average:         eth0     88.12   2066.94      5.52    110.04      0.00      0.00      0.00
		Average:           lo      0.73      0.73      0.04      0.04      0.00      0.00      0.00
		
		可以看出当前一段时间内eth0网卡，入口带宽平均速率为5.52kB/s发送的数据包，出口带宽平均速率为110.04kB/s;		
		如果想查看当前网络实时带宽情况，可以使用上述命令！！！
		
		测试服务器带宽：
			1.下载脚本：
				英文【推荐】：wget https://raw.githubusercontent.com/FunctionClub/ZBench/master/ZBench.sh
				中文：wget https://raw.githubusercontent.com/FunctionClub/ZBench/master/ZBench-CN.sh
			2.注释掉脚本中测试国外节点的函数【否则会在测试国外节点的时候卡住】（第178行开始）
				#speed() {
				#	rm -rf /tmp/speed.txt && touch /tmp/speed.txt
				#	speed_test 'http://cachefly.cachefly.net/100mb.test' 'CacheFly'
				#	speed_test 'http://speedtest.tokyo.linode.com/100MB-tokyo.bin' 'Linode, Tokyo, JP'
				#	speed_test 'http://speedtest.singapore.linode.com/100MB-singapore.bin' 'Linode, Singapore, SG'
				#	speed_test 'http://speedtest.london.linode.com/100MB-london.bin' 'Linode, London, UK'
				#	speed_test 'http://speedtest.frankfurt.linode.com/100MB-frankfurt.bin' 'Linode, Frankfurt, DE'
				#	speed_test 'http://speedtest.fremont.linode.com/100MB-fremont.bin' 'Linode, Fremont, CA'
				#	speed_test 'http://speedtest.dal05.softlayer.com/downloads/test100.zip' 'Softlayer, Dallas, TX'
				#	speed_test 'http://speedtest.sea01.softlayer.com/downloads/test100.zip' 'Softlayer, Seattle, WA'
				#	speed_test 'http://speedtest.fra02.softlayer.com/downloads/test100.zip' 'Softlayer, Frankfurt, DE'
				#	speed_test 'http://speedtest.sng01.softlayer.com/downloads/test100.zip' 'Softlayer, Singapore, SG'
				#	speed_test 'http://speedtest.hkg02.softlayer.com/downloads/test100.zip' 'Softlayer, HongKong, CN'
				#}
				以及第344行
			3.执行脚本
				bash ZBench.sh
				
				输入主机拥有人：tank （随便输）
				输入主机的客户端IP： ip(主机的IP地址)
				先会安装一些程序
				最后显示整个带宽网络传输速率，即网络带宽
				

### 51，sshpass 用于非交互SSH的密码验证,一般用在sh脚本中，无须再次输入密码。
		它允许你用 -p 参数指定明文密码，然后直接登录远程服务器，它支持密码从命令行、文件、环境变量中读取。
		首次使用时，必须得手动登录一次远程服务器，获取远程服务器秘钥。然后可以直接通过命令来进行操作远程服务器。
		示例如下：
		   (1) 后跟密码
				sshpass -p 111111 ssh root@172.30.154.242 'ls -l'	# 以root账户及111111密码连接到172.30.154.242服务器根目录后，查看根目录文件列表。
		   (2) 后跟保存密码的文件名，密码是文件内容的第一行。
				sshpass -f 1.txt  ssh root@192.168.56.102			# 1.txt内容就是一行密码	
		   (3) 将环境变量SSHPASS作为密码
				 export SSHPASS=123456
				 sshpass -e  ssh root@192.168.56.102
		   (4) 如在多台主机执行命令:
				#!/bin/bash
				for i in $(cat /root/1.txt)				#1.txt存放主机列表
				do
					echo $i
					sshpass -p123456 ssh root@$i 'ls -l'
				done
			
		业务场景：
			远程服务器上进程监控，查看服务进程是否在线，来判断服务上线或下线。【只能判断进程在不在，不能判断进程是否正常工作】
### 52, arping ip 检查局域网mac地址是否冲突