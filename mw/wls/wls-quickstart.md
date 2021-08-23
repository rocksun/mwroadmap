# WebLogic 快速入门

实验需要两个 Red Hat Enterprise Linux 7 虚拟机，分别为 wls1 和 wls2 , 如果没有环境, 可以参考此[文档](../../toolkit/env/create-vms-with-vagrant-and-virtualbox.md)。

<table>
    <tr>
      <td>主机名</td>
      <td>IP</td>
      <td>角色</td>
    </tr>
    <tr>
      <td>wls1</td>
      <td>172.16.0.151</td>
      <td>AdminServer</td>
    </tr>
        <tr>
      <td>wls2</td>
      <td>172.16.0.152</td>
      <td>ManagedServer</td>
    </tr>
</table>

* WebLogic 版本：wlserver_10.3.6
* JDK 版本： jdk1.7.0_80

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
mkdir -p /home/vagrant/Oracle/Middleware/user_projects/domains/base_domain/
```

静默建域需要指定 silent.rsp 文件:
可以参考 [WebLogic 静默建域官方文档](https://docs.oracle.com/cd/E13196_01/platform/docs81/confgwiz/silent.html#1043185)：

```shell
read template from "/home/weblogic/oracle/wlserver/common/templates/wls/wls.jar";
set JavaHome "/usr/local/jdk1.7.0_80";
set ServerStartMode "prod";
find Server "AdminServer" as AdminServer;
set AdminServer.ListenAddress "";
set AdminServer.ListenPort "7001";
create Cluster "Cluster-0" as Clustername1;
create Machine "machine-1" as Machinename1;
create Server "Server-1" as BASE;
set BASE.ListenAddress "";
set BASE.ListenPort "8001";
set BASE.cluster "Cluster-0";
set BASE.machine "machine-1";
create Server "Server-2" as BASE2;
set BASE2.ListenAddress "";
set BASE2.ListenPort "8002";
set BASE2.cluster "Cluster-0";
set BASE2.machine "machine-1";
find User "weblogic" as weblogic;
set weblogic.password "weblogic123";
write domain to "/home/weblogic/oracle/user_projects/domains/test_domain/";
close template;
                 
``` 

 要按照实际环境填写响应文件，如果响应文件编写的有问题，是无法成功建域的。

执行安装静默安装命令:

```shell
[vagrant@wls2]$ cd /home/vagrant/Oracle/Middleware/wlserver_10.3/common/bin
[vagrant@wls2 bin]$ ./config.sh \
-mode=silent \
-silent_script=/home/vagrant/Oracle/Middleware/wlserver_10.3/common/bin/create_domain.rsp \
-logfile=/tmp/domain_create/create_domain.log
```

## 启动AdminServer

编写AdminServer启动脚本，主要是为了修改JVM的启动参数, 不建议修改默认脚本, 最好自己编写：

```shell
#!/bin/sh
ABSBINPATH=$(cd "$(dirname "$0")"; pwd)
export ABSBINPATH
export DERBY_FLAG=false

export USER_MEM_ARGS="-Xms512m -Xmx512m -Djava.security.egd=file:/dev/./urandom"
nohup ${ABSBINPATH}/startWebLogic.sh AdminServer http://172.16.0.151:7001>${ABSBINPATH}/../servers/AdminServer/logs/nohup.out 2>&1 &
```

WebLogic服务启动时，需要交互式输入用户名密码。使用脚本启动时可以在对应的服务目录下创建security文件夹，编写存放用户名和密码的boot.properties文件:

```shell
[vagrant@wls1 servers]$ cd AdminServer/
[vagrant@wls1 AdminServer]$ pwd
/home/vagrant/Oracle/Middleware/user_projects/domains/base_domain/servers/AdminServer
[vagrant@wls1 AdminServer]$ ls
  security
[vagrant@wls1 AdminServer]$ vim security/boot.properties
  username=weblogic
  password=weblogic123
```

`服务启动时会自动加载boot.properties文件,不用手动输入用户名密码。`

执行启动脚本，查看服务启动状态:
   - 通过netstat命令查看对应端口。
   - 浏览器输入网址验证，需要关闭服务器的防火墙: 

```shell
[vagrant@wls1 ~]$ netstat -ntulp | grep :7001
[vagrant@wls1 ~]$ systemctl disable firewalld --now 
```

# 部署应用到 Admin

将写好的[war包](可以用自己写的做测试)上传到域文件目录下:

```shell
[vagrant@wls2 bin]$ mv ./loginweb.war    ~/Oracle/Middileware/user_projects/domains/base_domain/
[vagrant@wls2 bin]$ cd ~/Oracle/Middileware/user_projects/domains/base_domain/
[vagrant@wls2 base_domain]$ sudo ./startWebLogic.sh

```

上传完成后，可以通过浏览器访问页面验证， 比如本文中的测试地址为: http://172.16.0.152:7001/loginweb
