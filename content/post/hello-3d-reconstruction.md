+++
title = "Hello 3D重建"
date = 2013-07-05T11:36:05+08:00
images = []
tags = ["3D", "技术"]
categories = []
draft = false
+++

第一次接触3D Reconstruction，用的SFM方法，找到个[牛人](http://homes.cs.washington.edu/~ccwu/)写的软件[VisualSFM](http://homes.cs.washington.edu/~ccwu/vsfm/)，这个软件强调了运算速度，包括SIFT算法的GPU计算和BA算法的多核优化，在我的笔记本(Intel Core i5，Nvidia GT640，64位Win7)上跑起来还是挺快的。
照片数据是我自己拍的工作室的一个椅子，椅子上放了个盒子，总共38张照片，用的iPhone4拍摄，分辨率为都是2592*1936

![](/media/3d-reconstruction/img00004.jpg)

![](/media/3d-reconstruction/img00005.jpg)

用VisualSFM + CMVS得到dence reconstruction结果，效果还算不错：

![](/media/3d-reconstruction/img00001.jpg)

![](/media/3d-reconstruction/img00006.jpg)

椅子的骨架都还可以(四个腿没整好)，中间光滑部分重建得不行，但是地板却效果很好，初步推测是SIFT方法特征提取与匹配上的问题，后续慢慢研究。
之后还找到了个提供3D重建服务的网站 ，简介里写着只能用于研究用途，需要申请账号，果断发邮件过去，很快就回复了，给了个账号，然后就把图片数据上传上去，之前不知道可以打成zip包一次性上传，我硬是一张张传上去的。。。图片上传完毕后点run job，一直到dence reconstruction完成，竟然用了56分钟！估计服务器上的处理没用到GPU，慢了非常多，不过也可能是dence reconstruction的算法不同导致的。

![](/media/3d-reconstruction/img00002.jpg)

![](/media/3d-reconstruction/img00003.jpg)

这个服务器上用的dence reconstruction方法与CMVS不一样，表面材质连续拼接在一起(CMVS的结果还是很小块的碎片组成的，没有拼接到一起)，但是这个结果明显和实际相差较大。
