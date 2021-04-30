---
layout: post
title: logrotate介绍
categories: logrotate
description: 
keywords: logrotate
---

#### 1）logrotate的简单介绍

logrotate是一个linux系统日志的管理工具。可以对单个日志文件或者某个目录下的文件按时间/大小进行切割，压缩操作，指定日志保存数量；还可以在切割之后运行自定义命令。



#### 2）配置文件

 Linux系统默认安装logrotate工具，它默认的配置文件

1、文件 logrotate.conf，在 /etc/logrotate.conf  ， 这个文件是logroate的默认配置文件 。

```shell
# see "man logrotate" for details
# rotate log files weekly
weekly  

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}
```



2、logrotate.d 是一个目录 ，在/etc/logrotate.d/ ， 在这个目录下，我们可以将自己的需要滚动的日志配置放在这个下面。该目录里的所有文件都会被主动的读入/etc/logrotate.conf中执行（主要是logrotate.conf的配置参数

include /etc/logrotate.d）。如果 /etc/logrotate.d/ 里面的文件中没有设定一些细节，则会以/etc/logrotate.conf这个文件的设定来作为默认值。 

自定义的配置;

```shell
/apps/logs/bc_redis/redis-console/redis-console-deployment-8bd665f8b-zsgrd-20200729152820.log{
daily
rotate 7
missingok
dateext
compress
notifempty
sharedscripts
postrotate
    [ -e /root/testlogs ] && kill -USR1 `cat /root/testlogs`
endscript
}
```



3、 Logrotate是基于CRON来运行的，其脚本是/etc/cron.daily/logrotate，日志轮转是系统自动完成的。实际运行时，Logrotate会调用配置文件/etc/logrotate.conf。可以在/etc/logrotate.d目录里放置自定义好的配置文件，用来覆盖Logrotate的缺省值。 

```shell
/usr/sbin/logrotate /etc/logrotate.conf >/dev/null 2>&1
 
EXITVALUE=$?
 
if [ $EXITVALUE != 0 ]; then
 
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
 
fi
 
exit 0
```

 如果等不及cron自动执行日志轮转，可以手动强制切割日志 。

 **logrotate命令格式 ：**

```
logrotate [OPTION...] <configfile>
-d, --debug ：debug模式，测试配置文件是否有错误。
-f, --force ：强制转储文件。
-m, --mail=command ：压缩日志后，发送日志到指定邮箱。
-s, --state=statefile ：使用指定的状态文件。
-v, --verbose ：显示转储过程。
```

4、 logroate的状态文件目录是  /var/lib/lograotate.status ， 在这个目录下，会记录上次文件的运行状态 。



#### 3） 切割

 比如以系统日志/var/log/message做切割来简单说明下：
第一次执行完rotate(轮转)之后，原本的messages会变成messages.1，而且会制造一个空的messages给系统来储存日志。第二次执行之后，messages.1会变成messages.2，而messages会变成messages.1，又造成一个空的messages来储存日志。如果仅设定保留三个日志（即轮转3次）的话，那么执行第三次时，则 messages.3这个档案就会被删除，并由后面的较新的保存日志所取代！也就是会保存最新的几个日志。
日志究竟轮换几次，这个是根据配置文件中的rotate参数来判定的。 



**logrotate.conf文件中参数配置：**

rotate 7：  一次将存储7个归档日志。对于第8个归档，时间最久的归档将被删除。 

copytruncate： 用于还在打开中的日志文件，把当前日志备份并截断；是先拷贝再清空的方式，拷贝和清空之间有一个时间差，可能会丢失部分日志数据。 

missingok： 在日志轮循期间，任何错误将被忽略。

notifempty： 如果日志文件为空，轮循不会进行 。

compress： 在轮循任务完成后，已轮循的归档将使用gzip进行压缩。 

maxsize 5M： 当日志文件到达指定的大小时才转储 。

daily： 日志文件分割频度。可选值为 daily，monthly，weekly，yearly 。

dateext： 使用日期作为命名格式 。

dateformat -%Y%m%d-%s： 配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，必须配合dateext使用，只支持 %Y %m %d %s 这四个参数 。

create 0644 root root：  以指定的权限创建全新的日志文件，同时logrotate也会重命名原始日志文件。 



**其他的一些参数：**

 nocompress ： 不希望对日志文件进行压缩，设置这个参数即可 。

delaycompress ： 总是与compress选项一起用，delaycompress选项指示logrotate不要将最近的归档压缩，压缩将在下一次轮循周期进行。

nocopytruncate ： 备份日志文件不过不截断 。

nocreate ： 不建立新的日志文件 。 

errors address ： 专储时的错误信息发送到指定的Email 地址 。

ifempty ： 即使日志文件为空文件也做轮转，这个是logrotate的缺省选项。 

sharedscripts  ： 表示postrotate脚本在压缩了日志之后只执行一次 。

postrotate/endscript ： 最通常的作用是让应用重启，以便切换到新的日志文件, 在所有其它指令完成后，postrotate和endscript里面指定的命令将被执行。在这种情况下，rsyslogd 进程将立即再次读取其配置并继续运行。 




**附：配置**

[deployer@CS~]$ sudo cat /etc/logrotate.d/redis   
/apps/logs/bc_redis/\*/\*/*.log {  
    rotate 5  
    copytruncate  
    missingok  
    notifempty  
    compress  
    maxsize 5M  
    daily  
    dateext  
    dateformat -%Y%m%d-%s  
    create 0644 root root  
}  
