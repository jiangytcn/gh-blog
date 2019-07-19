---
layout: post
title: "Linux Iscsi Server Client Settings"
description: ""
category: "Linux"
tags: []
---
#服务器端

###创建ISCSI设备
- tgtadm --lld iscsi --op new --mode target --tid 2 -T iqn.cloud.test:storage.disk2
- tgtadm --lld iscsi --op new --mode logicalunit --tid 2 --lun 2 -b /dev/sdg #添加设备
- tgtadm --lld iscsi --op bind --mode target --tid 2 -I ALL # 赋权

#客户端

1. iscsiadm -m discoverydb [ -hV ] [ -d debug_level ] [-P printlevel] [ -t type -p ip:port -I ifaceN ... [ -Dl ] ] | [ [ -p ip:port -t type] [ -o operation ] [ -n name ] [ -v value ] [ -lD ] ] 
2. 客户端查看Iscsi设备
	-  iscsiadm -m discovery [ -hV ] [ -d debug_level ] [-P printlevel] [ -t type -p ip:port -I ifaceN ... [ -l ] ] | [ [ -p ip:port ] [ -l | -D ] ]
	- `[root@devstack:keystone_admin images]# iscsiadm -m discovery -t sendtargets -p 127.0.0.1`
    - `127.0.0.1:3260,1 iqn.2010-10.org.openstack:volume-7c9646ef-e1f1-4da4-b230-c4ec7679bb6a`
3. 执行命令
	- iscsiadm -m node [ -hV ] [ -d debug_level ] [ -P printlevel ] [ -L all,manual,automatic ] [ -U all,manual,automatic ] [ -S ] [ [ -T targetname -p ip:port -I ifaceN ] [ -l | -u | -R | -s] ] [ [ -o  operation  ] [ -n name ] [ -v value ] ]
	- ####查看设备
		- `[root@devstack nova]# iscsiadm -m node -T iqn.2010-10.org.openstack:volume-7c9646ef-e1f1-4da4-b230-c4ec7679bb6a --login`
		- `Logging in to [iface: default, target: iqn.2010-10.org.openstack:volume-7c9646ef-e1f1-4da4-b230-c4ec7679bb6a, portal: 192.168.0.28,3260] (multiple)`
	    `Logging in to [iface: default, target: iqn.2010-10.org.openstack:volume-7c9646ef-e1f1-4da4-b230-c4ec7679bb6a, portal: 127.0.0.1,3260] (multiple)`
	  	`Login to [iface: default, target: iqn.2010-10.org.openstack:volume-7c9646ef-e1f1-4da4-b230-c4ec7679bb6a, portal: 192.168.0.28,3260] successful.`
	    `Login to [iface: default, target: iqn.2010-10.org.openstack:volume-7c9646ef-e1f1-4da4-b230-c4ec7679bb6a, portal: 127.0.0.1,3260] successful.`
		- `fdisk -l`
4. iscsiadm -m session [ -hV ] [ -d debug_level ] [ -P  printlevel] [ -r sessionid | sysfsdir [ -R | -u | -s ] [ -o operation ] [ -n name ] [ -v value ] ]
5. iscsiadm -m iface [ -hV ] [ -d debug_level ] [ -P printlevel ] [ -I ifacename | -H hostno|MAC ] [ [ -o  operation  ] [ -n name ] [ -v value ] ] [ -C ping [ -a ip ] [ -b packetsize ] [ -c count ] [ -i interval ] ]
6. iscsiadm -m fw [ -l ]
7. iscsiadm -m host [ -P printlevel ] [ -H hostno|MAC ] [ [ -C chap [ -o operation ] [ -v chap_tbl_idx ] ] | [ -C flashnode [ -o operation ] [ -A portal_type ] [ -x flashnode_idx ] [ -n name ] [ -v value ] ] ]
iscsiadm -k priority

#使用中遇到的问题

1. ###iscsiadm: Could not perform SendTargets discovery

	<pre><code>iscsiadm -m discovery -t sendtargets -p 192.168.127.2
    iscsiadm: Could not scan /sys/class/iscsi_transport.
    iscsiadm: Could not scan /sys/class/iscsi_transport.
    iscsiadm: can not connect to iSCSI daemon (111)!
    iscsiadm: Cannot perform discovery. Initiatorname required.
    iscsiadm: Discovery process to 192.168.127.2:3260 failed to create a discovery session.
    iscsiadm: Could not perform SendTargets discovery.
	</code></pre>
	*是因为没有安装iCSCI daemon*
	
	- ubuntu下安装 open-iscsi（apt-get install open-iscsi）
	- Linux（Centos 6.4）下安装iscsi-initiator-utils-6.2.0.873-10.el6.x86_64
