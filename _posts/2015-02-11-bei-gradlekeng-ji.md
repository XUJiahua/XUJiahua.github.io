---
layout: post
title: "被Gradle坑记"
modified: 2015-02-11 07:36:01 -0800
category: []
tags: [kafka]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

临近春节，不大忙。最近打算在工作中构建一个集中式的日志系统，翻到点以前做了书签又未看的kafka资料，又在ubuntu上[quick start](http://kafka.apache.org/documentation.html#quickstart)了一把，打算看看源码更深入学习。

按照[github](https://github.com/apache/kafka/tree/0.8.2)上的指示搭建kafka开发环境（[apache网站](https://cwiki.apache.org/confluence/display/KAFKA/Developer+Setup#DeveloperSetup-IntellijSetup)的过时了，看了就被坑了），一波三折。


主要还是被Gradle给坑了，还被坑了好久。不得不吐槽下，自动构建工具真多，ANT, Maven, Gradle，还有Scala相关的sbt等等等，不断得造轮子有意思么。

在第一步gradle的时候出错，查看了下自己的网络连接也是通的：

    FAILURE: Build failed with an exception.
    
    * What went wrong:
    Could not resolve all dependencies for configuration ':classpath'.
    > Could not resolve nl.javadude.gradle.plugins:license-gradle-plugin:0.10.0.
      Required by:
      org.apache.kafka:kafka-0.8.2:0.8.2.0
       > Could not GET 'http://repo1.maven.org/maven2/nl/javadude/gradle/plugins/license-gradle-plugin/0.10.0/license-gradle-plugin-0.10.0.pom'.
      > Connection refused
       > Could not GET 'http://dl.bintray.com/content/netflixoss/external-gradle-plugins/nl/javadude/gradle/plugins/license-gradle-plugin/0.10.0/license-gradle-plugin-0.10.0.pom'.
      > Connection refused

Google了一把，Gradle有自己的proxy设置，添加到如下到gradle.properties：

    
    systemProp.http.proxyHost=<your proxy>
    systemProp.http.proxyPort=<proxy port>
    systemProp.https.proxyHost=<your proxy>
    systemProp.https.proxyPort=<proxy port>

继续gradle，又出状况：

    FAILURE: Build failed with an exception.
    
    * Where:
    Script '/home/jiahua/Downloads/kafka-0.8.2/gradle/license.gradle' line: 2
    
    * What went wrong:
    A problem occurred evaluating script.
    > Could not find method create() for arguments [downloadLicenses, class nl.javadude.gradle.plugins.license.DownloadLicenses] on task set.
    
    
    
又Google了一把，[一定要装最新版](http://mail-archives.apache.org/mod_mbox/kafka-users/201410.mbox/%3CCAJt5Qtqh6EBkEyzzuU-E4fPbD_U=1tDbS-+RVi1ckuiMPbW9bA@mail.gmail.com%3E)，ubuntu上apt-get下来的gradle版本太低。

官网下载GRADLE，拷贝到opt目录，在~/.bashrc内添加：

    GRADLE_HOME=/opt/gradle/bin
    export GRADLE_HOME
    PATH=$PATH:$GRADLE_HOME
    export PATH

然后更新下：source ~/.bashrc

因为太麻烦，差点又要放弃了。所幸没有其他事情打扰，坚持了下来。还是要感谢G老师及时得提供了答案。
