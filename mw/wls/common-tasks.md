# WebLogic 日常任务

本文的介绍需要在 weblogic10.3.6 的环境上才可以执行，如果没有环境，可以参考[文档](./wls-quickstart.md)


## 部署集群

部署一个集群，包含2个受管理服务器,分别在不同2个计算机上的。

### 环境概述

服务器情况：

|主机名|Ip|用途|
|----- |-----|----|
| wls1|172.16.0.151|部署管理服务器和一台受管理服务器|
| wls2|172.16.0.152|     部署另一台受管理服务器    |

集群部署情况：

| welogic | Ip | 用途 | 集群 |
|  ----  |  ----  |  ----  |  ----  |
|  AdminServer  |  172.16.0.151  |管理服务器|    |
|  server01  |  172.16.0.151  |受管理服务器| cluster01 |
|  server02  |  172.16.0.152  |受管理服务器| cluster01 |


### 创建域

具体步骤参考[文档](./wls-quickstart.md)中的“创建域”章节，因为是两台主机，所以在两台机器上都要做相同的操作，包括建域的安装目录以及域的名称。新建域的名称及安装目录如下：

|域名称|安装目录|
|----|----|
|pro_domain|/home/vagrant/Oracle/Middleware/user_projects/domains/base_domain|

### 创建集群

1. 进入控制台，先点击锁定并编辑，点击左侧域结构中的集群，新建集群，需要定义一个集群的名称，其他的默认  ，点击完成。
2. 点击新建好的集群，在集群设置里面添加服务器，在添加服务器时选择第二项创建新服务器并将其添加到此集群。




## 部署NodeManager 

### 使用nodeManger管理

1. 进入控制台，新建计算机
2. 新建计算机时，操作系统 windows 系统选为普通，unix、linux 等选为 unix，点击下一步
3. 类型选为普通，监听地址为自己的 ip， 地址点击完成
4. 回到 WebLogic 安装目录的 common/nodemanager 下，vim 打开 nodemanager.properties 文件，找到下面三项并更改为以下内容：
   
```powershell
    ListenAddress=自己的ip
    SecureListenr=false
    StartScriptEnabled=false
```

5. 启动 nodemanager，如果启动成功将会在控制台计算机见识选项下显示一下内容
状态：可访问
版本：10.3
6. 接下来就可以直接在控制台启动节点服务器了





## 巡检

### 系统配置

1. cpu
2. 内存

### Java配置

### Weblogic配置信息

### 控制台




### 基本监控：

  - 管理控制台监视：
    通过管理控制台，可以对 WebLogic 的性能以及运行状况，发布的应用，资源的进行监视
