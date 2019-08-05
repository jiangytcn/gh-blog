---
title: Add route for VirtualBOX hostonly ifs
date: 2019-08-05 19:54:12
tags:
- VM
---


最近在使用Mac上的VirtualBox 创建出的虚拟机做K8S相关的开发工作， 物理机有时重启后无法连接到虚拟机当中，ICMP拒绝， 但是在VM 内部以及VM间网络通信都是正常的。重启Mac后问题可以解决，但是不是解决问题之道，经过排查，发现Mac上的到虚拟机hostonly 网络的路由丢失，导致连接失败

<!-- more -->

查看当前物理机上的hostonly 网卡信息
```
$ VBoxManage list hostonlyifs
Name: vboxnet0
GUID: 786f6276-656e-4074-8000-0a0027000000
DHCP: Disabled
IPAddress: 192.168.50.1
NetworkMask: 255.255.255.0
IPV6Address:
IPV6NetworkMaskPrefixLength: 0
HardwareAddress: 0a:00:27:00:00:00
MediumType: Ethernet
Wireless: No
Status: Up
VBoxNetworkName: HostInterfaceNetworking-vboxnet0

Name: vboxnet1
GUID: 786f6276-656e-4174-8000-0a0027000001
DHCP: Disabled
IPAddress: 192.168.59.1
NetworkMask: 255.255.255.0
IPV6Address:
IPV6NetworkMaskPrefixLength: 0
HardwareAddress: 0a:00:27:00:00:01
MediumType: Ethernet
Wireless: No
Status: Down
VBoxNetworkName: HostInterfaceNetworking-vboxnet1

Name: vboxnet2
GUID: 786f6276-656e-4274-8000-0a0027000002
DHCP: Disabled
IPAddress: 192.168.99.1
NetworkMask: 255.255.255.0
IPV6Address:
IPV6NetworkMaskPrefixLength: 0
HardwareAddress: 0a:00:27:00:00:02
MediumType: Ethernet
Wireless: No
Status: Down
VBoxNetworkName: HostInterfaceNetworking-vboxnet2
```
Add dhcp server
`VBoxManage dhcpserver modify --ifname vboxnet0 --ip 192.168.50.2 --netmask 255.255.255.0 --lowerip 192.168.50.100 --upperip 192.168.50.199 --enable`

查看虚拟机网络网关的路由信息
```
$ route get 192.168.50.1
route to: 192.168.50.1
destination: default
mask: default
gateway: 192.168.1.1
interface: en0
flags: <UP,GATEWAY,DONE,STATIC,PRCLONING>
recvpipe sendpipe ssthresh rtt,msec rttvar hopcount mtu expire
0 0 0 0 0 0 1500 0
```
可以看到此网络的网关走到了192.168.1.1 此地址为Mac机器的网关，所以到该虚拟网络的流量都会通过gw 出去，因此也就到达不了虚拟机内部

添加到虚拟网络地址段的路由

`$ sudo route -nv add -net 192.168.50 -interface vboxnet0`

```
u: inet 192.168.50.0; u: link vboxnet0:a.0.27.0.0.0; u: inet 255.255.255.0; RTM_ADD: Add Route: len 140, pid: 0, seq 1, errno 0, flags:<UP,STATIC>
locks: inits:
sockaddrs: <DST,GATEWAY,NETMASK>
192.168.50.0 vboxnet0:a.0.27.0.0.0 255.255.255.0
add net 192.168.50: gateway vboxnet0
```

$ route get 192.168.50.114
route to: 192.168.50.114
destination: 192.168.50.0
mask: 255.255.255.0
interface: vboxnet0
flags: <UP,DONE,CLONING,STATIC,PRCLONING>
recvpipe sendpipe ssthresh rtt,msec rttvar hopcount mtu expire
0 0 0 0 0 0 1500 -438
此时可以看到，到该虚拟网络的地址都通过vboxnet0

$ ping 192.168.50.115
PING 192.168.50.115 (192.168.50.115): 56 data bytes
64 bytes from 192.168.50.115: icmp_seq=0 ttl=64 time=0.287 ms
^C
--- 192.168.50.115 ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.287/0.287/0.287/0.000 ms

