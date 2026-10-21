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

### 3.2 修改节点的计算机构名

hadoop要求：计算机构名必须与域名一致，我们使用本地域名。

两个方法：

1) 编辑法

命令：

```sh
sudo vim /etc/hostname
```

将文件中的内容改成需要的计算机构名。

2) 文件内容覆盖法

命令：

```sh
sudo echo 计算机名 >/etc/hostname
```

修改后重启生效。

### 3.3 所有节点建立相同的用户名

### 3.4 所有节点关闭防火墙

#### 3.4.1 master

```sh
sudo systemctl status ufw
```

防火墙已经关闭，并且disable。

#### 3.4.2 slave0

```sh
sudo systemctl status ufw
```

当前active，并且enable，所以需要stop，然后disable。

#### 3.4.3 slave1

```sh
sudo systemctl status ufw
```

当前active，并且enable，所以需要stop，然后disable。

### 3.5 配置固定ip地址

当前路由配置：

### 3.6 配置本地域名

### 3.7 配置时钟同步

### 3.8 安装JDK

### 3.9 配置环境变量