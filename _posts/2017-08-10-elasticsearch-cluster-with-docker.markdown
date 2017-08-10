---
title:  "Running an Elasticsearch cluster with Docker"
description: "Setting up a multi-node ES cluster on a single Docker host with bash"
date:   2017-08-10 12:30:00
categories: [Guides]
tags: [Elasticsearch,Docker]
---

If you want to deploy ***N*** node ***Elasticksearch*** cluster for testing purpose then it's fastest way to deploy ***Elasticearch*** using docker on single server.

### Running a single instance

In order to monitor my Elasticsearch cluster I've created an ES image that has the [HQ](https://github.com/royrusso/elasticsearch-HQ) and [KOPF](https://github.com/lmenezes/elasticsearch-kopf) plugins pre-installed along with a Docker [healthcheck](https://github.com/thedeploymentguy/docker-elastic/blob/master/docker-healthcheck) command that checks the cluster health status.

```
FROM elasticsearch:2.4.1

RUN /usr/share/elasticsearch/bin/plugin install --batch royrusso/elasticsearch-hq/v2.0.3
RUN /usr/share/elasticsearch/bin/plugin install --batch lmenezes/elasticsearch-kopf

COPY docker-healthcheck /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-healthcheck

HEALTHCHECK CMD ["docker-healthcheck"]
```

I’ve built my image and created a bridge network for the ES cluster:

```bash
docker build -t es-image .
docker network create es-net
```

Next I've started an Elasticseach node with the following command:

```bash
docker run -d -p 9200:9200 \
	--name es-t0 \
	--network es-net \
	-v "$PWD/storage":/usr/share/elasticsearch/data \
	--cap-add=IPC_LOCK --ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
	--memory="2g" --memory-swap="2g" --memory-swappiness=0 \
	-e ES_HEAP_SIZE="1g" \
	es-image \
	-Des.bootstrap.mlockall=true \
	-Des.network.host=_eth0_ \
	-Des.discovery.zen.ping.multicast.enabled=false
```

With `--memory="2g"` and `-e ES_HEAP_SIZE="1g"` I limit the container memory to 2GB and the ES heap size to 1GB.

***Prevent Elasticsearch from swapping***

In order to instruct the ES node not to swap its memory you need to enable memory and swap accounting on your system.

On Ubuntu you have to edit `/etc/default/grub` file and add this line:

```
GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
```

Then run `sudo update-grub` and reboot the server.

Now that your server supports swap limit capabilities you can use `--memory-swappiness=0` and set `--memory-swap` equal to `--memory`.
You also need to set `-Des.bootstrap.mlockall=true`.

### Running a two node cluster

For a second node to join the cluster I need to tell it how to find the first node.
By starting the second node on the ***es-net*** network I can use the other node's host name instead of its IP to point the second node to its master.

```bash
docker run -d -p 9201:9200 \
	--name es-t1 \
	--network es-net \
	-v "$PWD/storage":/usr/share/elasticsearch/data \
	--cap-add=IPC_LOCK --ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
	--memory="2g" --memory-swap="2g" --memory-swappiness=0 \
	-e ES_HEAP_SIZE="1g" \
	es-image \
	-Des.bootstrap.mlockall=true \
	-Des.network.host=_eth0_ \
	-Des.discovery.zen.ping.multicast.enabled=false \
	-Des.discovery.zen.ping.unicast.hosts="es-t0"
```

Since the first node is using the 9200 port I need to map different port for the second node to be accessible from outside.
Note that I'm not exposing the transport port 7300 on the host. This port is accessible only from the ***es-net*** network.

With `-Des.discovery.zen.ping.unicast.hosts="es-image0"` I point `es-image1` to `es-image2` address.

The problem with this approach is that the `es-t0` node doesn't know the address of `es-t1` so I need to recreate `es-t0` with `-Des.discovery.zen.ping.unicast.hosts="es-t1:9301"`.
Running multiple nodes in this manner seems like a daunting task.

### Provisioning and running a multi-node cluster

To speed things up, I've made a script that automates the cluster provisioning.
The script asks for the cluster size, storage location and memory limit.
With these informations it can compose the discovery hosts location and point each node to the rest of the cluster nodes. So inorder to do it fast follow the steps below.

```
$ git clone https://github.com/thedeploymentguy/docker-elastic.git`
$ cd docker-elastic
$ bash up.sh
$ Enter cluster size: 3
$ Enter storage path: /home/ES_STORAGE
$ Enter node memory (mb): 4096
```
You should change `-Des.node.disk_type=spinning` to `-Des.node.disk_type=ssd` in [up.sh](https://github.com/thedeploymentguy/docker-elastic/blob/master/up.sh) if your storage runs on SSD drives. Also you should adjust `Des.threadpool.bulk.queue_size` to your needs.

Output:

```
Removing intermediate container 607c9f3ceea2
Successfully built cf746ece399d
Starting node 0
6d9ea71078e78a775616988c86412573dd04c4d5fdd42a09b1ff7019198f9f32
Starting node 1
e72386068472947bd9aa3cb7808bc303fc08d2c14a717a488eccccf77baae638
Starting node 2
dce385396a86d11df78062bbbed3156c24e288d5c614825d5f53902065fbda74
waiting 15s for cluster to form
cluster health status is green

```

You can check elastic search is up and running as below.
```
[root@marx-vagrant-setup docker-elastic]# curl -XGET http://<HOST-IP>:9200/
{
  "name" : "es-image0",
  "cluster_name" : "cluster-t",
  "cluster_uuid" : "ktzpqMmyR2-Z-V5q6DkPSg",
  "version" : {
    "number" : "2.4.1",
    "build_hash" : "c67dc32e24162035d18d6fe1e952c4cbcbe79d16",
    "build_timestamp" : "2016-09-27T18:57:55Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.2"
  },
  "tagline" : "You Know, for Search"
}
[root@marx-vagrant-setup docker-elastic]#
```


You can now access HQ or KOPF to check your cluster status.

```
http://<HOST-IP>:9200/_plugin/hq/#cluster
http://<HOST-IP>:9200/_plugin/kopf/#!/cluster
http://<HOST-IP>:9200/_plugin/kopf/#!/nodes
```

![kopf]({{ "assets/kopf-es.png" | relative_url }})
![HQ]({{ "assets/hq-es.png" | relative_url }})
![HQ]({{ "assets/hq-es-1.png" | relative_url }})

If you want to teardown you cluster, then you can use following script.


```bash
#!/bin/bash

read -p "Enter cluster size: " cluster_size
image="es-image"

# stop and remove containers
for ((i=0; i<$cluster_size; i++)); do
    docker rm -f "$image$i"
done

# remove image
docker rmi -f "$image"
```

Run teardown:

```
[root@marx-vagrant-setup docker-elastic]# bash down.sh
Enter cluster size: 3
es-image0
es-image1
es-image2
Untagged: es-image:latest
Deleted: sha256:cf746ece399d88db3adfb7b49b094a68856adccbb195fb3a36f0e09c0f103642
Deleted: sha256:2bcdf8c1764fa6363ff106b6da98ee8446d9465079f6f3b6ae2c5c440c2a877e
Deleted: sha256:012d38bc6744035e79450115537a715d59db300eca331265faac8c594ce75034
Deleted: sha256:01fb04d700aeb61847b18f7f8f6a18c5f07d67ccd703d2e0ea995d874037588a
Deleted: sha256:e9fa1c90fd031e6244469a78351c1672ea578fed5dda75da5db06bf328bb07dd
Deleted: sha256:f6ce4ad966298b4f120ee559f318da1fb46ae4c46c05f595df30da7002bdc0c0
Deleted: sha256:23ce3d9755968723058a672c53500b05fbcf3259eba905ca2a498b8233690e46
Deleted: sha256:fd65ae7a55cc1455e71ca99054c8aaf4038e0ff8ec7d6b44cffde3afe3ee8448
Deleted: sha256:7003ac4a71f080ede3d5c5f4b617484d618821c51e66eae7dd49fb2ee501a0e0
[root@marx-vagrant-setup docker-elastic]#

```

This script ***down.sh*** is also available on github in same [repo](https://github.com/thedeploymentguy/docker-elastic/blob/master/down.sh)
