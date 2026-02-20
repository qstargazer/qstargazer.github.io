---
title: R2S外接风扇设置
subtitle: ""
date: 2023-01-25T21:16:42+08:00
draft: false
author: bugxch
authorLink: https://www.zhihu.com/people/bug_ahoy
description: ""
tags:
  - r2s
categories:
  - 技艺
image: https://pic.imgdb.cn/item/63d13639face21e9efb8dbc4.jpg
---

<!--more-->

r2s 每更新一次固件版本，就需要重新刷一下风扇的脚本，非常麻烦，记录下，下面是文章全文。

主要包括如下几个步骤：

1. 通过ssh登录路由器的shell界面，进入目录 `/etc/init.d`目录，查看是否有fa-fancontrol或者pwm-fan等包含fan字段的脚本，如果有则选中删除。

   ![image-20230125214026638](https://pic.imgdb.cn/item/63d1314bface21e9efaf3208.png)
2. 在当前目录，输入以下命令

   ```shell
   cd /etc/init.d/
   touch pwm-fan
   chmod 777 pwm-fan
   /etc/init.d/pwm-fan enable
   ```
3. 打开pwm-fan文件，输入如下内容

   ```shell
   #!/bin/sh /etc/rc.common

   START=99
   start () {
   echo 0 > /sys/class/pwm/pwmchip0/export
   echo 0 > /sys/class/pwm/pwmchip0/pwm0/enable
   echo 50000 > /sys/class/pwm/pwmchip0/pwm0/period
   echo 1 > /sys/class/pwm/pwmchip0/pwm0/enable
   echo 49990 > /sys/class/pwm/pwmchip0/pwm0/duty_cycle; # 初始风扇不转
   while true
   do
   	temp=$(cat /sys/class/thermal/thermal_zone0/temp) #去掉了$旁的空格
   	if [ $temp -gt 50000 ] ; then # 温度高于 50 风扇开始转，可修改，比如65000为65度；
   		echo 30000 > /sys/class/pwm/pwmchip0/pwm0/duty_cycle;
   		elif [ $temp -le 45000 ] ; then # 温度低于 45 风扇停转，可修改，比如55000为55度；
   		echo 49990 > /sys/class/pwm/pwmchip0/pwm0/duty_cycle;
   	fi #多加了fi
   	sleep 1s; # 1s检测一次，正常使用设置为60s
   done
   }
   ```
4. 使用下面的shell命令改成shell的format

   ```shell
   dos2unix pwm-fan
   ```
5. 然后我们登陆后台web管理界面，在【系统】--【启动项】--“启动脚本”下面能看到99号优先级名为pwm-fan的脚本，点击第二个【启动】按钮，风扇就开始转了，当温度降低到你设定的最低临界值的时候，风扇会自动停止。温度达到你设定的启动临界值的时候风扇会自动启动。最后重启一下你的路由器，确保设置生效。

   ![image-20230125215915284](https://pic.imgdb.cn/item/63d135b3face21e9efb776c1.png)

---

全文完🚀
