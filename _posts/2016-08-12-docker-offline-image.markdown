---
layout: post
title:  Docker image offline tips
tags: docker offline
categories: tools
---
use `docker save` command to save an image to file, and then `docker load` to docker offline

* pull the image on a computer that have access to the internet
> `docker pull xxx`
* Then save this image to a file
> `docker save -o filename xxx`
* Transfer the file on the offline computer and load it to docker
> `docker load < filename`
