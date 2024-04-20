# KVM桥接网络

## 让虚拟机和宿主机拥有一样的网络地位

在原本的局域网中有两台主机,1 win10ip:192.168.1.100
                            一台宿主机IP:192.168.1.7

## 修改宿主机的网络连接配置文件

```bash
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-enp3s0
TYPE="Ethernet"
DEVICE="enp3s0"
ONBOOT="yes"
BRIDGE=br0
#UUID="91d145be-26a4-4a1c-be02-39d7f42339ba"(因为该文件cp而来,要注释掉)
NAME="enp3s0"
```

### DNS服务器配置

不填写打不开网页

```bash
[root@localhost ~]# vim /etc/sysconfig/network

# Created by anaconda
DNS1=192.168.1.1
DNS2=8.8.8.8
```

### 配置桥

 一桥接的方式配置网络:

```bash
 [root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-br0

TYPE="Bridge"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
PEERROUTES=yes
PEERDNS=yes
IPV4_AUTOCONF=yes
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
NAME=br0
DEVICE="br0"
ONBOOT="yes"
IPADDR=192.168.1.7
GATEWAY=192.168.1.1
NETMASK=255.255.255.0
```

### 修改完上面的两个网络重启网卡

[root@localhost ~]# systemctl restart network
如果重启失败 ,检查配置文件格式,检查有误多余网络配置文件

## 配置kvm虚拟机网络

打开KVM图形工具:
virt-manager
选择编辑>连接详情>网络接口>将br0停掉>选择虚拟网络tab,将自带的一个default网络停止并删除,然后再重新创建一个虚拟网络,创建过程如下:

网络名称:vnet0
    步骤1 启用IPv4网络地址空间定义 
    步骤2 网络:192.168.100.0/24  不启用DHCPv4、静态路由。
    步骤3 前进
    步骤4 转发到物理网络(w) 目的:物理设备 enp3s0  模式:路由的 DNS域名: vnet0
    完成
配置完成后回到网络接口界面,启动br0,再回到虚拟网络界面启动vnet0。

此时宿主机有5个网络
br0 enp3s0 lo virbr0 vnet0

此时网络连接正常ping 8.8.8.8 ok

然后创建一台虚拟机,该虚拟机网卡配置为
    网络源:指定共享设备名称
            网桥名称:br0
            设备型号:virtio
