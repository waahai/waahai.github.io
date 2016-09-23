---
layout: post
title:  手工设置电脑AP
tags: AP
categories: wifi
---
在windows上可以直接通过命令行的方式开启AP，供其他无线设备连接，方法如下:

第一步：`netsh wlan set hostednetwork mode=allow ssid=access key=password` 热点为`access`, 密码为`password`

第二步: 确认网络连接中多了一个无线连接

第三步: 共享能够访问Internet网络的连接，允许新创建的无线连接共享

第四部: 启动AP `netsh wlan start hostednetwork`
