# WebLogic 快速入门

实验需要两个 Red Hat Enterprise Linux 7 虚拟机，分别为 wls1 和 wls2 , 如果没有环境, 可以参考此[文档](../../toolkit/env/create-vms-with-vagrant-and-virtualbox.md)。

主机名|IP|角色
:-: | :-: | :-:
wls1|172.16.0.151|AdminServer

* WebLogic 版本：wlserver_10.3.6  [下载地址](http://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html)
  
* JDK 版本： jdk1.7.0_80  [下载地址](https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html#jdk-7u80-oth-JPR)

## 静默安装

首先是准备 JDK，WebLogic 介质（如果使用前面的方式部署环境，宿主机的 weblogic 目录会映射到虚拟机的 weblogic_data）：

```bash
[vagrant@wls2 weblogic_data]$ pwd
/weblogic_data
[vagrant@wls2 weblogic_data]$ ls
jdk-7u80-linux-x64.tar.gz  linux_silent.xml  wls1036_generic.jar
```

解压 JDK：

```shell
[vagrant@wls2 weblogic_data]$ sudo mkdir /usr/java
[vagrant@wls2 weblogic_data]$ sudo tar -xf jdk-7u80-linux-x64.tar.gz -C /usr/java/ 
```

静默安装需要创建 silent.xml 文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bea-installer>
<input-fields>
<data-value value="/home/vagrant/Oracle/Middleware" name="BEAHOME"/>
<data-value value="/home/vagrant/Oracle/Middleware/wlserver_10.3" name="WLS_INSTALL_DIR"/> 
<data-value value="WebLogic Server/Core Application Server|WebLogic Server/Administration Console|WebLogic Server/Configuration Wizard and Upgrade Framework|WebLogic Server/Web 2.0 HTTP Pub-Sub Server|WebLogic Server/WebLogic SCA|WebLogic Server/WebLogic JDBC Drivers|WebLogic Server/Third Party JDBC Drivers|WebLogic Server/WebLogic Server Clients|WebLogic Server/WebLogic Web Server Plugins|WebLogic Server/UDDI and Xquery Support|WebLogic Server/Evaluation Database|Oracle Coherence/Coherence Product Files" name="COMPONENT_PATHS"/>
<data-value value="no" name="INSTALL_NODE_MANAGER_SERVICE"/>
<data-value value="no" name="INSTALL_SHORTCUT_IN_ALL_USERS_FOLDER"/>
<data-value value="/usr/java/jdk1.7.0_80" name="LOCAL_JVMS"/>
</input-fields>
</bea-installer>
```

根据情况修改 BEAHOME，WLS_INSTALL_DIR 和 LOCAL_JVMS。然后开始安装，可以通过指定的安装日志查看安装结果：

```bash
[vagrant@wls2 weblogic_data]$ /usr/java/jdk1.7.0_80/bin/java -jar  wls1036_generic.jar   \
    -mode=silent      \
    -silent_xml=linux_silent.xml  \
    -log=/tmp/install_weblogic/weblogic_install.log \
    -Djava.io.tmpdir=/tmp/install_weblogic
```

其他说明：

* -mode: 指定安装模式为silent, 默认为console
* -silent_xml: 指定silent_xml文件路径
* -log: 指定安装时输出日志的存放位置
* -Djava.io.tmpdir: 在Unix/Linux平台上，如果提示临时空间不足，可以加上-Djava.io.tmpdir=tmpdirpath指定一块区域做临时空间

## 创建域

使用静默方式创建域, 首先需要创建域目录：

```shell
[vagrant@wls1 ~]$ mkdir -p /home/vagrant/Oracle/Middleware/user_projects/domains/base_domain/
[vagrant@wls1 ~]$ 
```

静默建域需要指定 silent.rsp 文件，可以参考 [WebLogic 静默建域官方文档](https://docs.oracle.com/cd/E13196_01/platform/docs81/confgwiz/silent.html#1043185)：

```shell
read template from "/home/vagrant/Oracle/Middleware/wlserver_10.3/common/templates/domains/wls.jar";
set JavaHome "/usr/java/jdk1.7.0_80";
set ServerStartMode "prod";
find Server "AdminServer" as AdminServer;
set AdminServer.ListenAddress "";
set AdminServer.ListenPort "7001";
find User "weblogic" as weblogic;
set weblogic.password "weblogic123";
write domain to "/home/vagrant/Oracle/Middleware/user_projects/domains/base_domain";
close template;      
``` 

其他说明: 

* JavaHome： JDK的安装路径
* ServerStartMode： 服务启动时的模式，prod是生产模式
* AdminServer.ListenAddress： AdminServer的监听地址
* AdminServer.ListenPort: AdminServer的监听端口
* 创建用户weblogic，密码为weblogic123
* write domain to: 创建的域路径
  
要按照实际环境填写响应文件，如果响应文件编写的有问题，是无法成功建域的，执行安装静默安装命令:

```shell
[vagrant@wls2]$ cd /home/vagrant/Oracle/Middleware/wlserver_10.3/common/bin
[vagrant@wls2 bin]$ ./config.sh \
-mode=silent \
-silent_script=/home/vagrant/Oracle/Middleware/wlserver_10.3/common/bin/create_domain.rsp \
-logfile=/tmp/domain_create/create_domain.log
```

## 启动AdminServer

AdminServer默认的启动脚本在域目录下：

```shell
[vagrant@wls1 base_domain]$ pwd
/home/vagrant/Oracle/Middleware/user_projects/domains/base_domain
[vagrant@wls1 base_domain]$ ls
autodeploy  config       fileRealm.properties  lib    security  
bin         console-ext  init-info             startWebLogic.sh
```

创建AdminServer的启动目录，AdminServer启动时，需要交互式输入用户名密码。如果想使用脚本启动可以在对应的服务目录下创建 security 文件夹，编写存放用户名和密码的 boot.properties 文件：

```shell
[vagrant@wls1 base_domain]$ mkdir -p servers/AdminServer/security
[vagrant@wls1 base_domain]$ cd servers/AdminServer/security/
[vagrant@wls1 security]$ touch boot.properties
```

boot.properties 文件存放 AdminServer 登录的用户名密码即可，服务启动时会自动加载boot.properties文件,不用手动输入用户名密码：

```shell
username=weblogic
password=weblogic123
```

可以在domains目录下编写启动脚本，主要是为了修改JVM的启动参数, 不建议修改默认脚本, 最好自己编写：

```shell
[vagrant@wls1 base_domain]$ vim startAdmin.sh
#!/bin/sh
ABSBINPATH=$(cd "$(dirname "$0")"; pwd)
export ABSBINPATH
export DERBY_FLAG=false

export USER_MEM_ARGS="-Xms512m -Xmx512m -Djava.security.egd=file:/dev/./urandom"
nohup ${ABSBINPATH}/startWebLogic.sh  http://172.16.0.151:7001>${ABSBINPATH}/../servers/AdminServer/logs/nohup.out 2>&1 &
```

执行启动脚本，启动成功后日志中会提示`Server started in RUNNING mode`：

```shell
[vagrant@wls1 base_domain]$ vim startAdmin.sh
[vagrant@wls1 base_domain]$ tail -f servers/AdminServer/logs/nohup.out
```

通过netstat命令查看对应端口:

```shell
[vagrant@wls1 ~]$ netstat -ntulp | grep :7001
```

浏览器输入网址验证，需要关闭服务器的防火墙: 

```shell
[vagrant@wls1 ~]$ systemctl disable firewalld --now 
```

# 部署应用到 AdminServer

将写好的[war包](可以用自己写的做测试)上传到域文件目录下:

```shell
[vagrant@wls2 bin]$ mv ./loginweb.war  ~/Oracle/Middileware/user_projects/domains/base_domain/
[vagrant@wls2 bin]$ cd ~/Oracle/Middileware/user_projects/domains/base_domain/
[vagrant@wls2 base_domain]$ sudo ./startWebLogic.sh
```

将 war 包上传到域目录下，登录到 WebLogic 控制台，点击 base_domain 中的部署 (生产模式需要点击锁定并编辑)，部署操作完成后一定要激活所有配置。部署完成后可以输入网址测试，比如本文中的测试地址为: http://172.16.0.151:7001/loginweb
