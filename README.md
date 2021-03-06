# 中间件技术专家之路

本文提到的大部分资料可以在[相关资料下载](https://www.jianguoyun.com/p/DTf1CzQQko7ZCRjc74QE)中找到。还有一些资料属于保密学习材料，不提供对外下载。

推荐使用[专家工具箱](./toolkit/README.md)中提到的工具。

目录：

- [0.basic - 基础知识](#0basic---基础知识)
  - [0.1.计算机基础（cs-001）](#01计算机基础cs-001)
  - [0.2.Linux操作基础（linux-001）](#02linux操作基础linux-001)
- [1.tx-and-jee - 交易系统与 JEE（tx-001）](#1tx-and-jee---交易系统与-jeetx-001)
- [2.jee-mw - 各类 JEE 中间件](#2jee-mw---各类-jee-中间件)
  - [2.1.WLS](#21wls)
  - [2.2.WAS](#22was)
  - [2.3.Tomcat](#23tomcat)
- [3.private-mw - 各类非标准的中间件](#3private-mw---各类非标准的中间件)
  - [3.1.CICS](#31cics)
  - [3.2.Tuxedo](#32tuxedo)
- [4.mq-mw - 各类消息中间件](#4mq-mw---各类消息中间件)
  - [4.1.WMQ](#41wmq)

## 0.basic - 基础知识

### 0.1.计算机基础（cs-001）

操作系统、网络、密码学这些知识应该是从事计算机相关工作的基础，如果确实某种原因我们没有学好，那我们只能通过一些方式突击学习一下。[10分钟速成课：计算机科学](https://space.bilibili.com/5385034/channel/detail?cid=16059&ctype=0)是一个很棒的系列科普视频，大家有时间可以自己看看，以下是我认为可以优先观看的内容：

* [早期的计算](https://www.bilibili.com/video/BV1EW411u7th?p=1)
* [电子计算机](https://www.bilibili.com/video/BV1EW411u7th?p=2)
* [二进制](https://www.bilibili.com/video/BV1EW411u7th?p=4)
* [操作系统](https://www.bilibili.com/video/BV1EW411u7th?p=18)
* [文件系统](https://www.bilibili.com/video/BV1EW411u7th?p=20)
* [命令行界面](https://www.bilibili.com/video/BV1EW411u7th?p=22)
* [计算机网络](https://www.bilibili.com/video/BV1EW411u7th?p=28)
* [密码学](https://www.bilibili.com/video/BV1EW411u7th?p=33)

另外，阿里云的[面试题](https://hit-alibaba.github.io/interview/)也是一个很好的学习材料。

不过，专业人事还是应该更系统地学习，可以看看[这里](./basic/README.md)推荐的一些书。

### 0.2.Linux操作基础（linux-001）

Linux 操作基础也是一个必备条件，需要对 Linux 的基本概念和操作有所了解。“linux-001”是一份很好的学习材料，大家可以尝试学习。建议使用[文档](./toolkit/env/create-vms-with-vagrant-and-virtualbox.md)介绍的工具搭建虚拟机环境。

## 1.tx-and-jee - 交易系统与 JEE（tx-001）

了解 IT 系统的架构，历史背景，对我们工作的意义。

* 理解交易系统(见“相关资料下载”中的 PPT)
* JEE 介绍(见“相关资料下载”中的 PPT)

如果需要对这个话题有更深的认识，可以参考：

* [Principles of Transaction Processing（书）](https://book.douban.com/subject/3734011/)
* [计算机简史(书)](https://book.douban.com/subject/35043034/)

《计算机简史》虽然是讲故事为主，但是从中可以看到早期计算机的发展，计算机解决的业务问题，可以为你提供一个更朴素的视角。

## 2.jee-mw - 各类 JEE 中间件

### 2.1.WLS

WebLogic Server 是 Oracle 的一款 JEE 中间件，学习过程可以分为以下几个步骤：

* [WebLogic 的基本概念 :todo](:todo)
* [WebLogic 快速上手（实验）](./mw/wls/wls-quickstart.md)
* [WebLogic 应用开发（实验）](./mw/wls/jee-dev.md)
* [WebLogic 日常任务（实验）](./mw/wls/common-tasks.md)

如果希望深入的学习 WebLogic 的各个方面可以系统的阅读一下这几本书：

* Oracle WebLogic Server 12c Administration Handbook
* Oracle WebLogic Server 12c Advanced Administration Cookbook
* Professional Oracle WebLogic Server

### 2.2.WAS




### 2.3.Tomcat

## 3.private-mw - 各类非标准的中间件

### 3.1.CICS

CICS（ Customer Information Control System ）-IBM的交易中间件。详细的学习路径可以参考[这里](./mw/cics/README.md)。

### 3.2.Tuxedo

## 4.mq-mw - 各类消息中间件

### 4.1.WMQ

WMQ 是一种消息中间件，详细的学习路径可以参考[这里](./mw/wmq/README.md)。

