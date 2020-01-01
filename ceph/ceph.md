# Ceph的安装与实践

***

## 步骤1 安装配置虚拟机

1.下载最小化安装镜像

[镜像下载链接](https://mirrors.tuna.tsinghua.edu.cn/centos/7.7.1908/isos/x86_64/CentOS-7-x86_64-Minimal-1908.iso)

centos7 Minimal 虚拟机安装[图文教程](https://blog.csdn.net/Fly_Du_/article/details/88734553)

2.进入系统后，想要粘贴代码很不方便，用ssh软件连接VMware虚拟机

   先查询ip：ip addr


![](media/89aea9f30f3bcae07c131864196ccf87.png)


3.在shell软件上新建连接


![](media/93a2b2703a204d84e5400acb00566cfc.png)


4.连接成功

![](media/50bbcfa56bfffe7046a58c82242673fa.png)


![](media/9ac0c4a6d1e3f14f2bbd7a1671acc75d.png)

## 步骤2-配置所有节点

### 创建一个Ceph用户


1.在所有节点上创建一个名为“ **cephuser** ” 的新用户。

>*useradd -d /home/cephuser -m cephuser*  
>*passwd cephuser*

![](media/aee791ccf610d1d1c36242fa72ba154a.png)

2.创建新用户后，我们需要为“cephuser”配置sudo。他必须能够以root用户身份运行命令，并且无需密码即可获得root用户特权。

运行以下命令为用户创建一个sudoers文件，并使用sed编辑/ etc / sudoers文件。

>*echo "cephuser ALL = (root) NOPASSWD:ALL" \| sudo tee /etc/sudoers.d/cephuser*  

>*chmod 0440 /etc/sudoers.d/cephuser*  

>*sed -i s'/Defaults requiretty/\#Defaults requiretty'/g /etc/sudoers*

![](media/3eb3c4bb2833685ca94fa6ce05ceb1b2.png)

### 安装和配置NTP

1.安装NTP以同步所有节点上的日期和时间。运行ntpdate命令通过NTP协议设置日期和时间，我们将使用us
pool NTP服务器。然后启动并启用NTP服务器在引导时运行。

>*yum install -y ntp ntpdate ntp-doc*  

>*ntpdate 0.us.pool.ntp.org*  

>*hwclock --systohc*  

>*systemctl enable ntpd.service*  

>*systemctl start ntpd.service*

![](media/b2e2fa5009aaff5fd72e61cd56904459.png)

![](media/0df2314ed4d264d5bfd56a24b46884d4.png)

### 安装Open-vm-tools

如果要在VMware内部运行所有节点，则需要安装此虚拟化实用程序。否则，请跳过此步骤。

>*yum install -y open-vm-tools*

![](media/4e2cf29223e2d877cc6f746ab918df21.png)

### 禁用SELinux

通过使用sed流编辑器编辑SELinux配置文件，在所有节点上禁用SELinux。

*sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config*

![](media/9e93d22a9e5885b5bc1951745f018714.png)

*yum -y install vim*

![](media/5e3f76e6e325d69f670ae520d3f43a57.png)

### 配置主机文件

使用vim编辑器在所有节点上编辑/ etc /
hosts文件，并添加带有所有集群节点的IP地址和主机名的行。

*vim /etc/hosts*

![](media/412a783f4b5d4ad8ad9fd4363b111a94.png)

粘贴以下配置：

*192.168.42.145 ceph-admin*

*192.168.42.143 ceph-0*

*192.168.42.141 ceph-1*

*192.168.42.144 ceph-2*

![](media/1de7e2ff1337fed5355e07fcf0a9328e.png)

保存文件并退出vim。

到目前为止，全部节点相同步骤都已配置完毕，关机克隆复制虚拟机

![](media/5a1d4d47ee88e88f6737e1d2f4803991.png)

开启四台虚拟机

现在，您可以尝试使用主机名在服务器之间ping通，以测试网络连接。例：

*su - cephuser*

*ping -c 5 ceph-0*

[./media/image15.png](./media/image15.png)
------------------------------------------

![](media/6e747c4c9e69367b339902cb15bdefe8.png)

![](media/771cd6530ecbc8dabba37da96955253b.png)

## 步骤3-配置SSH服务器
-------------------

1.配置**ceph-admin节点**。
admin节点用于配置监视节点ceph-0和osd节点ceph-1、ceph-2。登录到**ceph** -admin节点并成为“ **cephuser** ”。

> su - cephuser

admin节点用于安装和配置所有群集节点，因此ceph-admin节点上的用户必须具有无需密码即可连接到所有节点的特权。
2.在“ceph-admin”节点上"cephuser”配置无密码的SSH访问。

为“ **cephuser** ” 生成ssh密钥。

>*ssh-keygen*

![](media/5eab1f0fbb1f53ece21111676c58d902.png)

将密码短语留空/空白。

接下来，为ssh配置创建配置文件。

>*vim \~/.ssh/config*

粘贴以下配置：

>*Host ceph-admin*  
        *Hostname ceph-admin*  
        *User cephuser*   
>*Host ceph-0*  
        *Hostname ceph-0*  
        *User cephuser*  
*Host ceph-1*  
        *Hostname ceph-1*  
        *User cephuser*  
*Host ceph-2*  
        *Hostname ceph-2*  
        *User cephuser*  


![](media/57dc4772bffdd60e69411337f086ae14.png)

保存文件。

更改配置文件的权限。

>*chmod 644 \~/.ssh/config*

现在，使用ssh-copy-id命令将SSH密钥添加到所有节点。

>*ssh-keyscan ceph-0 ceph-1 ceph-2 \>\> \~/.ssh/known_hosts*

![](media/1fcd5534b0092e4e3fbbc379d133200d.png)

分发密钥

>*ssh-copy-id ceph-admin*

*ssh-copy-id ceph-0*  
*ssh-copy-id ceph-1*  
*ssh-copy-id ceph-2*

![](media/715194dddbf69aa1872a245c675e1ee3.png)

![](media/7049f65f246d52b3e9233d96459c4cbe.png)

![](media/57b7f4139861a8c109e2aef23896d5f3.png)

![](media/829b295a56495f4c741b58329286f848.png)

根据要求输入您的“ cephuser”密码。

完成后，请尝试从ceph-admin节点访问osd1服务器ceph-1。

验证是否免密成功

>*ssh ceph-1*

[./media/image25.png](./media/image25.png)
------------------------------------------

### 步骤4-配置防火墙
----------------

我们将使用防火墙保护系统。在此步骤中，我们将在所有节点上启用firewald，然后打开ceph-admon，ceph-mon和ceph-osd所需的端口。

登录到ceph-admin节点并启动firewalld。

>*systemctl start firewalld*  
>*systemctl enable firewalld*

打开端口80、2003和4505-4506，然后重新加载防火墙。

>*sudo firewall-cmd --zone=public --add-port=80/tcp --permanent*  
>*sudo firewall-cmd --zone=public --add-port=2003/tcp --permanent* >*sudo firewall-cmd --zone=public --add-port=4505-4506/tcp --permanent*  
>*sudo firewall-cmd --reload*

![](media/4d3ef0d1f2f854f8fa9a481753818fc6.png)

打开监视节点“ ceph-0”，然后启动firewalld。

>*systemctl start firewalld*  
>*systemctl enable firewalld*

![](media/a9c55acdd6024d0ffb76b7a680c3f55e.png)

在Ceph监视节点上打开新端口，然后重新加载防火墙。

>*sudo firewall-cmd --zone=public --add-port=6789/tcp --permanent*  
>*sudo firewall-cmd --reload*

![](media/7fb6b6acb3bbf35c8dbf71ca995cf895.png)

最后，打开每个osd节点上的端口6800-7300-osd1，osd2和os3。

登录到每个osd节点。

>*sudo systemctl start firewalld*  
>*sudo systemctl enable firewalld*

打开端口并重新加载防火墙。

>*sudo firewall-cmd --zone=public --add-port=6800-7300/tcp --permanent*  

>*sudo firewall-cmd --reload*

osd节点*ceph-1*

![](media/3806b02f3e711d8734f2db08925e5623.png)

![](media/567d239716f06083d549eedeb821f4f0.png)

osd节点*ceph-2*

![](media/b4883814198a41b29c6b913325ee865d.png)

![](media/36c6f6c913c5582c2dfedbc519cd6c1f.png)

防火墙配置完成。

## 步骤5-配置Ceph OSD节点,构建Ceph集群

----------------------

1.在ceph-admin节点的所有节点上安装Ceph。

登录到ceph-admin节点。

>*su - cephuser*

### 在ceph-admin节点上安装ceph-deploy

添加Ceph存储库，并使用yum命令安装Ceph部署工具' **ceph-deploy** '。

>*sudo rpm -Uhv
http://download.ceph.com/rpm-jewel/el7/noarch?ceph-release-1-1.el7.noarch.rpm*  
>*sudo yum update -y && sudo yum install ceph-deploy -y*

确保所有节点都已更新。

安装ceph-deploy工具后，为ceph集群配置创建一个新目录。

![](media/25d176d77ee5148333af4773c73dcb61.png)

### 创建新的集群配置

1.创建新的群集目录。

>*mkdir cluster*  
>*cd cluster/*

![](media/948213a040615176aa5d02ac28552bdf.png)

2.使用“ **ceph -deploy** ”命令创建一个新的集群配置，将监视节点定义为“ ceph-0 ”。

>*ceph-deploy new* ceph-0

![](media/12827e68ace56f1410ca4e1d43bbb5ac.png)

![](media/0ff6fad3a30bc421afb721a440904369.png)

3.该命令将在集群目录中生成Ceph集群配置文件'ceph.conf'。

用vim编辑ceph.conf文件。

>*vim ceph.conf*

4.在[global]块下，在下面粘贴配置。

>*# Your network address*  
>*public network = 192.168.42.143/24*  
>*osd pool default size = 2*

![](media/dabb6e39e1e4355efd79fcea344a7b0d.png)

在ceph-0中输入ip addr 查看

![](media/488679e814f3961e1d64710ff426a1f4.png)

保存文件并退出vim。
在所有节点上安装Ceph

5.从ceph-admin节点在所有其他节点上安装Ceph。这可以通过单个命令完成。

>*ceph-deploy install ceph-admin ceph-0 ceph-1 ceph-2*

该命令将在所有节点上自动安装Ceph：*ceph-0 ceph-1
ceph-2*和ceph-admin-安装将花费一些时间。

![](media/2f85909174edb37feb4290748bfa50a6.png)

![](media/67582b0ba875f0045e6d1afcb441b7c5.png)

![](media/11bf696274896167c9f4b4b45d8b4ebc.png)

![](media/54de22d9517b3d491ffa030ec4b459ec.png)

6.初始化mon1节点,将ceph-mon部署在*ceph-0* 节点上。

*ceph-deploy mon create-initial*

![](media/223b98146ba666b642e5be14be5eb5cc.png)

![](media/91d6025c1bf36fbffa77ef5666851d91.png)

该命令将创建监视键，并使用“ ceph”命令检查并获取键。

*ceph-deploy gatherkeys ceph-0*

![](media/ad49ee047867ffb15ee1f512b729f10e.png)

![](media/c78d42b92a4ff942503aefcd87c3c76c.png)

### 将OSDS添加到群集

为osd守护进程创建目录

osd1节点：

>*ssh ceph-1*

>*sudo mkdir /var/local/osd1*

>*sudo chown ceph: /var/local/osd1*

>*exit*

![](media/664321d16925b5268056f3ef1d51353a.png)

![](media/e92b1f33ebbd11108bd5c0c2aae77aca.png)

osd2节点：

>*ssh ceph-2*

>*sudo mkdir /var/local/osd2*

>*sudo chown ceph: /var/local/osd2*

>*exit*

![](media/f031000d1a46e466729410e9b62f5b9c.png)

![](media/5fc27d87925b7373892f30bee5d09ee6.png)


### 将OSDS添加到集群

现在准备所有OSD节点上的ceph-1、ceph-2节点。

>ceph-deploy osd prepare ceph-1: /var/local/osd1 ceph-2:/ /var/local/osd2

![](media/44a57fcd8894eeadfb842fa760228cfe.png)

使用以下命令激活OSD：

>ceph-deploy osd activate ceph-1: /var/local/osd1 ceph-2:/ /var/local/osd2

![](media/c1d8209262a368d8c0421a8d750ec6ef.png)

将管理密钥部署到所有关联的节点。

>*ceph-deploy admin ceph-admin ceph-0 ceph-1 ceph-2*

![](media/26b7153fffb7d4dfa9626bd42b559c3b.png)

![](media/cc2c473e5c67ade46b284d08cdbaced4.png)

![](media/a682ebf6626c65a1cf0d0a9408921ea1.png)

![](media/cc3e6892ef4e77cea21d890ba96a044b.png)

通过在所有节点上运行以下命令来更改密钥文件的权限。

>*sudo chmod 644 /etc/ceph/ceph.client.admin.keyring*

![](media/0a4642aa892c87e33e9f5e301d6ad9d5.png)

在CentOS 7上的Ceph集群已创建。

## 第6步-测试Ceph设置
.........

在第4步中，我们安装并创建了新的Ceph集群，然后将OSDS节点添加到了集群中。现在我们可以测试集群，并确保集群设置中没有错误。

从**ceph** -admin节点登录到**ceph**监视服务器“ **ceph-0** ”。

>*ssh ceph-0*

运行以下命令以检查集群运行状况。

>*sudo ceph health*

![](media/03925bd6823bf0042df5384fc9c78121.png)

现在检查集群状态。

>*sudo ceph -s*

![](media/83a7bbb1e63ff23dd6b27e80730f94bc.png)

成功构建了一个新的Ceph集群。