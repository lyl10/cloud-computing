# cloud-computing 云计算

***
## 目录简介

**basic**:（实验1）

+ README.md 
[GitHub基本使用](https://github.com/lyl10/cloud-computing/blob/master/basic/README.md)

+ README1.md  
[购买腾讯云服务器，Xshell的简单配置](https://github.com/lyl10/cloud-computing/blob/master/basic/README1.md)

**Website**：（实验2）

+ 2.md
[centos下Wordpress个人网站的搭建](https://github.com/lyl10/cloud-computing/blob/master/Website/2.md)

**docker**：(实验3)

+ 1.md
[Docker基础实验]()
+ 2.md
[利用Dockerfile文件创建包含WordPress的镜像]()

## 云计算概念

    云计算（cloud computing）是分布式计算的一种，指的是通过网络“云”将巨大的数据计算处理程序分解成无数个小程序，然后，通过多部服务器组成的系统进行处理和分析这些小程序得到结果并返回给用户。云计算早期，简单地说，就是简单的分布式计算，解决任务分发，并进行计算结果的合并。因而，云计算又称为网格计算。通过这项技术，可以在很短的时间内（几秒种）完成对数以万计的数据的处理，从而达到强大的网络服务。

    现阶段所说的云服务已经不单单是一种分布式计算，而是分布式计算、效用计算、负载均衡、并行计算、网络存储、热备份冗杂和虚拟化等计算机技术混合演进并跃升的结果

## 虚拟化技术

+ Docker
+ KVM
+ Xen

## Docker的基本概念

Docker包含了三个基本的概念将在本文中出现：镜像、容器、仓库。

- 镜像（Images）
- 容器（Containers）
- 仓库（Repositories）

**镜像**

>只读的模板，包含可以创建容器的指令，类比于面向对象中的类。镜像采用分层设计的方式。上层的镜像依赖于下层的镜像，并且包含相关的配置。例如，图中为一个ubuntu15.04的镜像，你可以基于此镜像安装Apache Web服务，并进行相关配置，从而形成新的镜像。使用docker images命令可以列出本地的所有镜像。

**容器**

>容器是镜像的一个运行实例，类比于面向对象中类的实例对象，如上图中最上层。你可以创建、运行、停止、移动和删除容器。容器的读写并不会写到镜像之中，并且Docker可以很好的隔离多个运行的容器。使用docker ps -a命令可以列出本地所有的容器，包括非活跃的容器。

**仓库**

>仓库是镜像集中存放的地方。你可以把镜像存放与本地，但若想要共享镜像，需要一种镜像分发服务，比如Docker Registry。比较有名的公共Registry服务如Docker官方的Docker Hub，本文后续将使用到。

### 分布式算法

+Google Chubby
+ZooKeeper

## 课程使用平台及技术方案

+ 腾讯云
+ GitHub


