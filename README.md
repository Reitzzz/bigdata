# bigdata

大数据课程（Hadoop）的实验记录仓库。一次上机（一节课）建一个目录，把这次课的操作过程、命令和结果截图记在里面。

## 目录

- [9.30-10.14](9.30-10.14/hadoop-local模式.md) — 建立并验证 hadoop 的 local 模式：关防火墙、装 sshd、装 jdk、装 hadoop 并配置环境变量、用官方示例程序测试

## 实验环境

- 本地虚拟机：VMware 上的 Ubuntu 24.04，主机名 `reitzzz-VMware-Virtual-Platform`，用户 `reitzzz`
- 集群 master 节点：主机名 `master`，用户 `bigdata3`，软件放在 `~/software`
- jdk：`jdk-17.0.12`（官方压缩包解压，不用 apt 装的）
- hadoop：`hadoop-3.5.0`（官方压缩包解压）

## 笔记约定

- 一次课一个目录，目录名用这次课的日期；跨天的课写成 `9.30-10.14` 这样。
- 笔记按「命令 → 实际输出 → 结果说明」组织：命令和能用文字表达的输出写成代码块，只有在必须看实际画面时才放截图，避免整篇都是图片。
- 图片使用有含义的文件名，如 `ufw-status-enabled.png`、`java-version.png`，不用 `image-1.png` 这类序号名。
