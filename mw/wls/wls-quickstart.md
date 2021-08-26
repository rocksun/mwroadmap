# WebLogic 快速入门

## 环境准备

| 主机名 |      IP      |    角色     |
| :----: | :----------: | :---------: |
|  wls1  | 172.16.0.151 | AdminServer |

实验需要一个 Red Hat Enterprise Linux 7 虚拟机，名称为 wls1 , 如果没有环境, 可以参考此[文档](../../toolkit/env/create-vms-with-vagrant-and-virtualbox.md)。

## 准备介质

本实验使用 WebLogic10.3.6 版本，可以到[WebLogic的下载页面](http://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html)下找到版本10.3.6下的 Generic 。对应的 JDK 版本为 jdk1.7 ，可以到[JDK的下载页面](https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html#jdk-7u80-oth-JPR)中下载 jdk-7u80-linux-x64.tar.gz 。

如果使用前面的方式部署环境，宿主机的 weblogic 目录会映射到虚拟机的 weblogic_data， 把之前下载的介质放到 weblogic 目录下, 到虚拟机中可以看到:

```bash
[vagrant@wls2 weblogic_data]$ pwd
/weblogic_data
[vagrant@wls2 weblogic_data]$ ls
jdk-7u80-linux-x64.tar.gz   wls1036_generic.jar
```

## 安装 WebLogic

### 解压 JDK

这里将下载好的 JDK 解压到 /usr/java/ 目录下:

```shell
[vagrant@wls2 weblogic_data]$ sudo mkdir /usr/java
[vagrant@wls2 weblogic_data]$ sudo tar -xf jdk-7u80-linux-x64.tar.gz -C /usr/java/ 
```

### 静默安装

WebLogic 的静默安装方式，需要准备一个 silent.xml 文件，内容如下：

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

根据情况修改 BEAHOME，WLS_INSTALL_DIR 和 LOCAL_JVMS。然后开始安装：

```bash
[vagrant@wls2 weblogic_data]$ /usr/java/jdk1.7.0_80/bin/java -jar  wls1036_generic.jar   \
    -mode=silent      \
    -silent_xml=linux_silent.xml  \
    -log=/tmp/install_weblogic/weblogic_install.log \
    -Djava.io.tmpdir=/tmp/install_weblogic
```

参数说明：

* -mode: 指定安装模式为silent, 默认为console
* -silent_xml: 指定silent_xml文件路径
* -log: 指定安装时输出日志的存放位置
* -Djava.io.tmpdir: 在Unix/Linux平台上，如果提示临时空间不足，可以加上-Djava.io.tmpdir=tmpdirpath指定一块区域做临时空间
  
如果命令执行后有报错信息，可以通过指定的 /tmp/install_weblogic/weblogic_install.log 查看安装日志，查看详细的安装失败原因。

## 创建域

使用静默方式创建域, 首先需要创建域目录：

```shell
[vagrant@wls1 ~]$ mkdir -p /home/vagrant/Oracle/Middleware/user_projects/domains/base_domain/
```

静默建域需要指定 silent.rsp 文件，内容如下：

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

参数说明: 

* JavaHome： JDK的安装路径
* ServerStartMode： 服务启动时的模式，prod是生产模式
* AdminServer.ListenAddress： AdminServer的监听地址
* AdminServer.ListenPort: AdminServer的监听端口
* 创建用户weblogic，密码为weblogic123
* write domain to: 创建的域路径
  
根据实际环境修改响应文件，更多参数请参考 [WebLogic 静默建域官方文档](https://docs.oracle.com/cd/E13196_01/platform/docs81/confgwiz/silent.html#1043185)，然后可以执行静默安装命令:

```shell
[vagrant@wls2]$ cd /home/vagrant/Oracle/Middleware/wlserver_10.3/common/bin
[vagrant@wls2 bin]$ ./config.sh \
-mode=silent \
-silent_script=/home/vagrant/Oracle/Middleware/wlserver_10.3/common/bin/create_domain.rsp \
-logfile=/tmp/domain_create/create_domain.log
```

## 启动AdminServer

AdminServer 默认的启动脚本在域目录的 bin 下，在我们的环境里路径为 `/home/vagrant/Oracle/Middleware/user_projects/domains/base_domain/bin`。

服务启动时，需要交互式输入用户名密码，为了不用每次都手动输入，可以编写 boot.properties 文件。服务启动时会自动加载文件中的用户名和密码：

```shell
[vagrant@wls1 base_domain]$ mkdir -p servers/AdminServer/security
[vagrant@wls1 base_domain]$ cd servers/AdminServer/security/
[vagrant@wls1 security]$ cat > ./boot.properties <<EOF 
username=weblogic
password=weblogic123
EOF
```

在`/home/vagrant/Oracle/Middleware/user_projects/domains/base_domain/bin`目录下编写启动脚本，主要是为了修改JVM的启动参数, 不建议修改domain自动创建的脚本, 最好自己编写：

```shell
[vagrant@wls1 bin]$ cat > startAdmin.sh <<EOF
#!/bin/sh
ABSBINPATH=$(cd "$(dirname "$0")"; pwd)
export ABSBINPATH
export DERBY_FLAG=false
export USER_MEM_ARGS="-Xms512m -Xmx512m -Djava.security.egd=file:/dev/./urandom"
nohup ${ABSBINPATH}/startWebLogic.sh  http://172.16.0.151:7001>${ABSBINPATH}/../servers/AdminServer/logs/nohup.out 2>&1 &
EOF
```

执行启动脚本，可以到AdminServer下的 logs 目录中查看日志，启动成功后日志中会提示`Server started in RUNNING mode`：

```shell
[vagrant@wls1 base_domain]$ tail -f servers/AdminServer/logs/nohup.out
```

通过 netstat 命令查看对应端口，确保端口已监听:

```shell
[vagrant@wls1 ~]$ netstat -ntulp | grep :7001
```

为了保证能在浏览器中访问到控制台，需要关闭防火墙:
```shell
[vagrant@wls1 ~]$ sudo systemctl disable firewalld --now 
```

然后可以在宿主机的浏览登录控制台 `http://172.16.0.151:7001` ，登录用户名密码为创建 domain 时设置的用户名密码。


# 部署应用到 AdminServer

登录到 WebLogic 控制台，点击左上角的锁定并编辑， 依次点击左侧的部署 -> 安装 -> 上传文件，选择war包loginweb.war（下载链接见后边说明），点击将此安装部署为应用，下一步 -> 下一步 -> 完成 ，再点击左上角的保存并激活配置。返回控制台的部署界面查看，如果应用状态为 `ACTIVE` 表示部署成功。

部署应用成功后，可以直接输入控制台IP加页面名称进行访问，在我们的环境里，访问地址为: http://172.16.0.151:7001/loginweb。
部署失败的话可以到`/home/vagrant/Oracle/Middleware/user_projects/domains/base_domain/servers/AdminServer/logs/`下查看AdminServer的日志，确定具体的报错信息。

分享文章中测试用的war包，如果没有可以下载使用。
百度网盘链接: https://pan.baidu.com/s/1vr8hxVxjbFrnHdndfy0G2A 
提取码: 4ind