---
title: Docker Swarm 网络
tags: [docker, swarm]
date: 2021-07-16 11:59:16
categories: docker
---

## bridge：网桥模式

默认的网络配置，如果未指定网络配置，这是默认创建的网络类型。当应用程序需要通信的独立容器中运行时使用。详细参考：https://docs.docker.com/network/bridge/

## host：主机网络

对于独立的容器，将删除 Docker 和主机之间的网络隔离，直接使用主机网络。详细参考：https://docs.docker.com/network/host/

## overlay：叠加网络

网络将多个 Docker 守护程序连接在一起，并使集群服务能互相通信。可以使不同 Docker 守护程序上两个独立容器之间的通信。这种策略消除了在这些容器之间进行操作系统级路由的需要。详细参考：https://docs.docker.com/network/overlay/

## macvlan

网络允许将 MAC 地址分配给容器，使其在网络上显示为物理设备。Docker 守护程序通过其 MAC 地址将流量路由到容器。macvlan 在处理希望直接连接到物理网络而不是通过 Docker 主机的网络堆栈进行路由的应用程序时的最佳选择。详细参考：https://docs.docker.com/network/macvlan/

## none

对于此容器，禁用全部网络连接。详情参考：https://docs.docker.com/network/none/

## 第三方网络插件

详细参考：https://docs.docker.com/engine/extend/plugins_services/
