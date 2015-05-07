title: linux_ntp
date: 2015-03-05 20:08:16
tags: [linux, ntp]
---
## 背景
通过设置本地的ntpd服务，实现本地与服务器时间的同步。  
此文仅简单说明环境的搭建
## NTP简介
NTP(Network Time Protocol)是一种网络时间协议。是以封包交换把两台电脑的时钟同步化的网络协议。NTP使用UDP端口123作为传输层。它是用作抵销可变延迟的影响。

简单说明一下：NTP是一种协议；ntpd是系统中的服务，在linux下可以通过/sbin/service ntpd status或/etc/init.d/ntpd status来查看此服务的状态；ntpdate，ntpq，ntptrace等是一系列与ntp相关的命令。

安装NTP： yum install ntp  
一般系统上都会有ntp的包，可以提前通过rpm -qa | grep ntp查看机器上是否已经安装ntp。  
服务操作说明：  
>/sbin/service ntpd start //启动服务  
>/sbin/service ntpd stop //关闭服务  
>/sbin/service ntpd restart //重启服务  
>/sbin/service ntpd status //查看服务状态  

## ntpd服务设置
ntpd服务的相关设置文件有以下5个：  
1)/etc/ntp.conf  
这个是NTP daemon的主要设文件，也是 NTP 唯一的设定文件。  
2)/usr /share/zoneinfo/  
在这个目录下的文件其实是规定了各主要时区的时间设定文件，例如北京地区的时区设定文件在/usr/share/zoneinfo/Asia/Beijing 就是了。这个目录里面的文件与底下要谈的两个文件(clock 与localtime)是有关系的。  
3)/etc/sysconfig/clock  
这个文件其实也不包含在NTP 的 daemon 当中，因为这个是 linux 的主要时区设定文件。每次开机后，Linux 会自动的读取这个文件来设定自己系统所默认要显示的时间。  
4)/etc /localtime  
这个文件就是“本地端的时间配置文件”。刚刚那个clock 文件里面规定了使用的时间设置文件(ZONE) 为 /usr/share/zoneinfo/Asia/Beijing ，所以说，这就是本地端的时间了，此时， Linux系统就会将Beijing那个文件另存为一份 /etc/localtime文件，所以未来我们的时间显示就会以Beijing那个时间设定文件为准。  
5)/etc/timezone  
系统时区文件

其实主要用到的就只有/etc/ntp.conf，其它文件知道就行了。下面简单说一下/etc/ntp.conf文件的简单写法。  
1)上层主机设定  
server [IP | hostname] [prefer]  
上层主机就是通过server来设定的，设定好上层主机，并保持ntpd服务处于启动状态，这样就能让本机与上层主机保持时间同步了。prefer参数是指当有多个上层主机时，主要通过指定了prefer的上层主机进行时间同步。  
另外可以通过driftfile来指定用于记录与上层服务器联系时所花费的时间，默认为driftfile /var/lib/ntp/drift。  
2)权限设定  
权限设定主要是通过restrict来设定的，语法为：  
restrict IP mask netmask_IP parameter  
其中IP可以使具体值，也可以是ip段。netmask_IP是子网掩码。parameter有以下选项：  

>ignore：关闭所有的 NTP 联机服务  
>nomodify：表示 Client 端不能更改 Server 端的时间参数，不过，Client 端仍然可以透过 Server 端来进行网络校时。   
>notrust ：该 Client 除非通过认证，否则该 Client 来源将被视为不信任网域   
>noquery ：不提供 Client 端的时间查询  
>notrap ：不提供trap这个远程事件登入  

ps:如果 paramter 完全没有设定，那就表示该 IP (或网域)“没有任何限制”  
例如：  
restrict 127.0.0.1　　　 #允许本机任意操作。  
restrict 192.168.0.1 mask 255.255.255.0 nomodify  #局域网段内可以通过这台NTP Server进行时间同步，但是不能修改服务器时间。 

另外还有一个fudge，具体如下：
fudge IP stratum number，这个设定一般是将自身作为服务器，因此IP一般为127段stratum后的number为服务器层数，最大为16。  
默认为fudge 127.127.1.0 stratum 10

其它设置值，建议保持系统默认值。

## ntp时间同步

从ntpd服务设置中可以知道，设置好ntp.conf文件后，保持ntpd服务处于启动状态，机器就会与上层服务器进行时间同步了。现在主要介绍一些时间同步的命令。特别说明一下，想要通过命令进行时间同步时，需关闭ntpd服务。

1)ntpdate IP  
直接通过ntpdate加上想要进行时间同步的服务器IP，就能获得服务器上的时间，并设置到本机系统中。  
2)rdate -s IP  
与ntpdate类似，也是获得服务器上的时间，并设置到本机系统中。如果不加-s则仅是打印出服务器时间。 
 
说明一下，这两条命令以及上面通过ntpd的时间同步，仅仅是将时间设置到了系统中，如果想要设置到硬件时钟中，还需要hwclock。  
因此，如果想要设置本机的时间同步，可以将命令写入到crontab中，这样系统就会定期执行这两条命令实现时间同步了。不过通过查看网上资料，说是通过ntpdate来调节时间，时间会产生跃变，而通过ntpd服务来进行时间同步，是通过调节时钟频率渐变调节时间的。因此不推荐使用ntpdate + crontab来做时间同步。不过，在我测试时发现，ntpd服务进行时间同步时，也是通过跃变调节时间的。也有可能是我的设置有些问题(后来调查得知，需要在文件/etc/sysconfig/ntpd中加入-x 参数,变成: OPTIONS=”-U ntp -x -p /var/run/ntpd.pid”，这样ntpd就能实现渐变调整时间)。
## 其它
关于ntp的环境搭建是一件很复杂的事情，我也没有完全搞懂。这篇文章只是介绍了一些最简单的设置方法，通过这些设置，能够实现本地机器与服务器的时间同步。