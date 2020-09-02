+++
title = "3D重建实验"
date = 2014-02-18T19:55:58+08:00
images = []
tags = ["3D", "技术"]
categories = []
draft = false
+++

做实验有一定时间了，虽然目前设备不是太好，研究得也还不够深入，不过还是能看到些希望了。放几张照片，现在是采用1投影仪1相机的方式，时间编码结构光（格雷码），标定是用张正友平面棋盘标定方法

![](/media/3d-reconstruction-test/img00005.jpg)

![](/media/3d-reconstruction-test/img00006.jpg)

![](/media/3d-reconstruction-test/img00007.jpg)

![](/media/3d-reconstruction-test/img00008.jpg)

几张三维重建结果的图，只有一个面。

![](/media/3d-reconstruction-test/img00001.jpg)

![](/media/3d-reconstruction-test/img00002.jpg)

![](/media/3d-reconstruction-test/img00003.jpg)

![](/media/3d-reconstruction-test/img00004.jpg)

从重建结果可以看出，投影仪和相机的标定是比较准确的（结果图小熊背后的墙壁重建得很平整），不过目前各个步骤的方法都是用的相对简单、成熟的，比如结构光用的格雷码面结构光，要做到手持实时重建肯定要换结构光，标定方法也是要换的，准备采用多相机辅助的办法，再然后即使各个步骤的技术问题都解决了，要做成真正能用的“手持实时彩色三维重建系统”还可能存在各种各样工程上的问题的。
