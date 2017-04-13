---
layout: post
title: "利用Jenkins+Ansible做持续部署"
modified: 2017-04-13 17:27:59 +0800
category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

## Ansible

工作需要，接盘又做后端开发，本着不做重复劳动的理念，加上前人留下来的脚本不够优秀，需要重搞一套一键部署的脚本。

之前在Windows平台开发的时候，使用cmd、 powershell也写过一套部署脚本，还算凑合着用。

这次是在Linux平台上，在重整之前，我也调研了下业界经验。最后选择了[Ansible](https://www.ansible.com/how-ansible-works )。Ansible比起bash简单很多，而且不怕重复执行相同命令有副作用，Ansible内置操作都是幂等的。同样的功能，用bash都能写。但是站在巨人肩膀上，效率更高。

选择Ansible的时候也考虑过Puppet、Chef。

比较之下，Puppet等都需要在每台被部署机上安装agent程序。而Ansible，只要被部署机上有Python即可。

考虑到公司服务器都是内置Python的Ubuntu系统，就放心使用Ansible了。

我们也基于Ansible上编写了应用发布的滚动部署脚本、回退脚本。
也编写了一套一键配置服务器的脚本，目前支持Nginx环境、APP服务器环境、Logstash安装等。

主要用途，结合讯联场景，有这么几点：
* 服务器的配置（包括软件安装）可以做到脚本化、版本控制 —— 服务器的配置应该像源代码一样控制起来，而不是一个黑盒、或是随意性太多，比如redis的安装路径都不一样、测试环境与生产环境的nginx配置不一样。服务器部署文档并没有脚本那么精确，只能算是一份人工操作指南，程序员重复劳动太多
* 适合一套架构需要部署几个环境的场景 —— 比如支付宝国际项目需要一套独立后端系统的需求
* 服务器需要水平扩展，无需人工一台台处理 —— 构建一些应用服务器，将应用服务器加入到nginx load balancer里，都是可以脚本化的（包含在滚动部署里了）
* 应用部署、滚动部署 —— 云收银的部署，感觉有点奇怪，发布的时候需要重新编译一遍程序，两台生产服务器就需要编译两次。虽然部署有脚本，但还是有手工处理的场景。部署没问题的前提是程序员没有checkout错git分支。觉得这个没必要。测试通过，生产环境就直接用测试环境的包，除了配置。这个可以结合jenkins做到的，发布测试环境的同时构建一个生产的包。测试人员验收通过测试包，发布时就用对应的生产包
* 快速构建一套测试环境 —— 如何构建一套mongo集群？测试有这样的需求，刚入职的同事都有这样的需求
* 进程监控是否存在 —— 写好进程检查脚本，通过ansible脚本定时执行就能看到结果了


### 滚动部署，就是个循环

```
for i in webservers:
    for j in nginxservers:
        remove i from load balancer
    // wait for a while until webserver i finish processing requests
    sleep(1 * minute) 
    deploy(i)

    for j in nginxservers:
        add i from load balancer
```


## Jenkins

真正考虑要搭个Jenkins，是因为Android打包太慢了、测试催着开发要最新的测试包。

有了Jenkins，测试可以自己查看最新的包了、开发忙自己的。

题外话，因为历史原因，没在刚开始就在MacOS系统上搭建Jenkins。以至于最后存在了两套Jenkins，另外一套专门给iOS打包用的。

接盘后端系统，顺便规范下之前的开发、测试、发布流程。

发现使用Jenkins Pipeline做持续部署很方便。

## Jenkins权限控制

严格控制生产部署。
也不需要写Jenkins插件，做在pipeline里，因为其本身就是基于Groovy的DSL，可以在其中写点代码。（下载了几个Jenkins插件，对Pipeline项目不起作用，也就不研究插件了。总之，有点麻烦）
