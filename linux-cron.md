title: linux_cron
date: 2015-03-05 20:28:19
tags: [linux, cron]
---
## 背景
前段时间由于工作需要，所以比较系统的调查了一下cron，现在总结分享一下。
## crond简介
crond是linux用来定期执行程序的服务。当安装完成操作系统之后，默认便会启动此服务。crond每分钟会定期检查是否有要执行的工作，如果有要执行的工作便会自动执行该工作。而linux任务调度的工作主要分为以下两类：
* 系统执行的工作：系统周期性所要执行的工作，如备份系统数据、清理缓存。  
* 个人执行的工作：某个用户定期要做的工作，例如每隔10分钟检查邮件服务器是否有新信，这些工作可由每个用户自行设置。

特别说明一下：系统执行的工作是放在/etc/crontab中，而个人执行的工作是放在/var/spool/cron/user中。  
另外说明一下：crond是linux的服务，crontab是命令。

安装crontab：  
yum install crontabs  
服务操作说明：  
>/sbin/service crond start //启动服务  
>/sbin/service crond stop //关闭服务  
>/sbin/service crond restart //重启服务  
>/sbin/service crond reload //重新载入配置  
>/sbin/service crond status //查看服务状态  

## 使用权限

控制crond的使用权限，主要是两个文件：  
* /etc/cron.allow  
* /etc/cron.deny  

cron.allow的优先级比cron.deny的优先级高。也就是说当两个文件都存在时，首先根据cron.allow的内容进行判断。  
当cron.allow存在时，要想使用crond服务，user必须在该文件中列出；当cron.allow不存在而cron.deny存在时，要想使用crond服务，user不能出现在该文件中；若两个文件都不存在，则只有root能使用crond服务。  
因此，一般的使用情况是，不要cron.allow文件，放一个空的cron.deny文件在/etc下，这样所有用户都可以使用crond服务，若需要禁止某个用户使用，则可以直接在cron.deny文件中添加。

## crontab命令详解

>crontab [-u user] file (use file install user's crontab)  
>crontab [-u user] -e (edit user's crontab)  
>crontab [-u user] -l (list user's crontab)  
>crontab [-u user] -r (delete user's crontab)  
>crontab [-u user] -i (prompt before deleting user's crontab）   
>crontab [-u user] -s (selinux context)  

说明一下：当普通用户使用crontab命令时，是不需要也不能加[-u user]的，这个是给root使用的，root可以通过[-u user]操作其他用户的crontab。root用户查看自己的crontab也是不用加[-u root]的，当然你非要加也是可以的。

## crontab具体使用(个人执行)

用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：

minute   hour   day   month   week   command

具体说明一下：

首先各个字段用空格隔开。

>minute： 表示分钟，可以是从0到59之间的任何整数。  
>hour：表示小时，可以是从0到23之间的任何整数。  
>day：表示日期，可以是从1到31之间的任何整数。  
>month：表示月份，可以是从1到12之间的任何整数，当然也可以是英文缩写Jan、Feb等。  
>week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日，必须也可以是单词缩写Sun、Mon等。  
>command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。  

在这里，很有必要盗用一张图片，很形象。

![cron.png](http://images.cnitblog.com/blog/34483/201301/08090352-4e0aa3fe4f404b3491df384758229be1.png "cron")

举一个例子吧，比如要在每个月的15号和18号的早上7点到晚上23点之间，每隔两小时列出/etc目录下的文件：  
>\* 7-23/2 * * * ls /etc  

>分析一下：  

>\* 表示忽略，也就是都所有值都匹配  
>7-23/2 表示从23点到7点，每隔两小时  
>15,18 表示每个月的15号和18  

### 添加crontab文件
  一般情况下使用crontab -e这个命令，会通过编辑器打开crontab文件，然后按照上面提到的格式输入，一行代表一个周期性指令(ps:编辑器由环境变量$EDITOR指定)。
  
  另外一个比较常用的方法是将想要写入crontab中的内容先写入另外一个文件中(比如cronfile)，再使用crontab cronfile，就能将cronfile中的内容替换到之前提到的/var/spool/cron/user中。
  
  上面两个是比较常用的方法，再说说不太常用的方法：
  既然crontab的文件是/var/spool/cron/user，那么我们可以直接echo "* 23-7/2 * * * ls /etc" > /var/spool/cron/user，当然，必须使用root权限才可以。
  
  另外，还有一种方法就是echo "* 7-23/2 * * * ls /etc" | crontab，这种方法的原理其实与crontab cronfile相同。

### 列出crontab文件
  直接使用crontab -l，就能输出当前用户的crontab中的内容。顺带提一下，有时候，可能需要部分修改crontab中的内容，这时可以使用crontab -l > cronfile，在cronfile中做出对应的修改，再使用crontab cronfile添加crontab文件。

### 删除crontab文件
  直接使用crontab -r，就能删除当前用户的crontab。

## 其它一些想说的

### 环境变量

  除了上面说的$EDITOR是用来指定打开crontab的编辑器以外，还想提到的是在crontab中执行脚本的问题。

有时候想要在crontab中固定时间执行某个自己编写的脚本，可能会发现执行不成功，但是自己手动执行又是可以的，这个时候多半就是环境变量的问题了，需要修改一下crontab中的内容，在想要执行的脚本前，加上source ~/.bashrc，比如想要执行的脚本名为script，那么在crontab中就可以写为* 7-23/2 * * * source ~/.bashrc && script

### 系统执行crontab
  前面也说的，crontab有系统执行和个人执行之分，文件存放的位置也不同，当然，写法有些差别，可以看看系统执行的crontab，也就是/etc/crontab。
```bash
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# run-parts
01 * * * * root run-parts /etc/cron.hourly
02 4 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly
```
  与个人执行的crontab不同的是，首先指定了一些环境变量，其次多了root和run-parts两项。先说root，因为个人执行的crontab肯定是自己使用，因此不用指定这一项，而系统执行的crontab，需要指定是通过哪个用户来执行后面的指令的；对于run-parts，这个参数是执行后面指定的文件夹中的所有脚本，个人执行的crontab中，也是可以使用的。

### 邮件相关
  每次crontab执行完毕，系统都会将任务输出信息通过电子邮件的形式发送给当前系统用户，这样日积月累，日志信息会非常大，可能会影响系统的正常运行，因此，将每条任务进行重定向处理非常重要。