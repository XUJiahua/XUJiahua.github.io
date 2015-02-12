---
layout: post
title: "网页版幻灯片"
modified: 2015-02-11 22:29:47 -0800
category: []
tags: [IT,前端,JavaScript,Python]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

这几天趁工作闲暇在github上建了一个[作品展示网站](http://erinzhang.github.io/)。缘起于女朋友要更新自己的简历。我提议是不是作品放置在个人网站上会不会更有特色些。当然这个时候这个该如何实现还不知道呢。

想法很简单，以web的形式，提供类似桌面上浏览幻灯片的功能。想了想好像需要写不少代码的样子，JS水平又一般，太折腾了。开始求教Google老师。

找到一个JS库[Highside.js](http://highslide.com/)，demo用下来挺不错的。性能也可以，访问网站的时候只会加载缩略图，当点击缩略图才会下载大图，并会下载下一张图，提高用户体验。照着demo修修改改，差不多了。

现在唯一的问题就是我只有作品原图。用Python写了个生成缩略图的程序。为了美观，对图像等比缩放；原图大小参差，限制缩略图面积要差不多，约120*120。

    
    root = 'folder path'
    rawname = 'raw pic name'
    
    area = 120 * 120
    fullname = os.path.join(root, rawname)
    filename, ext = rawname.split('.')
    thumbname = os.path.join(root, filename + "_thumb." + ext)
    img = Image.open(fullname)
    zoom = math.sqrt(img.size[0] * img.size[1] / area)
    img = img.resize((int(img.size[0] / zoom),
    int(img.size[1] / zoom)), Image.ANTIALIAS)
    img.save(thumbname)

[代码点我](https://github.com/ErinZhang/ErinZhang.github.io/blob/master/ImgCutter.py)
