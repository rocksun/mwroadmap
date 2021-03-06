# 使用 VirtualBox 和 Vagrant 快速搭建虚拟环境

VirtualBox 是类似于 VMWare 的虚拟机工具，开源且跨平台。Vagrant 是开源的管理虚拟机管理软件，通过 Vagrant，结合 VirtualBox，避免了手工配置虚拟机参数，实现了虚拟机环境的快速搭建。

本文介绍了如何使用 VirtualBox 和 Vagrant 快速搭建虚拟机环境，希望这种方式可以成为我们以后配置虚拟机环境的标准方式。

## 基本操作步骤

### 软件下载及安装

* https://www.virtualbox.org/wiki/Downloads
* https://www.vagrantup.com/downloads

按照提示安装即可，并无特别的设置。

### 创建项目

在自己心仪的位置创建一个目录，目录名称可根据自己的使用习惯自行设置，本次教程使用的目录为 D:\Project\vagrant\rhel7。在该目录中创建一个子目录 weblogic 和 Vagrantfile 文件。

* 目录 weblogic 是 windows 与 linux 之间的共享文件夹，之后可以通过此目录传输文件
* 文件 Vagrantfile 是 Vagrant 在配置虚拟机时的需要的配置文件。


Vagrantfile的内容如下：

```powershell

VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT

echo root:vagrant | chpasswd

cat << EOF >> /etc/hosts
172.16.0.151 wls1
172.16.0.152 wls2
EOF
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    
    config.vm.define :wls1 do |wls1_config|
      wls1_config.vm.box = "generic/rhel7"
      wls1_config.vm.provision "shell", inline: $script
      
      
      wls1_config.vm.hostname = "wls1"
      wls1_config.vm.network "private_network", ip: "172.16.0.151"
      
      wls1_config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", "1536"]
          v.customize ["modifyvm", :id, "--cpus", "1"]
      end
    end
  
    
    config.vm.define :wls2 do |wls2_config|
      wls2_config.vm.box = "generic/rhel7"
      wls2_config.vm.provision "shell", inline: $script
      
      
      wls2_config.vm.hostname = "wls2"
      wls2_config.vm.network "private_network", ip: "172.16.0.152"
      
      wls2_config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", "1536"]
          v.customize ["modifyvm", :id, "--cpus", "1"]
      end
    end
  
    config.vm.synced_folder "weblogic", "/weblogic_data"

  end

```

### 启动虚拟机

在 Vagrantfile 所在目录的命令行中输入：

```powershell
vagrant up
```

如果没有报错则启动成功。

### 验证虚拟机

输入指令：

```powershell
#启用SSH登陆 wls1
vagrant ssh wls1
cd /
ls
```

可看到一个名为 weblogic_data 的文件夹，则启动成功。

## 一些 Vagrant 的基本操作

查看当前虚拟机状态


```powershell
vagrant status
```

启动

```powershell
#配置文件中的所有虚拟机全部启动
vagrant up
#单独启动 wls1
vagrant up wls1
```

启用SSH登陆虚拟机

```powershell
#启用SSH登陆 wls1
vagrant ssh wls1
```

停止

```powershell
#配置文件中的所有虚拟机全部停止
vagrant halt
#单独停止 wls1
vagrant halt wls1
```

销毁

```powershell
#销毁当前目录下所有的虚拟机
vagrant destroy
#销毁当前目录下的 wls1
vagrant destroy wls1
```
##### 备注：
例如在使用Xshell登录时，登录虚拟机使用的用户名为 vagrant，在输入密码处使用秘钥登录，秘钥的路径为 ./.vagrant/主机名/private_key。

## 参考资料

Vagrant基本命令详解
https://blog.csdn.net/chszs/article/details/51925179

vagrant box各种命令汇总
https://www.cnblogs.com/lovebing/p/9509923.html