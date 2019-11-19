---
title: Kolla-ansible安装Openstack(Ubuntu 18)全记录
date: 2019-11-16 19:48:30
tags:
category:
---



使用Kolla-ansible在Ubuntu18.04上安装Openstack

当前只完成了all-in-one的部分，multinode遇到了坑，晚些再找找解决办法

<!--more-->

[TOC]


# 准备工作

1. 两台机器 Ubuntu 18.04 Server版
    2 network interfaces
    8GB main memory
    40GB disk space
    
2. 安装pip
    ```
    apt-get update
    apt-get install python-pip
    pip install -U pip
    ```
# 配置Ubuntu

1. apt换源，这里换成了清华源（注意不同的Ubuntu这个url是不同的），而且如果安装的时候写明了源的话，进来应该就不用自己改了
    ```
    cp /etc/apt/sources.list /etc/apt/sources.list.bkp
    vim /etc/apt/sources.list
        
    # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

    # 预发布软件源，不建议启用
    # deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
    ```

2. pip源
    ```
    ~/.pip/pip.conf
    [global]
    index-url = https://pypi.tuna.tsinghua.edu.cn/simple
    ```

3. python-pip 问题
   
    可能会遇到说`cannot import main`这样的错误，网上都让你直接修改pip的源码，但是我改了好几个都GG了，发现重装Pip比较简单

    
    ```
    sudo python3 -m pip uninstall pip && sudo apt install python3-pip --reinstall
    sudo python -m pip uninstall pip && sudo apt install python-pip --reinstall
    ```
    按以上方式重装，注意python版本


4. 允许root用户登录
    ```
    sudo vi /etc/ssh/sshd_config

    修改
    PermitRootLogin yes
    #PermitEmptyPasswords yes 无密码登录

    重启SSH服务
    service sshd restart  # 或者 /etc/initd.d/sshd restart
    ```


3. 安装一些基础依赖
    ```
    apt-get install python-dev libffi-dev gcc libssl-dev python-selinux python-setuptools
    ```
    
4. 安装ansible
    ```
    apt-get install ansible
    ```
    
5. 升级Ansible
    ```
    pip install -U ansible
    ```
    
6. 【可选】添加到 /etc/ansible/ansible.cfg:
    ```
    [defaults]
    host_key_checking=False
    pipelining=True
    forks=100
    ```

7. 安装Docker和配置Docker hub的国内源

   安装：<https://docs.docker.com/install/linux/docker-ce/ubuntu/>

   配置国内源推荐阿里云，点进去控制台找`容器`，里边有配置加速器的选项，就可以了，我用了中科大和daocloud的都不成，估计晚些会自己配一个docker registry，不然项目大规模部署的话，拉取确实太慢了

# 网络配置

1. 虚拟机选择3个网卡

   * 一个是NAT，负责连外网；
   * 一个是主机模式，给主机上的虚拟网卡配置成了`10.0.0.X/24`
   * 一个随便设置了，用来给neutron的那个配置用

2. Ubuntu 显示网卡

   ```
   ifconfig -a
   ```

   不加-a的话，如果有未启用的网卡是不显示的，一般最好加上-a

3. 配置netplan

   ```
   vim /etc/netplan/50-cloud-init.yaml
   
   
   network:
       ethernets:
           ens33:
               dhcp4: true
           ens38:
               addresses: [10.0.0.3/24]
               routes:
                   - to: 10.0.0.0/24
                     via: 10.0.0.1
               dhcp4: no
       version: 2
   ```

   注意不要配置gateway4，而是使用routes进行路由，不然会导致路由的异常，可查看`route -n`查看路由表

   我现在的路由表是

   ```
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   0.0.0.0         192.168.178.2   0.0.0.0         UG    100    0        0 ens33
   10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 ens38
   10.0.0.0        10.0.0.1        255.255.255.0   UG    0      0        0 ens38
   172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
   192.168.178.0   0.0.0.0         255.255.255.0   U     0      0        0 ens33
   192.168.178.2   0.0.0.0         255.255.255.255 UH    100    0        0 ens33
   ```

   192.168.178.2是通外网的，设为默认路由

4. 启用

   ```
   netplan apply
   ```

5. 使用`ifconfig`测试正确性

6. 编辑`/etc/hosts`文件，添加解析



# 安装Kolla-ansible

## 【选】安装kolla-ansible的部署版

1. 安装kolla-ansible
    ```
    pip install kolla-ansible
    ```
    
2. 拷贝文件到工作目录 Copy globals.yml and passwords.yml to /etc/kolla directory
    ```
    cp -r /usr/local/share/kolla-ansible/etc_examples/kolla /etc/
    ```
3. 拷贝Inventory到当前目录
    ```
    cp /usr/local/share/kolla-ansible/ansible/inventory/* .
    ```

## 【荐】安装kolla-ansible开发模式

1. clone 两个仓库
    ```
    git clone https://github.com/openstack/kolla
    git clone https://github.com/openstack/kolla-ansible
    ```
    
2. 安装requirements.txt
    ```
    pip install -r kolla/requirements.txt
    pip install -r kolla-ansible/requirements.txt
    ```
    
3. 复制配置文件到 /etc/kolla
    ```
    mkdir -p /etc/kolla
    cp -r kolla-ansible/etc/kolla/* /etc/kolla
    ```
    
4. Copy the inventory files to the current directory. 
    ```
    cp kolla-ansible/ansible/inventory/* .
    ```

5. 如果要使用Kolla-ansible命令的话，就要进`kolla-ansible/tools`目录下使用`kolla-ansible`文件执行

# 杂项

1. 新建用户
    ```
    useradd csdn
    usermod -s /bin/bash csdn
    usermod -d /home/csdn csdn
    ```

2. 添加到sudoers
    ```
    chmod u+w /etc/sudoers
    vim /etc/sudoers
    
    csdn ALL=(ALL) ALL
    
    chmod u-w /etc/sudoers
    ```

3. 用户可能对自己的目录没有权限
    ```
    chown -R csdn /home/csdn
    chgrp -R csdn /home/csdn
    ```
