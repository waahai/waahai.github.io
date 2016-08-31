---
layout: post
title:  jmeter distribution test
tags: jmeter distribution remote performance
categories: test
---
jmeter distribution test require controlling client and remote server be in the same subnet network.
```
export RMI_HOST_DEF=-Djava.rmi.server.hostname=192.168.148.72

export RMI_HOST_DEF=-Djava.rmi.server.hostname=192.168.148.75

nohup jmeter-server > /dev/null &

jmeter -n -R192.168.148.72,192.168.148.75 -t tc001.jmx -l tc001_log.jtl
```

make sure the time is synced, or it lead to report error!
```
ntpdate -vd server
date -R
```
