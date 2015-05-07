title: su和su -
date: 2015-02-03 22:39:22
tags: [linux,su]
---
今天晚上突然发现，用普通用户通过putty登陆后，su到root，好多命令没有。。。而且切换到/root目录下source .bashrc和.bash_profile都没有用。
```c
[zhaojc@centos ~]$ su
口令：
[root@centos zhaojc]# useradd
bash: useradd: command not found
[root@centos zhaojc]# cd ~
[root@centos ~]# source .bashrc
[root@centos ~]# source .bash_profile
[root@centos ~]# useradd
bash: useradd: command not found
[root@centos ~]# echo $PATH
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/bin:/bin:/usr/bin:/home/zhaojc/bin:/root/bin
```
网上查了一下，才发现su是所谓的半切换，切换后，还是会保留原用户的环境(细心的话，会发现，su后，还在/home/zhaojc目录下)。至于为什么source /root下的两个文件也没用，我想可能是因为root相关环境配置接在了原用户的后面，比如PATH的话，/home/zhaojc/bin在/root/bin前面(个人猜想，没有依据)。  
真正的切换用户应该是su -，这样会重新配置用户的环境。
```c
[zhaojc@centos ~]$ su -
口令：
[root@centos ~]# groupadd
Usage: groupadd [options] group

Options:
  -f, --force                   force exit with success status if the specified
                                group already exists
  -r,                       create system account
  -g, --gid GID                 use GID for the new group
  -h, --help                    display this help message and exit
  -K, --key KEY=VALUE           overrides /etc/login.defs defaults
  -o, --non-unique              allow create group with duplicate
                                (non-unique) GID

[root@centos ~]# echo $PATH
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
```
可以看到，用su -切换后，PATH中没有了/home/zhaojc/bin，说明我之前的猜想，还是有点沾边的。`(*∩_∩*)′