# hadoop集群配置

## 1 基本概念

hadoop集群由几个至数千个节点构成，节点功能：

* namenode：名字节点，针对的是HDFS，保存的是目录结构（包括：目录树、目录名、文件名、目录和文件属性等）和数据块的URL和校验值，不保存具体内容。
* datanode：数据节点，针对的是HDFS，保存的是文件的内容具体。
* resourcemanager：资源管理器yarn的资源管理服务。
* nodemanager：yarn的节点管理服务。

## 2 节点分配

典型的，最少节点数是3，可以这样分配节点功能：

* master节点：namenode和resourcemanager，其他辅助功能，例如web页面和任务管理
* workers节点：先分配两个物理节点，功能是datanode和nodemanager，初步概念理解后，将扩展到3个节点。

## 3 节点准备

### 3.1 准备3个Linux节点

### 3.2 修改节点的计算机名

hadoop要求：计算机名必须与域名一致，我们使用本地域名。

两个方法：

1) 编辑法

命名：

```sh
sudo vim /etc/hostname
```

将文件中的内容改成需要的计算机名。

2) 文件内容覆盖法

命名：

```sh
sudo sh -c "echo 计算机名 >/etc/hostname"
```

修改后重启生效。

![通过 ssh 登录后在 slave 上用 echo 覆盖 /etc/hostname 并重启](3.2-hostname-echo-reboot.png)

### 3.3 所有节点建立相同的用户名

### 3.4 所有节点关闭防火墙

#### 3.4.1 master

命令：

```sh
sudo systemctl status ufw
```

![master 上 ufw 处于 inactive (dead) 且 disabled](3.4.1-master-ufw-disabled.png)

防火墙已经关闭，并且disable。

#### 3.4.2 slave0

![slave0 上 ufw 当前 active 且 enable](3.4.2-slave0-ufw-active-stop-disable.png)

当前active，并且enable，所以需要stop，然后disable。

<!-- TODO: 补一张 slave0 执行 stop 与 disable 之后的截图 -->

#### 3.4.3 slave1

![slave1 上 ufw 当前 active 且 enable，执行 stop 与 disable 后的输出](3.4.3-slave1-ufw-active-stop-disable.png)

当前active，并且enable，所以需要stop，然后disable。

<!-- TODO: 补一张 slave1 执行 stop 与 disable 之后的截图 -->

### 3.5 配置固定ip地址

当前路由配置：

子网：192.168.58.0　子网掩码：255.255.255.0　网关：192.168.58.2

DHCP地址范围：192.168.58.128~192.168.58.254

网络规划：

* master：192.168.58.10
* slave0：192.168.58.11
* slave1：192.168.58.12

改之前 master 是 DHCP 动态地址：

![master 上 ip a，ens36 拿到 192.168.58.129](3.5-master-ip-a-dynamic.png)

slave1 同样是动态地址：

![slave1 上 ip a，ens33 拿到 192.168.58.131](3.5-slave1-ip-a-dynamic.png)

#### 3.5.1 master

master是desktop版，最好使用图形界面设置：

网络设置：

![图形界面「有线」里把 IPv4 设为手动 192.168.58.10，掩码 255.255.255.0，网关 192.168.58.2，DNS 172.16.8.3](3.5.1-master-ipv4-manual.png)

测试：

<!-- TODO: 补一张改完固定 IP 后的验证截图（ip a 或 ping） -->

### 3.6 配置本地域名

### 3.7 配置时钟同步
