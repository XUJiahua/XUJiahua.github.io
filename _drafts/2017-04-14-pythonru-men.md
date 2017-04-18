---
layout: post
title: "Python入门"
modified: 2017-04-14 16:19:17 +0800
category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---


开发环境 MacOS

### Virtualenv —— 独立的Python环境

[https://virtualenv.pypa.io/en/stable/](https://virtualenv.pypa.io/en/stable/)


MacOS系统最好使用Virtualenv隔离出一个新的Python环境。默认的Python环境，安装一个包可能需要sudo权限，有sudo还不一定管用。

```Shell
// installation
sudo easy_install pip 
sudo pip install --upgrade virtualenv

// create a new env
virtualenv --system-site-packages ~/tensorflow

// activate the env
source ~/tensorflow/bin/activate
```

### Jupyter notebook —— 交互式编程

[http://jupyter.org/](http://jupyter.org/)

一步到位，将编辑器和终端消灭了。

```Shell
// installation
pip install jupyter

// run
jupyter notebook

```


### NumPy —— 数值计算

### Pandas —— 数据分析？？？

### matplotlib —— 数据可视化

http://matplotlib.org/gallery.html

看gallery哪个图适合，直接在这些代码上修修改改即可。

