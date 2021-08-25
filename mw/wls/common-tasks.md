# WebLogic 日常任务

配置节点管理器管理受管理服务器

环境准备如下：
服务器情况：

 ```powershell
主机名	        Ip		    用途
ms		192.168.226.129	    安装管理服务器和一台受管理服务器
as1		192.168.226.130	    安装另一台受管理服务器
 ```

集群安装情况：

 ```powershell
Weblogic服务	        Ip			        用途
AdminServer		192.168.226.129	
cs			192.168.226.129 	    Weblogic集群
as1			192.168.226.129 	    第一台受管理服务器
as2			192.168.226.130 	    第二台受管理服务器
 ```

Weblogic 计算机配置情况：

 ```powershell
计算机名称	节点管理器IP	    节点管理器端口	关联的受管理服务器
Machine-0	192.168.226.129		5556		as1
Machine-1	192.168.226.130		5556		as2
 ```


## 创建域

1. 使用 root 用户登录后使用 #su weblogic 将用户切换到 weblogic 用户
2. 使用 cd 命令进入到 weblogic 安装目录的 /common/bin 下， ls 找到 config.sh 文件，使用 ./config.sh 命令执行 weblogic 服务域配置启动文件，启动文件初始化成功后，
3. 选择 "Create a new weblogic domain" ,创建一个新的服务域，Next 回车
4. 选择创建域的类型这里我们选择 "Choose Weblogic Platform components" 生成一个自动配置的域以支持下列产品，输入"1"回车(或 next 回车)
5. 选择应用模板类型，直接 next 回车
6. 指定 domin 域的名称， 使用默认值或者输入域名称，next 回车
7. 指定 domin 域的存储路径，使用默认值或者输入存储路径，next 回车
8. 设置 weblogic 用户密码，用于登录 weblogic 控制台，输入"2"，设置登录密码，next 回车，回到设置用户名和密码界面，输入"3"，确认用户密码，next 回车，回到设置用户名和密码界面，
注：密码长度必须大于8位，且必须包含数字类型，若输入的密码信息不符合规则，则返回错误的提示信息：

 ```powershell
    60050：用户"weblogic"的密码无效
    60045：密码必须至少为8位字母数字字符，且至少包含一个数字或特殊字符
 ```

9. 设置域的模式配置：（根据项目需要设置）

 ```powershell
   开发模式 
   生产模式
 ```
 
10.  绑定jdk
    - 绑定默认jdk版本
    - 绑定自己安装的jdk版本：选择"2"，输入自己安装的JDK路径回车，next回车

 ```powershell
    ->1|Sun SDK 1.7.0_80 @ /weblogic/jdk1.7.0_80
      2|Other Java SDK
 ```

11.  选择服务类型：默认选择管理服务器，输入1，next 回车
12.  服务器参数设置：服务名、监听地址、监听端口等信息，此处不做修改，使用默认值，直接 next 回车
13.  开始创建服务域。
14.  显示 "**** Domain Created Successfully! ****" 域创建成功，执行如下命令
执行 ./startWebLogic.sh 命令启动

## 配置里创建集群和server

创建 Server

首先进入控制台
1. 点开左边域结构中的环境
2. 点击服务器，点击左上角更改中心的锁定并编辑
3. 在服务器概要中点击新建
4. 在新建的服务器信息中填写服务器名称和服务器监听地址，更改服务器的监听端口
5. 点击下一步，点击完成。
   
创建集群

1. 在控制台中，点开左边域结构中的环境
2. 点击集群，点击左上角更改中心的锁定并编辑，
3. 在集群概要界面点击新建，给新建的集群命名，点击确定
4. 在集群概要界面点击新建好的集群名称，在界面设置上方点击配置-->服务器
5. 点击添加，点击第一项选择现有服务器并将其添加为此集群的成员，选择服务器，点击下一步完成
6. 或者选择第二项创建新服务器并将其添加到此集群（此处创建新服务器请看前一节创建server），点击完成。

## 配置相关目录和启动脚本

创建启动脚本前的准备工作
配置相关目录：

1. 进入到域目录下的servers文件夹中
创建与 server 的同名文件夹如“AdminServer”（如果存在多个server，则创建多个文件夹并分别执行下面的操作），并进入其中，创建文件夹“security”与文件夹“logs”
进入创建好的“security”文件夹，在其中创建并编辑“boot.properties”文件，编辑内容为：
```powershell
    username=（weblogic用户名）
    password=（密码）
```

启动脚本：

2. 创建启动脚本
例如创建“AdminServer”的启动脚本，进入域目录的“bin”目录下创建并编辑“start_AdminServer.sh”文件编辑内容为：

```powershell
#!/bin/sh

ABSBINPATH=$(cd "$(dirname "$0")"; pwd)
export ABSBINPATH
export DERBY_FLAG=false	

export USER_MEM_ARGS="-Xms512m -Xmx512m -XX:PermSize=256M -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Djava.security.egd=file:/dev/./urandom"

Nohup ${ABSBINPATH}/startWebLogic.sh >${ABSBINPATH}/../servers/AdminServer/logs/nohup.out 2>&1 &
```

其中部分内容需要根据情况做适当修改（例如，在为 AdminServer 创建启动脚本时所需的 JVM 参数参考值为：-Xms512m -Xmx512m）,编辑完成后，为 AdminServer.sh 添加可执行权限


## 部署应用和测试

在 weblogic 控制台部署应用、测试应用

1. 启动 weblogic 服务，打开管理控制台，登录
2. 左侧域结构下选择部署，也可在主页下方选择部署
3. 点击部署后进入部署概要界面，需点击左上方的更改中心的锁定并编辑按钮进行编辑
4. 点击部署概要界面的安装按钮
5. 在安装应用程序辅助界面
    - 确保 web 应用 war 包放置的路径正确
    - 点击上载文件，选择要部署的应用war包
    - 点击下一步按钮
6. 如果存在同名项目，会出现提示信息，提示您在接下来的部署中，另起一个项目名称来区分，选择定位样式：将此部署安装为应用程序点击下一步
7. 选择部署目标，勾选部署目标服务器的名称点击下一步
8. 在安装应用程序辅助程序界面，如有重名项目，需要在这里更爱项目名称以区分项目，选中将此应用程序复制到每个目标，点击下一步
9.  查看部署项目信息，核对后点击完成按钮，完成部署
10. 点击保存，点击左上角的激活更改
11. 所有的更改被激活，不需要重新启动，此时已经完成部署，部署后项目处于准备就绪状态，需要启动 web 应用
12. 在"部署-概要"界面中，勾选需要启动的 web 项目，点击启动，选择为所有请求提供服务，选"是"启动应用，启动后可以在"部署-概要"界面看到 web 应用处于活动状态
13. 检验项目是否成功，在浏览器输入，URL 服务器端口号 上下文根，来访问项目

## 使用nodeManger管理

1. 进入控制台，新建计算机
2. 新建计算机时，操作系统 windows 系统选为普通，unix、linux 等选为 unix，点击下一步
3. 类型选为普通，监听地址为自己的 ip， 地址点击完成
4. 回到 weblogic 安装目录的 common/nodemanager 下，vim 打开 nodemanager.properties 文件，找到下面三项并更改为以下内容：
   
```powershell
    ListenAddress=自己的ip
    SecureListenr=false
    StartScriptEnabled=false
```

5. 启动 nodemanager，如果启动成功将会在控制台计算机见识选项下显示一下内容
状态：可访问
版本：10.3
6. 接下来就可以直接在控制台启动节点服务器了

## 相关问题 基本监控

基本监控：
  - 管理控制台监视：
    通过管理控制台，可以对 Weblogic 的性能以及运行状况，发布的应用，资源的进行监视