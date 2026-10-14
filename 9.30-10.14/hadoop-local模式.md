# 建立并验证 hadoop 的 local 模式

本次上机（9.30 – 10.14）的内容：装好 jdk 和 hadoop，配好环境变量，验证 `java`、`hadoop` 命令可用。

用到的机器有两台，注意区分：

- **本地 VMware 虚拟机**：主机名 `reitzzz-VMware-Virtual-Platform`，用户 `reitzzz`。在这里做环境准备（关防火墙、装 sshd）。
- **集群 master 节点**：主机名 `master`，用户 `bigdata3`，软件统一放在 `~/software`（= `/home/bigdata3/software`）。jdk 和 hadoop 都装在这里。

对应课程文档的章节：本节「安装 jdk」= 课程文档 1.1.3，「安装 hadoop local 模式」= 课程文档 1.3（解压 hadoop 1.3.1、配置 hadoop 1.3.2，笔记里对应 1.4.1、1.4.2）。

## 1 环境准备

### 1.1 关闭防火墙 ufw（本地 VMware 虚拟机）

Ubuntu 默认启用 ufw（Uncomplicated firewall），它会拦截 hadoop 各组件之间通信所用的端口，所以在搭建之前先把它关掉。

**1) 查看当前状态**

```sh
sudo systemctl status ufw
```

![systemctl status ufw](ufw-status-enabled.png)

输出中 `Loaded: ... enabled` 表示 ufw 服务已设为开机自启，`Active: active (exited)` 表示它当前处于活动状态（ufw 是一次性服务，加载完规则后即以 exited 状态结束，这是正常的）。`Main PID: 457 (code=exited, status=0/SUCCESS)` 说明服务正常退出。

**2) 禁用开机自启**

```sh
sudo systemctl disable ufw
```

![禁用 ufw 开机自启的输出](ufw-disable.png)

输出里的 `Synchronizing state of ufw.service with SysV service script with /usr/lib/systemd/systemd-sysv-install` 是 systemd 在同步兼容的 SysV 启动脚本，属于正常提示，不是报错。

**3) 验证**

```sh
sudo ufw status
```

![ufw status 显示不活动](ufw-status-inactive.png)

输出 `状态：不活动`，说明防火墙已经关闭。

> 注意：`systemctl disable` 取消的是 ufw 的开机自启；如果当前会话里防火墙仍是活动状态，还需要执行 `sudo ufw disable` 立即关闭。

### 1.2 安装 sshd、首次 ssh 连接（本地 VMware 虚拟机）

hadoop 的启动脚本需要 ssh 登录到各个节点去启动守护进程，因此先确认 openssh-server 可用。

**1) 安装 openssh-server**

```sh
sudo apt install openssh-server
```

![apt 安装 openssh-server](sshd-install.png)

本机此前已经安装过，apt 提示 `openssh-server 已经是最新版（1:9.6p1-3ubuntu13.19）`，无需重复安装。

**2) 首次 ssh 连接集群节点**

```sh
ssh 172.21.2.5
```

![首次 ssh 连接时的指纹确认提示](ssh-first-connect.png)

第一次连接该主机时，ssh 会提示无法确认对方身份，并给出 ED25519 密钥指纹：

```
The authenticity of host '172.21.2.5 (172.21.2.5)' can't be established.
ED25519 key fingerprint is SHA256:gKZAbI+RWM3m7Ei2NLzSJhgS9ZkzzJxNuwWkAhxw9jI.
```

确认指纹无误后输入 `yes`，该主机就会被写入 `~/.ssh/known_hosts`，之后连接不再提示。

### 1.3 安装 jdk（集群 master 节点）

课程文档里写：不能直接用 apt 推荐的方式安装，因为 **apt 安装的 jdk 内容不全，无法用于开发**；正确做法是下载完整的 jdk 包，解压后配置路径。

**1) 确认 java 情况**

虚拟机上 apt 装的 jdk 是可用的：

```sh
java -version
```

![虚拟机上 apt 装的 java 输出](java-version.png)

```
openjdk version "17.0.20.1" 2026-08-18
OpenJDK Runtime Environment (build 17.0.20.1+1-1-24.04-Ubuntu)
OpenJDK 64-Bit Server VM (build 17.0.20.1+1-1-24.04-Ubuntu, mixed mode, sharing)
```

而 master 节点上还没有 java：

```
bigdata3@master:~$ java -version
Command 'java' not found, but can be installed with:
sudo apt install openjdk-17-jre-headless  # version 17.0.19+10-1-24.04.2, or
sudo apt install openjdk-21-jre-headless  # version 21.0.11+10-1-24.04.2
sudo apt install default-jre              # version 2:1.17-75
……
```

**2) 解压 jdk 压缩包**

压缩包放在 `~/software` 下，与 hadoop 的安装包同目录：

```sh
cd ~/software
tar -zxvf jdk-17.0.12_linux-x64_bin.tar.gz
```

解压出 `jdk-17.0.12` 目录。

**3) 配置环境变量**

在 `~/.bashrc` 末尾添加两项：`JAVA_HOME` 指向 jdk 根目录，并把 `$JAVA_HOME/bin` 加进 `PATH`。

```sh
# my set
export JAVA_HOME=/home/bigdata3/software/jdk-17.0.12
export PATH=$JAVA_HOME/bin:$PATH
```

保存退出后，新打开一个终端，或者在当前终端重新装载 .bashrc（用 `source` 或 `.`）：

```sh
. ~/.bashrc
```

**4) 验证**

```sh
java -version
```

```
java version "17.0.12" 2024-07-16 LTS
Java(TM) SE Runtime Environment (build 17.0.12+8-LTS-286)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.12+8-LTS-286, mixed mode, sharing)
```

版本是 17.0.12（压缩包里的版本），说明 `PATH` 已经指向新装的 jdk，环境变量配置生效。

## 2 安装 hadoop local 模式（集群 master 节点）

hadoop 也不用 apt 装，同样是下载官方压缩包后解压使用：解压出来的目录就是后面要配的 `HADOOP_HOME`。

### 2.1 解压 hadoop

**1) 进入软件目录，确认压缩包在**

```sh
cd ~/software
ls
```

此时应该能看到 `hadoop-3.5.0.tar.gz`（`ls` 里如果已经有 `hadoop-3.5.0` 目录，说明之前解压过了）。

**2) 解压**

```sh
tar -zxvf hadoop-3.5.0.tar.gz
```

参数含义：

- `-z`：按 gzip 处理（压缩包后缀是 `.gz`）
- `-x`：解压
- `-v`：显示解压过程
- `-f`：指定后面跟的文件名

解压默认就地展开到当前目录，不需要另外指定目标路径。

**3) 验证**

```sh
ls
```

解压后会多出一个 `hadoop-3.5.0` 目录，目录名就是压缩包去掉 `.tar.gz`：

```
hadoop-3.5.0  hadoop-3.5.0.tar.gz  jdk-17.0.12  jdk-17.0.12_linux-x64_bin.tar.gz
```

看到 `hadoop-3.5.0` 目录即为解压成功。该目录的绝对路径 `/home/bigdata3/software/hadoop-3.5.0` 就是下一步 `HADOOP_HOME` 要填的值。

### 2.2 配置 hadoop

在 `~/.bashrc` 中配置两项：

- 新建一个环境变量 `HADOOP_HOME`
- 在 `PATH` 中添加 `hadoop/bin` 和 `hadoop/sbin`

```sh
export HADOOP_HOME=/home/bigdata3/software/hadoop-3.5.0
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

加上前面 jdk 的配置，`~/.bashrc` 末尾这一段是：

```sh
# my set
export JAVA_HOME=/home/bigdata3/software/jdk-17.0.12
export PATH=$JAVA_HOME/bin:$PATH
export HADOOP_HOME=/home/bigdata3/software/hadoop-3.5.0
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

重新装载 .bashrc 后测试：

```sh
. ~/.bashrc
hadoop
```

```
Usage: hadoop [OPTIONS] SUBCOMMAND [SUBCOMMAND OPTIONS]
 or    hadoop [OPTIONS] CLASSNAME [CLASSNAME OPTIONS]
  where CLASSNAME is a user-provided Java class
```

能打出 Usage 说明 `hadoop` 命令已经可用（`PATH` 生效），local 模式的安装部分完成。
