---
title: pFsense tcp offloading in kvm virtio
date: 2019-11-06 14:47:36
tags: [KVM,Network]
---

如果在KVM中创建 pFsense 实例用于 OpenStack 虚拟机的三层网关，在虚拟机之间的私有网络或者虚拟机对外网络会出现问题。例如虚拟机的DNS 解析（UDP Port 53）不正常，内部可以ping通dns 服务器，也可以ping通目标机器，但就是DNS解析无法工作。

<!-- more -->

在排除了是虚拟机的Egress规则以及pFsense中的防火墙问题外，那么可能就是由于虚拟网卡（VirtIO）以及物理网卡的问题。
通过勾选pFsense中的'Disable hardware checksum offload'和'Disable hardware TCP segmentation offload' 可以解决问题。
TCP Offloading 的功能可能在ssl termination 中会有用处，但我们只用到了pFsense 的路由及防火墙功能，应用上面的负载均衡，漏洞扫描等尚未涉及到，所以关闭该功能对虚拟机来说没有影响。

![pfsense tco](/images/pfsense_tcpoffloading.png)

Ref:

1. https://docs.netgate.com/pfsense/en/latest/hardware/troubleshooting-lost-traffic-or-disappearing-packets.html
2. https://forum.netgate.com/topic/82332/new-sg2440-disable-hardware-tcp-segmentation-offload
