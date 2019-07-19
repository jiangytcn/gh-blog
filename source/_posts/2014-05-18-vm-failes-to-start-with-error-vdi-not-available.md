---
layout: post
title: "VM Failes to start with error: VDI not available"
description: ""
category: "Xenserver"
tags: [Cloudstack,Xenserver,VDI]
---

触发条件: Ssh 至 VM内部，执行关机命令(shutdown -h now), 在NFS backend下的Vm通过Cloudstack无法启动

产生原因：Xenserver 与存储设备或者Lun失去连接

解决方法：

- Extract VDI UUID of Virtual Machine(VM) that is failing to start from /var/log/cloud/management/management-server.log file
    For example, VDI UUID will be 6f97582c-xxxx-xxxx-xxxx-9aa5686bcbd36.
    VM are failing to start with "errorInfo: [SR_BACKEND_FAILURE_46, The VDI is not available [opterr=VDI6f97582c-xxxx-xxxx-xxxx-9aa5686bcbd36 already attached RW]" error

- Connect to XenServer and make a note of the SR UUID and name-label of the VDI

<pre><code># xe vdi-list uuid=<VDI uuid from step 1> params=sr-uuid, name-label</code></pre>

- Run the following commands in XenServer

<pre><code># xe vdi-forget uuid=<volume extracted from step 1>
# xe sr-scan uuid=<SR uuid extracted from step 2>
# xe vdi-param-set uuid=< volume extracted from step1 > name-label=< name label extracted from step2 >
</code></pre>

- Restart Cloudstack Management Service
