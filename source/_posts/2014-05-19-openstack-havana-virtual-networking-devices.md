---
title: "Openstack Havana Neutron 虚拟网络设备分析"
description: "Openstack Havana Neutron 虚拟网络设备分析"
category: "openstack"
tags: [neutron,Havana]
---

<pre><code>
Openstack网络设计中有：Tap设备、veth对，linux 桥接、OvS 桥接四中虚拟网络设备。对于一个流经vm中的eth0到物理host的eth1的以太网数据帧来说，要利用host上的9个设备完成：Tap设备vnet0(vm nic),linux 桥接qbrXXX, veth pair(qvbXXX,qvoXXX),Open vSwitch 桥接br-int, veth pair(intbr-eth1,phy-br-eth1),以及最后的物理主机的网卡eth1。

Tap设备:例如KVM、Xen虚拟一个网卡（通常称作VIF或者vNIC）vnet0,供vm使用。Guest OS因此接收到所有发送到Tap设备的以太网数据帧。

Veth pairs 是一对直接相连的虚拟网卡（virtual network interfaces），发送到veth对中的任意一方的以太网数据帧，另一方也会接收到。网络因此利用veth pairs作为VPC(virtual patch cables)来连接virtual bridges.

Linux brige 象一个hub一样，可以将多个网络设备，包括物理或者虚拟机的，连接到一个Linux 桥接设备上。以太网数据帧会在hub上相连的的所有网络设备上进行传输

Open vSwitch 桥接设备的想一个虚拟的交换机，网卡连接到OVS 桥街上的端口，端口可以象物理交换机一样，对其进行配置VLAN等信息

由于openstack现在的防火墙的实现机制，使得不能够直接将Tap设备（vnet0、vnet1……）直接接入到桥接设备br-int(like a hub).防火墙实现机制是在tap设备上通过iptables实现防火墙，但是OVS又不支持直接连接到OVS port上的tap设备上的iptables

因此，网络设计上，增加了一个Linux 桥接设备以及一个veth pair来解决这个问题。将tap设备vnet0连接到linux bridge上（qbrXXX），而不是直接到连到ovs的桥街上，这个ovs桥接设备通过（qvbXXX,qvoXXX）veth pair连接到br-int(linke a hub)上

[root@devstack:keystone_admin ~]# brctl show
bridge name     bridge id          STP enabled     interfaces
qbr69e73747-c0          8000.52a89a03f5a9     no          qvb69e73747-c0
qbrff6ee26c-fc          8000.febb4e384e96     no          qvbff6ee26c-fc
</code></pre>
