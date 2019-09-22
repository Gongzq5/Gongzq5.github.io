---
title: Nova 源码学习笔记
date: 2019-06-26 18:47:24
tags:
category:
---



How to study nova?

U can follow me in this blog for the whole process.

<!-- more -->

[TOC]

# 前置要求

* 安装Openstack
* 下载Nova的源代码



# 源码分析

主要的代码都在`nova`目录下

```
nova
├── api                     # API 服务
├── availability_zones.py
├── baserpc.py
├── block_device.py
├── cache_utils.py
├── cmd
├── common
├── compute                 # 处理计算资源
├── conductor
├── conf
├── config.py
├── console
├── consoleauth
├── context.py
├── crypto.py
├── db
├── debugger.py
├── exception.py
├── exception_wrapper.py
├── filters.py
├── hacking
├── hooks.py
├── i18n.py
├── image
├── __init__.py
├── ipv6
├── keymgr
├── loadables.py
├── locale
├── manager.py
├── monkey_patch.py
├── network                     # nova网络的实现
├── notifications
├── objects
├── pci
├── policies
├── policy.py
├── privsep
├── profiler.py
├── quota.py
├── rpc.py
├── safe_utils.py
├── scheduler                   # nova的调度任务
├── service_auth.py
├── servicegroup
├── service.py                  # 所有服务的基类
├── test.py
├── tests
├── utils.py
├── version.py
├── virt
├── vnc
├── volume
├── weights.py
└── wsgi.py
```

