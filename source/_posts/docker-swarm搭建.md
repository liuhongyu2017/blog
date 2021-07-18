---
title: docker swarm搭建
tags: [docker, swarm]
date: 2021-07-16 10:20:22
categories: docker
---

> 官网地址：https://docs.docker.com/engine/swarm/

## 环境信息

### 机器环境：

- IP：10.0.2.101
  - hostname：manager
  - 集群角色：swarm manager
  - 系统：Ubuntu 18.04.4
  - CPU：1
  - 内存：1G
- IP：10.0.2.102
  - hostname：node01
  - 集群角色：swarm node
  - 系统：Ubuntu 18.04.4
  - CPU：1
  - 内存：1G
- IP：10.0.2.103
  - hostname：node02
  - 集群角色：swarm node
  - 系统：Ubuntu 18.04.4
  - CPU：1
  - 内存：1G

## 准备工作

### 修改主机名
```shell
# 10.0.2.101
$ sudo hostnamectl set-hostname manager

# 10.0.2.102
$ sudo hostnamectl set-hostname node01

# 10.0.2.103
$ sudo hostnamectl set-hostname node02
```
### 配置hosts文件
```shell
# 10.0.2.101
$ sudo vim /etc/hosts
10.0.2.101 manager
10.0.2.102 node01
10.0.2.103 node02

# 10.0.2.101
$ scp /etc/hosts root@10.0.2.102:/etc/hosts
$ scp /etc/hosts root@10.0.2.103:/etc/hosts
```
### 关闭防火墙
```shell
# 10.0.2.101
$ sudo ufw disable
# 10.0.2.102
$ sudo ufw disable
# 10.0.2.103
$ sudo ufw disable
```
### 安装docker
```shell
# 全部服务器都要安装 docker
# 国内 daocloud 一键安装
$ curl -sSL https://get.daocloud.io/docker | sh
# 官方安装脚本
$ curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

## 创建 Swarm manager 和 Swarm node

### 创建 Swarm 集群
```shell
# 10.0.2.101
$ docker swarm init --advertise-addr 10.0.2.101
Swarm initialized: current node (z2n633mty5py7u9wyl423qnq0) is now a manager.

To add a worker to this swarm, run the following command:
 
    # 这就是添加节点的方式
    # 要保存初始化后 token，因为在节点加入时要使用 token 作为通讯的密钥
    docker swarm join --token SWMTKN-1-2lefzq18zohy9yr1vskutf1sfb2a590xz9d0mjj2m15zu9eprw-2938j5f50t35ycut0vbj2sx0s 10.0.2.101:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

# 这时候可以看见 manager 节点信息
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
olqo167xr383t5k6k73qtd9uw *   manager             Ready               Active              Leader              19.03.12

```

### 添加节点到 Swarm 集群
```shell
# 10.0.2.102
$ docker swarm join --token SWMTKN-1-2lefzq18zohy9yr1vskutf1sfb2a590xz9d0mjj2m15zu9eprw-2938j5f50t35ycut0vbj2sx0s 10.0.2.101:2377
# 10.0.2.103
$ docker swarm join --token SWMTKN-1-2lefzq18zohy9yr1vskutf1sfb2a590xz9d0mjj2m15zu9eprw-2938j5f50t35ycut0vbj2sx0s 10.0.2.101:2377
```

### 在 manager 上查询集群状态
```shell
# 10.0.2.101
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
olqo167xr383t5k6k73qtd9uw *   manager             Ready               Active              Leader              19.03.12
sht8ry2l5boj2vanpbwk1sanu     node01              Ready               Active                                  19.03.12
p99qm4gaybcmwcbbxo782qxqf     node02              Ready               Active                                  19.03.12
```

### 修改 docker 源
在所有节点运行
```shell
$ vim /etc/docker/daemon.json
{
    "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```
国内源：
- Docker 中国官方源：https://registry.docker-cn.com
- 网易：http://hub-mirror.c.163.com
- 中国科技大学：https://docker.mirrors.ustc.edu.cn
- 阿里云：https://<key>.mirror.aliyuncs.com（阿里云源需要单独申请）

## 在 Swarm 集群上安装 portainer

在 manager 节点上运行
```shell
# 10.0.2.101
$ curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml
$ docker stack deploy --compose-file=portainer-agent-stack.yml portainer
```
