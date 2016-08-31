---
layout: post
title:  docker learning notes
tags: docker
categories: docker
---
# Concept
`container`
`image`
`docker engine`
`docker compose`

# notes
* consist of `server` and `client`
* available docker tool for window version below 10, with virtual box and docker official vm image
* avoid complex env setup steps for test, such as `jekyll`
* local file system mapping
* `docker compose` to organize multi-containers
* a `container` will stop on the command exit

# command
* `docker run` start a container from image, `-i` interact, `-d` deamon
* `docker ps` the running container, with `-l` option to view the last container status
* `docker exec` exec command on a running container
* `docker start/stop` start/stop an exist container by its name(random string default)
