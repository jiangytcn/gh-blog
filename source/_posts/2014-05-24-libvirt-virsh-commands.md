---
layout: post
title: "Libvirt Virsh Commands"
description: ""
category: "Libvirt"
tags: [libvirt,Linux,Virsh]
---

##Libvirt Virsh 命令

###磁盘文件
- 查看domain的挂载的磁盘信息
	- virsh domblklist instance-0000002d
		<pre><code>vda        /opt/stack/data/nova/instances/f8780dbc-e2fd-46e9-ae0c-9a654d4eb95a/disk
		vdb        /opt/stack/data/nova/instances/f8780dbc-e2fd-46e9-ae0c-9a654d4eb95a/disk.local
		vdc        /opt/stack/data/nova/instances/f8780dbc-e2fd-46e9-ae0c-9a654d4eb95a/disk.swap
		vdd        /dev/disk/by-path/ip-192.168.0.28:3260-iscsi-iqn.2010-10.org.openstack:volume-7c9646ef-e1f1-4da4-b230-c4ec7679bb6a-lun-1
		</code></pre>
- 查看磁盘文件状态
	- virsh domblkstat instance-0000002d vda
		<pre><code>vda rd_req 7970
		vda rd_bytes 214534656
		vda wr_req 1503
		vda wr_bytes 247596032
		vda flush_operations 182
		vda rd_total_times 32382476906
		vda wr_total_times 646733610926
		vda flush_total_times 214371604
		</code></pre>
