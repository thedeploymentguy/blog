---
title:  "Installing Ansible on CentOS 7"
description: "IInstalling Ansible on CentOS 7"
date:   2017-08-22 12:30:00
categories: [concepts]
tags: [Ansible]
---

***Ansible*** is an automation engine, similar to Chef or Puppet, that can be used to ensure deployment and configuration consistency across many servers, and keep servers and applications up-to-date. Though, unlike some other tools, Ansible does not require a client component/agent. Following are the steps to install Ansible on Centos7

### Add the EPEL Repository

Ansible is part of Extra Packages for Enterprise Linux (EPEL), which is a community repository of non-standard packages for the RHEL distribution. First, we’ll install the EPEL repository:

``` sh
[root@marx-vagrant-setup ~]# rpm -iUvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-10.noarch.rpm
Retrieving http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-10.noarch.rpm
warning: /var/tmp/rpm-tmp.EGKbln: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:epel-release-7-10                ################################# [100%]
[root@marx-vagrant-setup ~]#
```

### Update packages and then install ansible

```
[root@marx-vagrant-setup ~]# yum -y update
[root@marx-vagrant-setup ~]# yum -y install ansible
```

### Verify The Installation
If above commands get completed successfully then you can verify ansible installed as below

```
[root@marx-vagrant-setup ~]# ansible --version
ansible 2.3.1.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = Default w/o overrides
  python version = 2.7.5 (default, Nov  6 2016, 00:28:07) [GCC 4.8.5 20150623 (Red Hat 4.8.5-11)]
[root@marx-vagrant-setup ~]#
```
Fantastic  :satisfied:

You can also build an RPM yourself. From the root of a checkout or tarball, use the make rpm command to build an RPM you can distribute and install. Make sure you have `rpm-build`, `make`, `asciidoc`, `git`, `python-setuptools` and `python2-devel` installed.
```
$ git clone git://github.com/ansible/ansible.git --recursive
$ cd ./ansible
$ make rpm
$ sudo rpm -Uvh ./rpm-build/ansible-*.noarch.rpm
```
