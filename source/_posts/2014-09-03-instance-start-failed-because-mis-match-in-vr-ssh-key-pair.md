---
layout: post
title: "Instance start failed because mismatch in VR ssh key Pair"
description: ""
category: "cloudstack"
tags: [cloudstack,trouble-shooting]

---

### Exception

Failed to authentication SSH user root on host 10.147.40.164
2013-06-15 01:36:13,977 ERROR [vmware.resource.VmwareResource] (DirectAgent-18:10.147.40.30) Unable to execute NetworkUsage command on DomR (10.147.40.164), domR may not be ready yet. failure due to Exception: java.lang.Exception
Message: Failed to authentication SSH user root on host 10.147.40.164
java.lang.Exception: Failed to authentication SSH user root on host 10.147.40.164
at com.cloud.utils.ssh.SshHelper.sshExecute(SshHelper.java:144)
at com.cloud.utils.ssh.SshHelper.sshExecute(SshHelper.java:37)
at com.cloud.hypervisor.vmware.resource.VmwareResource.networkUsage(VmwareResource.java:5451)
at com.cloud.hypervisor.vmware.resource.VmwareResource.execute(VmwareResource.java:2301)
at com.cloud.hypervisor.vmware.resource.VmwareResource.executeRequest(VmwareResource.java:480)
at com.cloud.agent.manager.DirectAgentAttache$Task.run(DirectAgentAttache.java:186)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
at java.util.concurrent.FutureTask$Sync.innerRun(FutureTask.java:334)
at java.util.concurrent.FutureTask.run(FutureTask.java:166)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$101(ScheduledThreadPoolExecutor.java:165)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:266)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1110)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:603)
at java.lang.Thread.run(Thread.java:679)
2013-06-15 01:36:13,980 DEBUG [vmware.resource.VmwareResource] (DirectAgent-18:10.147.40.30) Executing resource GetDomRVersionCmd: {"accessDetails":
{"router.ip":"10.147.40.164","router.name":"r-8-VM"}
,"wait":0}
2013-06-15 01:36:13,980 DEBUG [vmware.resource.VmwareResource] (DirectAgent-18:10.147.40.30) Run command on domR 10.147.40.164, /opt/cloud/bin/get_template_version.sh 
2013-06-15 01:36:13,981 DEBUG [vmware.resource.VmwareResource] (DirectAgent-18:10.147.40.30) Use router's private IP for SSH control. IP : 10.147.40.164
2013-06-15 01:36:14,081 DEBUG [cloud.api.ApiServlet] (catalina-exec-12:null) ===START=== 10.101.255.119 – GET command=queryAsyncJobResult&jobId=43dde7f6-fb5d-4ead-8691-5071feb44dfd&response=json&sessionkey=tN07%2BJ4GVSCHGNV%2FPjdO3V%2Bs3Tg%3D&_=1371220848020
2013-06-15 01:36:14,163 DEBUG [cloud.api.ApiServlet] (catalina-exec-12:null) ===END=== 10.101.255.119 – GET command=queryAsyncJobResult&jobId=43dde7f6-fb5d-4ead-8691-5071feb44dfd&response=json&sessionkey=tN07%2BJ4GVSCHGNV%2FPjdO3V%2Bs3Tg%3D&_=1371220848020
2013-06-15 01:36:14,231 ERROR [utils.ssh.SshHelper] (DirectAgent-18:10.147.40.30) Failed to authentication SSH user root on host 10.147.40.164
2013-06-15 01:36:14,235 ERROR [vmware.resource.VmwareResource] (DirectAgent-18:10.147.40.30) GetDomRVersionCmd failed due to Exception: java.lang.Exception
Message: Failed to authentication SSH user root on host 10.147.40.164
java.lang.Exception: Failed to authentication SSH user root on host 10.147.40.164
at com.cloud.utils.ssh.SshHelper.sshExecute(SshHelper.java:144)
at com.cloud.utils.ssh.SshHelper.sshExecute(SshHelper.java:37)
at com.cloud.hypervisor.vmware.resource.VmwareResource.execute(VmwareResource.java:2143)
at com.cloud.hypervisor.vmware.resource.VmwareResource.executeRequest(VmwareResource.java:488)
at com.cloud.agent.manager.DirectAgentAttache$Task.run(DirectAgentAttache.java:186)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
at java.util.concurrent.FutureTask$Sync.innerRun(FutureTask.java:334)
at java.util.concurrent.FutureTask.run(FutureTask.java:166)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$101(ScheduledThreadPoolExecutor.java:165)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:266)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1110)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:603)
at java.lang.Thread.run(Thread.java:679)
2013-06-15 01:36:14,237 DEBUG [agent.manager.DirectAgentAttache] (DirectAgent-18:null) Seq 1-965476380: Cancelling because one of the answers is false and it is stop on error.
2013-06-15 01:36:14,238 DEBUG [agent.manager.DirectAgentAttache] (DirectAgent-18:null) Seq 1-965476380: Response Received

##Solutions

[root@ms2 ~]# cat /var/cloudstack/management/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAuqhNj72tdEVhc/Cbph6EyPLcPm6KKrgVMdFKXRn7uW5S6UT+QqtM4ec7S0lA0roVsbnst2/DdLQZhCcBDMggRRP7BJSM1O5cXcFQib+gwAaneaUwomVMPiA/uAdCJBxgId5qTZw4cyWnemLkkZTcta96aIIhT0/ycal4PMPbXRE9u78RRVeoo19BnjAYd6o7zo+O8fNaRLdQRTcpGCm5KTQRhKREOA/OREHvX0mBuuEAie74e3QVd+ZA/6kQ7uspKsFnQTWEM0I8YR2UPj9JL1pyPAawV3QiycEOaChYhUZ6DcuebEJEbzahiRbtBnDkRz+gQ/TpHx96pmAzzDaQyQ==cloud@ms2
[root@ms2 ~]#
###Mount System ISO
[root@ms2 ~]# mkdir /tmp/is
[root@ms2 ~]# mount -o loop /var/cloudstack/mnt/VM/7566222426160.3051fff0/systemvm/systemvm-4.2.0-SNAPSHOT.iso 

root@ms2 ~]# cd /tmp/is/

[root@ms2 is]# ls
authorized_keys cloud-scripts.tgz systemvm.zip

[root@ms2 is]# cat authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAv2Ym6oJKNnSiMPg0F/RxNhCNvmiB0HJwWLbWC0GQLkB6/Q7F93X8jConFbjehQqBPXePZhEV8veGyH8AmjRDlnP42qpxvCsOnD7BGrXm+Uv+/VovzZ719wcMny0GO2spJgpJcut0YrJqr9OXRlrgbis9dweeB0SyZDKT2ja4c6V2E2Fo9D7BdqWg2S2ptKW7oNkL6846Rfa9/WHh0PO7DmJ5mIuJbazNfBOM0SH6M2FOORgBFytC2XEf0o2aYHGDKOyuwDb2YOuThfDNP3r29qhRUHdZY8Ozs9S0iF+m/L3vBvuOCk0eJMjuYJnsuMTH7KScP5GILurlBCaU9vXmvw== cloud@ms2


We can see that the private keys are not equal, so we update the system.iso with correct authorized_keys, and then replace system.iso, destroy ssvm ,cpvm along with it's base template image within hypervisor, then the new system vms will be bring up.

Now you can deploy you virtual machines