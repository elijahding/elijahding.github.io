---
layout: post
title:  "Points and Vectors"
date:   2017-09-01
author:     "Kyle Ding"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags: 
    - Math
---



#  Points and Vectors - 点和向量

[几何学](https://zh.wikipedia.org/wiki/%E5%87%A0%E4%BD%95%E5%AD%A6)的两大基本概念是[点](https://zh.wikipedia.org/wiki/%E7%82%B9)和[向量](https://zh.wikipedia.org/wiki/%E5%90%91%E9%87%8F)，本质上点是`空间`中的一个位置。

我们用“·”表示`点`，我们可以在[坐标系](https://zh.wikipedia.org/wiki/%E5%9D%90%E6%A8%99%E7%B3%BB)中，用坐标表示一个`点`：
![](/img/pav.png)
例如 对于[二维空间](https://zh.wikipedia.org/wiki/%E4%BA%8C%E7%BB%B4%E7%A9%BA%E9%97%B4)，标准坐标系是[笛卡儿坐标系](https://zh.wikipedia.org/wiki/%E7%AC%9B%E5%8D%A1%E5%84%BF%E5%9D%90%E6%A0%87%E7%B3%BB)。我们在该坐标系中用有序的坐标对表示点。
首先是 `X 轴` 然后是 `Y 轴`，这个点在[原点](https://zh.wikipedia.org/wiki/%E5%8E%9F%E9%BB%9E)右侧两个单位，和下方一个单位处 表示为 (2, -1)。
![](/img/pav-1.png)
在[三维空间](https://zh.wikipedia.org/wiki/%E4%B8%89%E7%B6%AD%E7%A9%BA%E9%96%93)里，我们可以用类似的方式表示`点`，当然是三个坐标轴了。
首先是 `x 轴` 然后 `y 轴` 最后是 `z 轴`，和点相比，[向量](https://zh.wikipedia.org/wiki/%E5%90%91%E9%87%8F)是表示位置更改的对象。在[欧几里得空间](https://zh.wikipedia.org/wiki/%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%97%E7%A9%BA%E9%97%B4)里，向量可以看做连接两个点的箭头，向量的重要属性是**大小**，即箭头的`长度`以及`方向`。
![](/img/pav-2.png)
给出`坐标系`后，我们可以用在每个坐标轴方向的更改大小，以数字方式表示向量。
例如上图向量向右移了 2 个单位，所以 x 轴为 2，向上移了 4 个单位 所以 y 轴是 4。
> **注意：**
> 向量的坐标轴表示方法和点的表示方法不一样，通常会以`列`的方式表示向量，并用`方括号`括起来。而点用`坐标轴列表`表示，并用`小括号`括起来。点和向量的主要区别之一是**向量没有固定的位置**。
> 如果在不同的位置画出相同的箭头，则依然等于第一个箭头，这是很重要的区别。
> 如果两个向量表示同一方向相同的变化量，那么二者就相等，但是两个点要相等 ，必须位于相同的位置。
虽然存在这一区别，但是在学习线性代数时，通常按照以下方式表示向量中的点对于给定坐标系。
![](/img/pav-3.png)
点 P 与从原点开始到点 P 结束的向量有关联。虽然这是一种记忆便捷方法，但是有时候了解这两个概念之间的区别会有帮助。


