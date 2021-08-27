# WebLogic 日常任务
- [部署集群](#部署集群)
  - [环境概述](#环境概述)
  - [创建域](#创建域)
  - [配置集群](#配置集群)
    - [创建集群配置](#创建集群配置)
    - [初始化 server 目录](#初始化-server-目录)
  - [部署NodeManager](#部署nodemanager)
  - [使用nodeManger管理](#使用nodemanger管理)
- [巡检](#巡检)
  - [系统配置](#系统配置)
  - [Java配置](#java配置)
  - [Weblogic配置信息](#weblogic配置信息)
  - [控制台](#控制台)


本文的介绍需要在 weblogic10.3.6 的环境上才可以执行，如果没有环境，可以参考[文档](./wls-quickstart.md)。


## 部署集群

部署一个集群，包含2个受管理服务器，分别在不同的2个计算机上。

### 环境概述

服务器情况：

|主机名|Ip|用途|
|:-----: |:-----:|:----:|
| wls1|172.16.0.151|部署管理服务器和一台受管理服务器|
| wls2|172.16.0.152|     部署另一台受管理服务器    |

集群部署情况：

| welogic | Ip | 用途 | 集群 |
|  :----:  |  :----:  |  :----:  |  :----:  |
|  AdminServer  |  172.16.0.151  |管理服务器|    |
|  server01  |  172.16.0.151  |受管理服务器| cluster01 |
|  server02  |  172.16.0.152  |受管理服务器| cluster01 |


### 创建域

具体步骤参考[文档](./wls-quickstart.md#创建域)中的“创建域”章节，因为是两台主机，所以在两台机器上都要做相同的操作，包括建域的安装目录以及域的名称。新建域的名称及安装目录如下：

|域名称|安装目录|
|:----:|:----:|
|pro_domain|/home/vagrant/Oracle/Middleware/user_projects/domains/base_domain|

### 配置集群

#### 创建集群配置

  - 进入控制台，先点击锁定并编辑，点击左侧域结构中的集群，新建集群，需要定义一个集群的名称，其他的默认，点击完成。
  - 点击新建好的集群，在集群设置里面添加服务器，在添加服务器时选择第二项创建新服务器并将其添加到此集群。
  - 激活更改。


#### 初始化 server 目录

AdminServer 初始化目录以及创建启动脚本请参考[文档](./wls-quickstart.md#启动AdminServer)

进入到域目录下的 servers 文件夹中，创建与 server 的同名文件夹如“server01”（如果存在多个 server，则创建多个文件夹并分别执行下面的操作）并进入其中，创建文件夹“security”与文件夹“logs”

```shell
[vagrant@wls1 base_domain]$ mkdir -p servers/AdminServer/security
[vagrant@wls1 base_domain]$ cd servers/AdminServer/security/
```

进入创建好的“security”，在其中创建并编辑“boot.properties”文件，编辑内容为：

```shell
[vagrant@wls1 security]$ cat > ./boot.properties <<EOF 
username=weblogic
password=weblogic123
EOF
```

创建启动脚本
创建“server01”的启动脚本，进入域目录的“bin”目录下创建并编辑“start_server01.sh”文件编辑内容为：

```shell
[vagrant@wls1 bin]$ cat > start_server01.sh <<EOF
#!/bin/sh
ABSBINPATH=$(cd "$(dirname "$0")"; pwd)
export ABSBINPATH
export DERBY_FLAG=false
export USER_MEM_ARGS="-Xms512m -Xmx512m -Djava.security.egd=file:/dev/./urandom"
nohup ${ABSBINPATH}/startManagedWebLogic.sh server01 http://172.16.0.151:7001>${ABSBINPATH}/../servers/server01/logs/nohup.out 2>&1 &
EOF
```

执行启动脚本，可以到 server01 下的 logs 目录中查看日志，启动成功后日志中会提示`Server started in RUNNING mode`：

```shell
[vagrant@wls1 base_domain]$ tail -f servers/server01/logs/nohup.out
```

通过 netstat 命令查看对应端口，确保端口已监听：

```shell
[vagrant@wls1 ~]$ netstat -ntlp | grep :7001
```

初始化 server02 目录，操作步骤与 server01 相同，只需要更改一下 server 名称和日志的目录



### 部署NodeManager 

### 使用nodeManger管理

1. 进入控制台，新建计算机
2. 新建计算机时，操作系统 windows 系统选为普通，unix、linux 等选为 unix，点击下一步
3. 类型选为普通，监听地址为自己的 ip 地址点击完成
4. 回到 WebLogic 安装目录的 common/nodemanager 下，vim 打开 nodemanager.properties 文件，找到下面三项并更改为以下内容：

```shell
    ListenAddress=自己的ip
    SecureListenr=false
    StartScriptEnabled=false
```

1. 启动 nodemanager，如果启动成功将会在控制台计算机见识选项下显示一下内容
状态：可访问
版本：10.3
6. 接下来就可以直接在控制台启动节点服务器了





## 巡检

### 系统配置

1. cpu
2. 内存
3. 文件系统
4. 操作版本

### Java配置

1. JDK版本

### Weblogic配置信息

### 控制台