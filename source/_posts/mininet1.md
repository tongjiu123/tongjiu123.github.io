---
title: mininet1
date: 2022-09-13
categories:
  - problem
---

安装mininet坑

<!-- more -->

### 1.连接不上git clone git://github.com/mininet/mininet

网络问题，挂上蓝灯代理依然不行

解决方案：直接从github上下载

github.com/mininet/mininet

修改mininet-master文件名为mininet

```plaintext
cd mininet
util/install.sh -a
sudo mn --test pingall
```

此后可能出现问题openflow连接失败，见下一条

### 2.openflow

多尝试几次在mininet文件下

```plaintext
util/install.sh -0  #安装openflow1.0
或者
util/install.sh -3  #安装openflow1.3
```

此后会出现问题pox连接不上

### 3.pox

需要使用sudo权限

尝试只安装pox

```shell
sudo util/install.sh -p
```

后缀为p代表只安装pox

执行install.sh脚本

后缀分别有

```plaintext
-a 默认全部安装
-b 安装benchmark：oflops
-c安装核心之后清空已有配置
-d 删除某些敏感文件
-e 安装Mininet开发依赖
-f 安装OpenFolw协议支持
-h 打印帮助信息
-i 安装indigo Virtual Switch
-k 安装新的内核
-m 从源目录安装open vSwitch内核模块
-n 安装Mininet依赖和核心文件
-p 安装pox控制器
-r 删除已存在的open vSwitch包
-s 依赖源码
-t 完成其他的虚拟机创建任务
-v 安装Open switch
-V 指定Open vSwitch的版本
-w 安装Wireashark解析器
-x 安装NOX Classic控制器
-y 安装Ryu控制器
-0 安装openflow1.0
-3 安装openflow1.3
```

### 4.install ryu

安装其他版本python

本人安装python3.10.2

并指定优先级（除anaconda3中的3.7外）

步骤：

1、在Python官网下载想要安装Python版本的压缩包：https://www.python.org/

2、解压压缩包：

```shell
tar -xzvf Python-3.10.2.tgz
```

3、指定安装路径

```shell
cd Python-3.10.2
./configure --prefix=/usr/bin/python3.10 
sudo make
sudo make install
```

设置默认Python版本
update-alternatives系列命令对一个候选列表进行操作。

在这个候选列表中，我们可以：（1）添加候选Python版本；（2） 删除列表中已有的Python版本。

通过这个列表，我们可以：（1）手动为系统指定默认Python版本；（2）通过配置权重自动指定默认Python版本。

以root权限操作：

查看候选列表中已有的Python版本：

```plaintext
update-alternatives --list python
```

添加候选Python版本：

```plaintext
update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
update-alternatives --install /usr/bin/python python /usr/bin/python3 2
update-alternatives --install /usr/bin/python python /usr/bin/python3.10/bin/python3.10 3 # 自己后来安装的Python
```

语法：–install <链接> <名称> <路径> <优先级>，<链接> 需保持一致。优先级数字越大越高。

删除列表中已有的Python版本：

```plaintext
update-alternatives --remove python /usr/bin/python2.7
```

查看目前列表中每个Python版本的配置情况：

```plaintext
update-alternatives --config python
```

选择0，自动以优先级数字最大的为默认python版本，选择其他则手动指定版本。

此后，使用

```plaintext
pip3 install ryu
```

即可安装ryu

推测原因：ubuntu16自带的python出现了包的抵触

### 5. testbed无法运行run.sh

1. bad for loop variable

```plaintext
cd testbed
sudo ./run.sh
```

失败，显示bad for loop variable

解决方法：加上#!/bin/bash

2. 10461挂起 sudo：python找不到命令

缺少pbr

### 补充：mininet使用

###### 3.启动Mininet

安装完成后，通过sudo mn命令启动mininet，更多相关主要命令参考如下：

###### 1. 主要网络构建启动命令

–topo 制定拓扑类型或文件
–custom 自建拓扑
–switch 设置交换机类型
–controller 设置控制器类型
–mac 自动设置主机mac

###### 2. 内部交互主要命令

dump 输出节点信息
net 查看网络拓扑信息
nodes 查看全部节点信息
dpctl 操作datapath
iperf 制定节点之间的tcp
h1 ping h2 测试主机的连通性

###### 4.Mininet简单示例

1. 单一拓扑
sudo mn –topo=single，3
其中3为主机数目的设定参数，可更换其他。

2. 线形拓扑
sudo mn –topo=linear，4
对于线性拓扑，数字代表交换机数目和主机数目。

3. 树形拓扑
sudo mn –topo=tree，depth=2，fanout=2
depth代表深度，fanout代表扇出，即深度代表交换机的深度，扇出代表每个交换机下挂载主机数目。

4. 自定义拓扑
sudo mn –custom file.py –topo mytopo
file.py代表自己编写的拓扑脚本文件

参考：https://www.jianshu.com/p/71e29d487ea9
