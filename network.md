title: 网络知识记录
date: 2015-05-04 17:12:14
tags: network
---
这段时间接触到了好多网络方面的知识，都是些基础知识，不知道就是不知道，因此不得不在平时多多补充网络方面的知识。

现在把平时新学到的知识记录在这里。

## 域名和别名
[百度百科](http://baike.baidu.com/link?url=r5FtzXC7Mtd2D46xtLMrrA3e0iVyjN7Q9oyAxQBJsHqHzMt_bVmrVRr9vZvivK3R#5 "")有个简单的解释。  
按照我的理解就是：  

假设有一台服务器的ip是192.168.1.1，它可以有www.web1.com，www.web2.com，www.web3.com三个域名；也可以有www.web1.com一个域名，同时有www.web2.com，www.web3.com两个作为www.web1.com这个域名的别名。  
这两种情况相同的地方是，访问这三个url，都是访问192.168.1.1这台服务器。不同点是如果服务器的ip变成了192.168.1.2，那么第一种情况，需要修改web1，web2，web3，三个域名都需要更改；而第二种情况就只需修改web1就行了。

第一种情况就相当于是三个（web1,web2,web3）指向同一地址的指针；第二种情况是一个（web1）指向地址的指针和两个（web2，web3）指向指针的指针。
## VLAN相关
此小节参考[VLAN原理详解](http://blog.csdn.net/phunxm/article/details/9498829 "")
### VLAN概念
VLAN（Virtual LAN）即虚拟局域网，也就是一个广播域，[广播域](http://baike.baidu.com/link?url=NBT9oo8h3CI4GRV3tjLoGK6_wCbagdz0o-vwH1rUNAOyEBZYBKx70oJZUEgfoU7lRBVdwmlAG6NoWjxPjTa7wK "")是指能过直接通信的范围。广播域的划分，可以根据子网掩码来计算。  
按照目前我的理解来看，如果子网掩码是255.255.255.0，那么192.168.1.0~192.168.1.255是一个广播域，192.168.2.0~192.168.2.255是另外一个广播域；  
如果子网掩码是255.255.255.224，那么ip每隔32（256-224）为一个广播域，即192.168.1.0~192.168.1.31，是一个广播域，192.168.1.32~192.168.1.63。  
一个广播域内，第一个地址（192.168.1.0）是网络地址，最后一个地址（192.168.1.31）是广播地址。

VLAN间的通信需要经过路由。
### 路由
[路由](http://baike.baidu.com/link?url=e8vwGyZKeqEpfqH-pSSUerDJFtOSUfWQh2SV6o72lmVWB5p6CuPQvDg_3qlKYj60T91sAuGrVRs-KlX9ZTek8aZ8c41n6625E1icOwl0EGK#1 "")就是将一个接口的数据包转发到另一个接口的过程，工作在OSI模型的第三层（网络层）。包括：
> 确定最佳路径  
> 通过网络传输信息

### VLAN链接
VLAN间的链接分为：
> 访问链接（Access Link）  
> 汇聚链接（Trunk Link）

#### Access Link（访问链接）
同一台交换机，分为静态VLAN和动态VLAN。  
静态VLAN是通过设置交换机的不同端口为不同的VLAN。 如下图（图片均是盗用参考链接处）：
![access_link_port](http://img.blog.csdn.net/20130726174909406 "access_link_port")  
动态VLAN是根据客户端的MAC或ip或用户名设置为不同的VLAN。如下图（图片均是盗用参考链接处）：

基于mac：
![access_link_mac](http://img.blog.csdn.net/20130726174941218 "access_link_mac")    

基于ip：
![access_link_ip](http://img.blog.csdn.net/20130726175015406 "access_link_ip")

至于基于用户，就是根据连上的用户的用户名判断属于哪个VLAN。
#### Trunk Link（汇聚链接）
交换机间设置相同的VLAN。  
交换机间用一个端口设定为汇聚链接，也就是用来传输VLAN的信息。其它端口设置具体的VLAN号。
![trunk_link](http://img.blog.csdn.net/20130726175217593 "trunk_link")

### VLAN间通信
参考文章中写得很详细，这边就按照我自己的理解来阐述一下：
![vlan_communication](http://img.blog.csdn.net/20130726175659000 "vlan_communication")  
比如要从A发送信息到C。  
1. A从目标ip中判断C不在同一网段，因此会通过[ARP](http://baike.baidu.com/link?url=v4lWRw5HOygXoig4vh67Yp71bkgv6w5nYxv4D8g_FXEDO5EHwM1QoV_V81RUJrD5jQ6GSTYK_7bkDXTwyOQw_rpGy-7tc21QzBpP0vrLz5S "ARP")获取路由器的MAC地址。再将包含源MAC、源IP及目标IP的数据发送到路由器。交换机1口收到后，转到6口（汇聚链路），附上红色的VLAN识别信息（比如VLAN10），发送给路由器。  
2. 路由器收到数据帧后，确认VLAN的识别信息，交由属于红色VLAN的子接口接收。  
3. 由内部的**路由表**及目标IP，判断向哪里中继。交由蓝色VLAN子接口转发，附上蓝色的VLAN识别信息，并将目标MAC改为C的MAC（现在的目标MAC是路由器的MAC），发送到交换机。   
4. 交换机收到数据帧后，通过VLAN识别信息判断是蓝色VLAN，通过目标MAC判断为3口，因此将VLAN识别信息去掉后，发送到3口，这样C就收到了A的数据。