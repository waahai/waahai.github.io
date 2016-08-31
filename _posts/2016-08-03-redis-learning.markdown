---
layout: post
title:  redis learning notes
tags: redis
categories: redis
---
# General
* consist of server and client
* default port to 6379

# Server
* `redis-server` is under `src` folder
* could check the server via `telnet` to listening port

# Client
* built-in tool is `redis-cli`
* [redis client link](http://redis.io/clients)

# Usefull command
> * `KEYS *` to list all KEYS
> * `FLUSHDB` - Removes data from your connection's CURRENT database.
> * `FLUSHALL` - Removes data from ALL databases.
