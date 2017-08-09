---
title:  "Installing docker-compose"
description: "Installing docker-compose"
date:   2017-08-09 17:44:00
categories: [concepts]
tags: [docker]
---

When you install ***docker*** on centos7, ***docker-compose*** won't be installed by default. Which you can check
using following command.

```sh
[root@marx-vagrant-setup docker-monitoring]# which  docker-compose
/usr/bin/which: no docker-compose in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
[root@marx-vagrant-setup docker-monitoring]#

```

Following commands can be used to install ***docker-compose***.

``` sh
$ curl -L https://github.com/docker/compose/releases/download/1.12.0/docker-compose-`uname -s`-`uname -m` > ./docker-compose
$ sudo mv ./docker-compose /usr/bin/docker-compose
$ sudo chmod +x /usr/bin/docker-compose
```


You can check if ***docker-compose*** is installed successfully or not by using following command.

```sh
[root@marx-vagrant-setup docker-monitoring]# docker-compose -v
docker-compose version 1.12.0, build b31ff33
[root@marx-vagrant-setup docker-monitoring]#

```
