---
layout: post
title: "item-based collaborative filtering"
modified: 2019-06-10 11:43:48 +0800
category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

本文的基础是下面这篇文章，本文主要着重讲几个容易忽视的问题。

Item-item collaborative filtering with binary or unary data
https://medium.com/radon-dev/item-item-collaborative-filtering-with-binary-or-unary-data-e8f0b465b2c3

这篇文章挺好，比较注重implicit data（binary data，是否听了某个artisit），而implicit data是日常工作中最容易收集的数据了。

### Training: item-item similarity matrix

使用用户评分（向量）表示商品。求解商品之间的相似度，就是求解向量之间的相似度。

使用机器学习训练-预测的两阶段划分，求解商品之间的相似度矩阵是一个训练过程。

![](/media/15601386053473.jpg)

### cosine similarity and normalization

看看没有归一化（normalization）有什么问题。

假设：

1. 用户A对商品的评分：(1, 1, 1, 1, 1)
1. 用户B对商品的评分：(5, 5, 5, 5, 5)

可知，两个用户的品味是不同的。

而根据余弦相似度公式计算得出结果：1。也就是口味完全一致。显然与我们的认知是不一致的。
![](/media/15601420452086.jpg)

让我们利用Mean normalization来重新调整用户A、B的评分。

![](/media/15601432984898.jpg)
https://en.wikipedia.org/wiki/Feature_scaling

* average(x) = 3
* max(x) = 5
* min(x) = 1

这样，评分向量变化为：

1. 用户A对商品的评分：(-0.5, -0.5, -0.5, -0.5, -0.5)
1. 用户B对商品的评分：(0.5, 0.5, 0.5, 0.5, 0.5)

归一化后的向量之间余弦值为-1。品味完全不一致，与预期相符。


这里有一个假设，用户A、B之间的评分宽容度是一致的。所谓宽容度，有人看完电影觉得不错，会给5分，总体评分都很高，而有人评价比较刻薄，不错的电影最多4分，总体评分偏低。

所以一般推荐系统中对评分做归一化，一般是基于用户维度做归一化，而不是总体用户。

例如，在Andrew Ng的Machine Learning课上，采用的是每个用户评分减去每个用户自己的评分均值。
![](/media/15601439981816.jpg)


参考文中的归一化方式：

![](/media/15601451777277.jpg)
![](/media/15601451823717.jpg)



### 相似度计算优化

### Prediction: recommendation

给用户商品推荐需要两份数据：

1. 用户的历史行为（商品评分）
2. 商品相似度矩阵

（没有历史行为，就会有冷启动问题。这是协同过滤的通病。）

具体，某个用户u对某个商品i的评分计算如下：

![](/media/15601458001143.jpg)

* W: 权重矩阵，也就是学习到的商品之间相似度矩阵。
* Wij：商品i,j之间的相似度。
* rui: 用户u对商品i的评分（或是，是否点击、参与）。

TopK推荐，就是对每个商品进行评分计算，然后筛选出分数最高的K个。


### 推荐计算优化

注意上节的公式，参与计算的，与商品i相似的商品j，来自整个商品集合。复杂度，O(n)。

再加上需要计算每个商品的评分才能计算TopK，所以复杂度，O(n ^ 2)。

其实，只需要挑选商品i的Top K个相似商品参与计算，没必要计算全部。复杂度，O(k)，在大型电商中，K应该是远小于N的。

基于此，计算TopK的方式还能再优化。因为只选取了商品i的k个相似商品，如果用户评分过p个商品，那么最多k * p个商品需要计算评分，还是远少于N的。所以复杂度，O(k * k * p)。计算量上可以简化不少。


### 矩阵角度思考

用户商品评分预测用矩阵计算就一行。

```
score = data_matrix.dot(user_rating_vector).div(data_matrix.sum(axis=1))
```

data_matrix是商品相似度矩阵。

![](/media/15601482597748.jpg)

user_rating_vector是商品评分表示的用户偏好向量。
![](/media/15601482880774.jpg)

两者的shape分别是：

![](/media/15601483882690.jpg)

(285, 285) * (285, 1) ./ (285, 1) = (285, 1) 

矩阵计算的结果是(285, 1)的向量，也就是285个商品的评分。

怎么理解矩阵计算呢？我们可以从单个商品评分计算来看这个过程。



![](/media/15601495555671.jpg)

A商品的行向量，是A商品与其他商品之间的相似度。
A商品的行向量（1, 0.5, 0.6）乘以，用户的商品偏好表示的列向量 (0, 1, 1)，然后除以A商品行向量的和。是满足这个计算公式的。 
![](/media/15601458001143.jpg)

一个个商品评分计算stack起来，就成了矩阵计算。

