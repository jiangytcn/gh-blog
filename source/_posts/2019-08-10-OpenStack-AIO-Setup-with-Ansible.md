---
title: OpenStack AIO Setup with Ansible
date: 2019-08-10 08:31:38
tags:
- OpenStack
---

如果你是OpenStack 的developer的话，那么相信对DevStack的会十分的熟悉。 DevStack 可以在一台机器上快速的交付AIO（all-in-one）的OpenStack环境十分的方便。 OpenStack-Ansible 是一种通过ansible来部署OpenStack 环境的方案，具体的设计初衷、以及如果使用请参考rackspace 以及platform9开发博客

<!--- more --->

物理环境

|名称|	信息
|---| ---|
|OS Dist|	Ubuntu 16.04.2 LTS
|Disk	|1T + 2T sata 盘
|Mem	|32G
|CPU|	Intel(R) Core(TM) i7-6820HQ CPU @ 2.70GHz
```
sdb             8:16   0 931.5G  0 disk
├─sdb1          8:17   0   243M  0 part
├─sdb2          8:18   0     1K  0 part
└─sdb5          8:21   0 931.3G  0 part
  ├─dvlp-root 252:0    0    78G  0 lvm  /
  ├─dvlp-home 252:1    0 262.3G  0 lvm  /home
  └─dvlp-data 252:2    0 553.7G  0 lvm
sdc             8:32   0   1.8T  0 disk /openstack
```
>2T盘用于存储OpenStack相关的数据
>1T盘操作系统相关数据盘

### 在使用时遇到的问题在此说明
1. 安装时启动的lxc container 无法获得private ip
正常情况下在执行了ansible 相关命令后应该每一个container应该有内网及外网IP， 如下
```SHELL
→ sudo lxc-ls -f
NAME                                        STATE   AUTOSTART GROUPS            IPV4                                          IPV6
aio1_cinder_api_container-f3bdb579          RUNNING 1         onboot, openstack 10.255.255.244, 172.29.236.67, 172.29.247.153 -
aio1_cinder_scheduler_container-25e9e567    RUNNING 1         onboot, openstack 10.255.255.198, 172.29.238.236                -
aio1_designate_container-b3ca6e11           RUNNING 1         onboot, openstack 10.255.255.64, 172.29.239.241                 -
aio1_galera_container-59168dd6              RUNNING 1         onboot, openstack 10.255.255.172, 172.29.236.218                -
aio1_glance_container-e43dfd64              RUNNING 1         onboot, openstack 10.255.255.176, 172.29.237.35, 172.29.247.172 -
aio1_heat_apis_container-9008349f           RUNNING 1         onboot, openstack 10.255.255.112, 172.29.238.189                -
aio1_heat_engine_container-2d62ff94         RUNNING 1         onboot, openstack 10.255.255.25, 172.29.239.228                 -
aio1_horizon_container-2a77938a             RUNNING 1         onboot, openstack 10.255.255.74, 172.29.239.99                  -
aio1_keystone_container-3441527b            RUNNING 1         onboot, openstack 10.255.255.28, 172.29.236.169                 -
aio1_memcached_container-43bf9c7f           RUNNING 1         onboot, openstack 10.255.255.238, 172.29.239.33                 -
aio1_neutron_agents_container-31363c11      RUNNING 1         onboot, openstack 10.255.255.187, 172.29.236.66, 172.29.242.144 -
aio1_neutron_server_container-c6d33080      RUNNING 1         onboot, openstack 10.255.255.160, 172.29.236.141                -
aio1_nova_api_metadata_container-b04322cb   RUNNING 1         onboot, openstack 10.255.255.203, 172.29.237.253                -
aio1_nova_api_os_compute_container-53803a56 RUNNING 1         onboot, openstack 10.255.255.61, 172.29.237.166                 -
aio1_nova_api_placement_container-d09947c4  RUNNING 1         onboot, openstack 10.255.255.22, 172.29.236.157                 -
aio1_nova_conductor_container-e9e82699      RUNNING 1         onboot, openstack 10.255.255.120, 172.29.239.36                 -
aio1_nova_console_container-2517dd0e        RUNNING 1         onboot, openstack 10.255.255.54, 172.29.239.129                 -
aio1_nova_scheduler_container-db02b5f9      RUNNING 1         onboot, openstack 10.255.255.243, 172.29.237.239                -
aio1_rabbit_mq_container-66fd8455           RUNNING 1         onboot, openstack 10.255.255.2, 172.29.236.124                  -
aio1_repo_container-cf8b6b77                RUNNING 1         onboot, openstack 10.255.255.17, 172.29.238.136                 -
aio1_rsyslog_container-d461d2ed             RUNNING 1         onboot, openstack 10.255.255.76, 172.29.236.228                 -
aio1_swift_proxy_container-00c23b79         RUNNING 1         onboot, openstack 10.255.255.20, 172.29.236.166, 172.29.245.74  -
aio1_utility_container-9755697d             RUNNING 1         onboot, openstack 10.255.255.219, 172.29.239.126                -
ubuntu-xenial-amd64                         STOPPED 0         -                 -                                             -
```

由于之前配置了dnsmasq作为本机的dns server，而lxc 需要用dnsmasq作为所启动的container的dhcp 服务器，所以container 无法通过dhcp获得ip导致deploy失败

**解决办法**: 清空dnsmasq的配置后，重新deploy container 即可获得内部IP

2. 重启物理节点后无法识别usb设备
在OpenStack-Ansible 的使用文档中说明了如何处理在重启物理节点后，openstack 服务如何恢复的问题。在使用时，由于sdc盘通过的usb 接口连接到的server中，重启后出现无法识别存储设备的问题。
在ansible 的role 里面有security、security role 相关的设置,在安装好openstack-ansible 后会在host中使用modporb禁用了USB设备

```SHELL
→ cat /etc/modprobe.d/openstack-ansible-security-disable-usb-storage.conf
install usb-storage /bin/true
```
**解决办法**: 删除此文件后，重启物理节点，usb设备即可识别

