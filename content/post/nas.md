---
title: "NAS折腾小记"
subtitle: ""
date: 2022-12-19T16:35:51+08:00
draft: false
author: "bugxch"
authorLink: "https://www.zhihu.com/people/bug_ahoy"
description: ""

tags: ["NAS"]
categories: ["投资"]

image: "https://pic.imgdb.cn/item/63b250255d94efb26faa6c53.jpg"
---

前段时间买了个nas，折腾了两个礼拜多，总算摸到点门道。


本月初心里痒痒拿下威联通TS-216，某东价格1199，为什么会买这个呢？我的脑回路也比较清奇，我在之前给我的R2S计划启用openclash，但是发现那个固件的插件有问题，于是在网上搜索了一番，找到了[R2S/R4S 设备 | 易有云 文档中心](https://doc.linkease.com/zh/guide/istoreos/install_r2s.html)的固件，发现安装之后的界面还挺好看，openclash也顺利用起来了（虽然后面是从[Tags · vernesong/OpenClash · GitHub](https://github.com/vernesong/OpenClash/tags)页面自己下载手动安装的）。

![image.png](https://pic.imgdb.cn/item/63a02655b1fccdcd36738d47.png)


细看上图里面有个易有云的存储服务，顺手google 下什么是易有云，发现就是个NAS。我在某宝上搜了，发现盒子比较简陋，可以安装一块硬盘，非常便宜，就下图的样子，看起来玩得花样比较多。联想之前华为发布家用NAS，加上我的时光相册不能上传比较大的图片和视频，让我觉得云盘备份瞬间不香了，要不整个家用NAS玩玩？

![image.png](https://pic.imgdb.cn/item/63a0265db1fccdcd36739a0a.png)


于是，我又在[最新NAS存储新品推荐\_什么值得买](https://post.smzdm.com/fenlei/nascunchufuwuqi/)下面泡了一段时间，发现NAS用的最多的就是群晖和威联通，鉴于威联通颜值能打，而且看到论坛里面的讨论这么全面，将来入手遇到问题也能方便找到解决方案，最终放弃了某宝非常便宜的方案，买了入门款的威联通TS216，先把这个玩熟，等过几年用的差不多了，升级的时候再换更高级的吧。

## 基本配置

这个东西买回来很快，但是我发现要真的玩起来还是有比较高的学习成本的，下面是我边学边练的折腾记录。

### 安装系统
我刚开始安装的是西数红盘4T plus，后来发现同步照片的时候NAS炒豆子的声音太大了，所以后来又买了两条m.2的闪迪500G固态硬盘，2个硬盘600大洋，所以整个一套下来应该花了3400大洋（还是蛮肉疼的）。

![image.png](https://pic.imgdb.cn/item/63a02668b1fccdcd3673a92d.png)


安装系统可以参考[NAS无法拒绝SSD的N个理由！这些厉害玩法，你知道吗？](https://post.smzdm.com/p/alldezle/)，选择RAID 1阵列。为了搞清楚安装里面各种名词，我也看了很多文章，终于搞清楚了存储池、静态卷、厚卷、精简卷和RAID、Qtier这些名词的含义，当时本来计划看说明书的，奈何500页太长了，而且也发现了威联通居然还有个[QNAPedia](https://wiki.qnap.com/wiki/Main_Page)网站，全英文你能信？下图是我当前存储池的状态

![image.png](https://pic.imgdb.cn/item/63a02672b1fccdcd3673b8b2.png)

### 远程访问
家用NAS需要能在外部访问才能发挥它最大的威力，这一节从下面的几个步骤一步步做介绍，我主要参考了[威联通 TS-451D外网高速访问：公网IP、DSM主机、端口映射、DDNS动态域名设置\_NAS存储\_什么值得买](https://post.smzdm.com/p/a0dq84nr/)，基本上照着配置即可。需要注意两点：
1. 外网IP申请下来之后，使用中国电信的网络app设置DMZ主机时，需要输入的是**威联通连接的那台路由器在光猫中的IP地址**。这里要特别提醒使用软路由的朋友，我的网络拓扑是入户光猫 -> r2s软路由 -> 无线硬路由（有线桥接模式）-> 威联通，所以我要输入的就是r2s的软路由在入户光猫中分配的ip地址，这个可以在中国电信网络关键app首页最下面的在线设备中看到。
    ![image.png](https://pic.imgdb.cn/item/63a02640b1fccdcd36736c3f.png)

2. 我的r2s刷的是openwrt系统，这个系统的内部端口转发可以参考[OpenWRT 设置 端口转发 小白教程 - 彧繎博客](https://opssh.cn/luyou/167.html)，我的端口转发设置长这样

	 ![image.png](https://pic.imgdb.cn/item/63a02682b1fccdcd3673d099.png)

 3. 如果没有外网IP，可以使用威联通自带的myCloud和DDNS服务，但是速度比较慢，设置系统查看状态应该可以，但是下载上传文件就勉为其难了。更好的方案是使用有偿的服务，比如上面易有云就提供内网穿透服务，最高到8M速度，我后来没有用。

#### 域名映射
如果外网IP申请下来，一般是在浏览器中直接输入外网IP地址和端口号，但是外网IP会因为光猫重新上电而发生变化，这个就不太方便，所以可以绑定域名，自动DDNS解析解决这个问题。我参考[威联通使用腾讯DNSPod域名DDNS解析完全小白指南+HTTPS证书部署（阿里、华为等国内主流云服务商通用）\_NAS存储\_什么值得买](https://post.smzdm.com/p/a5ozer6k/)设置好的，但是没有做最后的证书配置。

### 安装APP
以我一周左右的使用经验来看，最有的APP有下面几个，
- Qsirch，智能搜索各种文档，甚至支持图片的OCR识别后的内容搜索，非常好用，高级搜索功能非常好玩，甚至可以根据相机的镜头筛选照片；
- QuMagic，智能相册，比较有趣的是人物识别、物体分类、地点分类和智能相册；
- Qfiling，非常好用文档整理软件，填写好条件和要求，就可以自动将源目录中的文档按照要求搬移到对应的目的目录里面，比如我喜欢使用相机拍照，就可以使用这个功能，将所有的富士相机的照片按照年/月的文件夹的结构给我归档到多媒体的文件夹里面

上面最有用的3个软件具体怎么配置和使用可以参考
- [Qfiling，我注意你很久了！——威联通文件整理神器使用体验\_NAS存储\_什么值得买](https://post.smzdm.com/p/a9g4lz35/)
- [Qumagie 与 PhotoPrism丨两个NAS相册神级工具，AI人脸、元数据、搜索功能都有\_NAS存储\_什么值得买](https://post.smzdm.com/p/ad9dlk8d/)
- [QNAP威联通发布Qsirch 5.0搜索神器，图片搜索和归档更便捷好用\_软件应用\_什么值得买](https://post.smzdm.com/p/awx0wop4/)

## 应用设置
### 同步Obsidian笔记
从去年开始，我在[初识卡片笔记写作法 - 不写，就无法思考](http://blog.bugxch.top/takesmartnotes/#%E5%85%A8%E5%B9%B3%E5%8F%B0%E7%AC%94%E8%AE%B0%E8%BD%AF%E4%BB%B6)里面使用Obsidian，之前也使用过坚果云 + folerSync同步PC端，但是同步效果不理想。这次使用NAS，我发现APP Center里面有[ Resilio Sync](https://www.resilio.com/individuals/)，顺便就google了下，发现很适合NAS啊，可以参考[Resilio Sync | 全平台多设备文件同步/传输终极产品 - 知乎](https://zhuanlan.zhihu.com/p/459403503)。这个软件特点是可以多达60M+的速率在所有安装该软件的跨平台设备上同步，但是需要保证多个同步文件夹中至少有一个是在线的。所以，我的做法是在NAS上安装该软件作为共享的源文件夹，然后在PC和手机端分别安装并同步这个源文件夹，都赋予读写权限，这样无论哪一端的修改都会同步到其他两端。我在手机上安装后使用，可以实现瞬时同步，体验极佳。

### 同步博客
使用类似的方法，我将hugo博客的文件夹也放在了NAS上，PC端与其同步，这样就能实现二者的同步了。

### 搭建书屋
参考[NAS指南 篇三：打造最强家用NAS之私人书库搭建教程\_NAS存储\_什么值得买](https://post.smzdm.com/p/a992p6e0/)就能搭建起个人书库，唯一的缺憾是不能批量上传，我把中亚上面的几乎所有电子书和文档全部上传到NAS电子书服务器上了。
## 参考资料
- [威联通 TS-451D外网高速访问：公网IP、DSM主机、端口映射、DDNS动态域名设置\_NAS存储\_什么值得买](https://post.smzdm.com/p/a0dq84nr/)
- [新手轻松玩转的NAS丨威联通TS-216 +东芝N300开箱体验\_QNAP TS-221\_游戏硬件存储-中关村在线](https://diy.zol.com.cn/798/7985029.html)
- [NAS无法拒绝SSD的N个理由！这些厉害玩法，你知道吗？\_NAS存储\_什么值得买](https://post.smzdm.com/p/alldezle/)
- [Resilio Sync，兼具文件同步/共享/备份/加密的神器 - 知乎](https://zhuanlan.zhihu.com/p/348134043)

---

全文完🚀
