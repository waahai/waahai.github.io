---
layout: post
title:  Jmeter distribution deploy
tags: jmeter distribution performance remote
categories: test
---
压力测试中，单个机器往往不能够构造足够的压力，这就需要用到Jmeter分布式测试。
JMeter分布式测试有以下需要注意的地方
* 每个机器都使用相同的配置，而不是均分，比如一个10线程50TPS的配置，3台分布式就相当于30线程150TPS
* 需要每台机器都处在相同的网络子网网段
* 每台机器的时间必须同步，不然可能导致report不准确的问题，但是不影响测试的执行
* 每台机器的Jmeter版本需保持一致
* 当机器具有多个IP地址时，需要在`system.properties`中指定rmi连接的IP地址

为了达到上述的要求，我写了个基于windows的一键部署方案：
* 检测slave机的环境，包括和控制机以及测试对象服务器的网络连接
* 获取本机的IP地址和系统时间，然后通过curl上报给控制机
* 控制机上用docker搭了个nodejs+mysql来获取并存储slave机上报的信息，达到汇总机器信息以及生产需要在jmeter配置中的IP串的目的

详细实现可参考Github项目 [auto_deploy_jmeter](http://redis.io/clients)
