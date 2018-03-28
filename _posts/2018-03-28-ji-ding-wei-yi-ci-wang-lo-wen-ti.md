---
layout: post
title: "记定位一次网络问题"
modified: 2018-03-28 23:57:46 +0800
category: []
tags: [ssh,网络]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---


现象：通过阿里云隧道ssh到家里电脑马上被踢出，报错信息：closed by remote host。

初步定位：阿里云特别卡，没法ssh，解决办法就是重启机器，好用一段时间。感觉是建立隧道脚本的问题，反复查看，没有眉目。

为什么特别卡：连接数太多了。阿里云机器上装了监控，连接数线性增长！CPU、内存的占用也线性增长了。

![alt text](images/180328/0328_1.png)
![alt text](images/180328/0328_2.png)
![alt text](images/180328/0328_3.png)


定位TCP连接的外部IP：
1. 先用htop大概看到了staging用户的进程有点多。
1. 通过ps命令过滤出staging用户的进程： ps -ef | grep staging
1. 进一步查看TCP连接： netstat -t
![alt text](images/180328/0328_4.jpeg)

排查IP：
1. 组员和自己的IP排除在外后
1. 就剩下这个IP，151.251.82.218.br 前面的IP地址是保加利亚的，有点惊！
1. 连接这么多，而且只连接不干坏事呢？.br不知道什么意思。无法理解。。。死路了。

![alt text](images/180328/0328_5.png)

试着假设151.251.82.218.br是家里IP：

a. 家里电脑敲击命令：
```
➜  ~ netstat -t | grep 118.31.32.201
tcp        0      0 192.168.1.15:34562      118.31.32.201:ssh       ESTABLISHED
tcp        0      0 plant:58000             118.31.32.201:ssh       ESTABLISHED
tcp        0      0 192.168.1.15:34558      118.31.32.201:ssh       ESTABLISHED
```

b. 阿里云敲击命令：
```
➜  ~ netstat | grep 151.251.82.218
tcp        0      0 plant:ssh               151.251.82.218.br:34562 ESTABLISHED
tcp        0      0 plant:ssh               151.251.82.218.br:34558 ESTABLISHED
tcp        0    140 plant:ssh               151.251.82.218.br:57969 ESTABLISHED
```

虽然不相信，但就是对应上了。就是家里的IP。


再次查看建立隧道脚本：
1. 脚本没问题，难道是crontab问题。
1. 查看日志，定位到Host key verification failed.。

```
tail -f /var/log/syslog

Mar 28 22:51:01 plant CRON[29957]: (plant) CMD (/home/plant/createTunnel.sh)
Mar 28 22:51:01 plant postfix/pickup[29854]: 82A61B00EC2: uid=1002 from=<plant>
Mar 28 22:51:01 plant postfix/cleanup[29929]: 82A61B00EC2: message-id=<20180328145101.82A61B00EC2@plant>
Mar 28 22:51:01 plant postfix/qmgr[29855]: 82A61B00EC2: from=<plant@plant>, size=745, nrcpt=1 (queue active)
Mar 28 22:51:01 plant postfix/local[29931]: 82A61B00EC2: to=<plant@plant>, orig_to=<plant>, relay=local, delay=0.01, delays=0.01/0/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
Mar 28 22:51:01 plant postfix/qmgr[29855]: 82A61B00EC2: removed
Mar 28 22:52:01 plant CRON[29981]: (plant) CMD (/home/plant/createTunnel.sh)
Mar 28 22:52:01 plant postfix/pickup[29854]: C71AEB00EC2: uid=1002 from=<plant>
Mar 28 22:52:01 plant postfix/cleanup[29929]: C71AEB00EC2: message-id=<20180328145201.C71AEB00EC2@plant>
Mar 28 22:52:01 plant postfix/qmgr[29855]: C71AEB00EC2: from=<plant@plant>, size=745, nrcpt=1 (queue active)
Mar 28 22:52:01 plant postfix/local[29931]: C71AEB00EC2: to=<plant@plant>, orig_to=<plant>, relay=local, delay=0.01, delays=0.01/0/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
Mar 28 22:52:01 plant postfix/qmgr[29855]: C71AEB00EC2: removed
```

```
sudo tail -f /var/mail/plant

From plant@plant  Wed Mar 28 22:52:01 2018
Return-Path: <plant@plant>
X-Original-To: plant
Delivered-To: plant@plant
Received: by plant (Postfix, from userid 1002)
    id C71AEB00EC2; Wed, 28 Mar 2018 22:52:01 +0800 (CST)
From: root@plant (Cron Daemon)
To: plant@plant
Subject: Cron <plant@plant> /home/plant/createTunnel.sh
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/home/plant>
X-Cron-Env: <PATH=/usr/bin:/bin>
X-Cron-Env: <LOGNAME=plant>
Message-Id: <20180328145201.C71AEB00EC2@plant>
Date: Wed, 28 Mar 2018 22:52:01 +0800 (CST)

Host key verification failed.
Creating new tunnel connection
bind: Address already in use
channel_setup_fwd_listener_tcpip: cannot listen to port: 19922
Could not request local forwarding.
Tunnel created successfully
```

结论：原来是一直没确认yes才不断重连的。算报错了，导致一直在建隧道，连接越建越多。阿里云小破机器连接数一多就挂了。

![alt text](images/180328/0328_6.png)
