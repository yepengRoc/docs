1. centos 镜像  centos-7-x86

2. 创建虚拟机：打开virtual box，点击 新建 按钮，点击 下一步，输入虚拟机的名称为hbase-standalone,选择操作系统为linux,版本为Red-hat-64bit,分配 1024内存（可适当调大），后面选项全部默认，在Virtual Dist File Location and size 中，一定选择一个目录来保存虚拟机文件，最后点击 create 按钮，开始创建虚拟机

3. 设置虚拟机网卡：选择创建好的虚拟机 ，点击 设置 ，在网络一栏中，连接方式，选择 Bridged Adapter。通过什么方式和宿主机通信

4. 安装虚拟机中的centos 7:选择创建好的虚拟机，点击 开始，选择安装介质（即本地cent os镜像文件），按照课程选择后自动安装即可

5. 安装完成后，cent os提示重启，reboot即可

6. 配置网络

   vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

- 动态分配一个ip地址

ONBOOT=yes

保存之后，进入命令行执行  service network restart 使修改的网络配置生效

ip addr 查看当前分配的ip地址

- 再设置静态ip地址

vi /etc/sysconfig/network-scripts/ifcfg-enp0s3  #编辑配置，然后添加以下内容

BOOTPROTO=static

IPADDR=192.168.31.250

NETMASK=255.255.255.0 #子网掩码

GATEWAY=192.168.31.1 #网关

保存后，service network restart 

ip addr进行查看

- 配置DNS

检查NetManager的状态：systemctl status NetworkManager.service

检查NetManager管理的网络接口：nmcli dev status

检查NetManager管理的网络连接：nmcli connection show

设置dns: nmcli con mod enp0s3 ipv4.dns "114.114.114.114 8.8.8.8"

让dns生效：nmcli con up enp0s3

- 配置hosts

vi  /etc/hosts

  #配置本机hostname 到ip的映射

- 配置SecureCRT 连接到虚拟机

- 关闭防火墙

  sytemctl stop firewalld.service

  sytemctl disable firewalld.service

  如果是虚拟机，记得关闭windows防火墙

- 配置yum(可配可不配)

  yum clean all

  yum makecache

  yum install wget

- 安装jdk

   安装jdk rpm -ivh jdk

  配置在当前用户下

  vi .bashrc

  source ./bashrc 使编辑生效

  tar -zxvf

