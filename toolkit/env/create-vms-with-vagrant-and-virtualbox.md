# 使用 VirtualBox 和 Vagrant 快速搭建虚拟环境

VirtualBox是类似于VMWare的虚拟机工具，开源且跨平台。Vagrant是开源的管理虚拟机管理软件，通过Vagrant，结合VirtualBox，避免了手工配置虚拟机参数，实现了虚拟机环境的快速搭建。

本文介绍了如何使用VirtualBox和Vagrant快速搭建虚拟机环境，希望这种方式可以成为我们以后配置虚拟机环境的标准方式。

## 基本操作步骤

### 软件下载及安装

VirtualBox  [点击这里进行下载](https://www.virtualbox.org/)
Vagrant     [点击这里进行下载](https://www.virtualbox.org/)

按照提示安装即可,并无特别的设置。

VirtualBox安装完成后，需要将VirtualBox命令添加到系统路径，默认情况下路径为 C:\Program Files\Oracle\VirtualBox。在“我的电脑”点击右键，选择属性，然后依次点选“高级系统设置>高级>环境变量”,选择PATH，在PATH的值中添加“;C:\Program Files\Oracle\VirtualBox”。

### 创建项目

在自己心仪的位置创建一个工程文件夹，文件夹名称可根据自己的习惯自行设置，本教程使用的名称路径为 "D:\Project\vagrant\rhel7"，之后进行以下操作：

1.在该目录中创建一个文件夹“weblogic”作为之后windows与linux之间的共享文件夹，方便日后的文件传输；
2.在该目录中创建一个文件“Vagrantfile”作为Vagrant的配置文件。

“Vagrantfile”的内容如下：

```powershell

VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT
# Silly Ubuntu 12.04 doesn't have the
# --stdin option in the passwd utility
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
可直接将其复制到自己创建的“Vagrantfile”中，关于本文档的详细介绍请看最后一章。

### 根据配置安装虚拟机

在当前目录使用命令行输入

```powershell
vagrant up
```

第一次使用本命令即可根据配置文件“Vagrantfile”进行虚拟机的安装

### Vagrant 的初步操作

再次使用命令行输入

```powershell
vagrant up
```

即可打开配置的两个虚拟机wls1，wls2,使用ssh工具(如Xshell)进行链接，连接IP为：

```powershell
# wls1 IP
172.16.0.151
# wls2 IP
172.16.0.152
```

连接使用的用户名为vagrant，密码分别使用其对应的秘钥(例如wls1秘钥的路径为：“./vagrant/wls1/private_key”)。

登陆成功后进入到根目录下即可看到一个名为“weblogic_data”的文件夹，当在windows中的“./weblogic”目录中放置或更改文件时，这些操作也会同步到虚拟机中的“weblogic_data”文件夹。
