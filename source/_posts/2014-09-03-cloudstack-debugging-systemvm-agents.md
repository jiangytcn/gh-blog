---
title: "Cloudstack Debugging SystemVm agents"
description: ""
category: "cloudstack"
tags: [cloudstack]
---

##Cloudstack Debug a live agent
login into ssvm, either console proxy or ssh(port 3922)
kill all the processes named as(run.sh/_run.sh, and java)

1. cd /usr/local/cloud/systemvm
2. add parameters "-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8787" after "java " in the last line of _run.sh
3. ./run.sh, the java agent will start, with debug port 8787 is listened on.
4. allow port 8787 in ssvm, "iptables -I INPUT -i eth1 -p tcp -m state --state NEW -m tcp --dport 8787 -j ACCEPT", either eth1 or eth2 is ok.

<!-- more -->

5. Log in into ssvm - Log into the hypervisor and then type the following command   "ssh -i /opt/xensource/bin/id_rsa --p 3922 root@privateIP_or_LinkLocalIpofSSVM", or "ssh -i /root/.ssh/id_rsa.cloud -p 3922 root@LinkLocal" on  XenServer.  Private ip in case of vmware and linklocal in case Xenserver.  Vmware do not have link local ip. SSVM can be accessed from Management server using private ip address .   
 Ex:  ssh -i  /var/cloudstack/management/.ssh/id_rsa  -p 3922 root@<Private Ip address of SSVM>

6. Log in into ssvm - Log into the hypervisor and then type the following command   "ssh -i /opt/xensource/bin/id_rsa --p 3922 root@privateIP_or_LinkLocalIpofSSVM", or "ssh -i /root/.ssh/id_rsa.cloud -p 3922 root@LinkLocal" on  XenServer.  Private ip in case of vmware and linklocal in case Xenserver.  Vmware do not have link local ip. SSVM can be accessed from Management server using private ip address .   
 Ex:  ssh -i  /var/cloudstack/management/.ssh/id_rsa  -p 3922 root@<Private Ip address of SSVM>

7. SSVM health check - Run the following script inside ssvm  /usr/local/cloud/systemvm/ssvm-check.sh

	It checks for 
	
		1. connectivity with  DNS server 
		2. resolving of  domain names 
		3. status of secondary storage 
		4. ability to write to secondary storage 
		5. connectivity with management server at port 8250 and  status of java process.
		NOTE - If you dont find the health check script then go to #9. Most probably your ssvm didnt get patched right.

8. Template not ready / not available when creating an instance - Many a times the SSVM is running but still the templates do not show as ready or to say templates are not available when creating an instance. Run the health check script above and diagnose. The most probable reason reason is that the agent running on SSVM hasn't been able to connect with MS which could also be validated by checking the host table in DB. select * from host where type like 'SecondaryStorageVM'. If the status shows as Alert then definitely that is the reason. There could be a number of reasons for the agent not being able to connect with MS. Below three could be one of them.

Check whether port 8250 is open on MS and there is no firewall rule. This is the port on which the agent and MS communication happens.

Check whether the SSVM is trying to connect to the right ip of MS. If it is incorrect it could be due to the wrong ip being set in the global settings (configuration table) for 'host' in MS. Change that, restart MS and SSVM and see if it solves the issue.

Check the agent status on SSVM- See if the agent is running by typing "service cloud status" in SSVM. Try to run it and see if that's successful or changes the alert status.

To check the state of templates whether is has downloaded or there is an error - Log into DB and check table template_host_ref and observe the download_state and error_string.

Templates stuck in download in progress - Either stop and then start the SSVM. Or, run service cloud restart on the SSVM. You can also restart MS. This would trigger template sync which essentially will try and resume such stuck templates or redo the download of erred out templates. Whenever there is a handshake between MS and the agent running on the SSVM there is a template sync which syncs the template status on the MS DB and the template's physical Location and triggers the download if its not complete or retries in case there was some error. (This handshake happens on ssvm re/start, agent re/start and MS re/start)
Connection refused as the status for the template - Check whether the config parameter "secstorage.allowed.internal.sites" has been set to allow the internal n/w URL's.

Retrying the download of templates - Refer #5 above

###no route to host
This error often implies your firewall blocks the traffic, check iptable rules in SSVM then host then physical firewall.
###SSVM agent not running
You see error something like below. Most probably your systemvm didnt get patched with the agent specific code. This patching happens through the ISO - something like  systemvm***.iso. Check the size and location of the iso and whether you are not using the old version iso. For XS its on the host so grep for it and for vmware its on secondary storage. Do also check that if its vmware that you built your ssvm using noredist flag for mvn command.
2013-12-20 13:35:12,954 DEBUG [cloud.utils.ProcessUtil] (main:null) Execution is successful.
2013-12-20 13:35:12,960 ERROR [cloud.agent.AgentShell] (main:null) Unable to start agent: Resource class not found: com.cloud.storage.resource.PremiumSecondaryStorageResource due to: java.lang.ClassNotFoundException: com.cloud.storage.resource.PremiumSecondaryStorageResource
Dont see any health check script on SSVM or the agent running - The issue for sure is as in #9 above. For vmware setup I saw that because the systemvm.iso size wasnt right.
SSVM Logs - /var/log/cloud/cloud.log
*ERROR: Java process not running. Try restarting the SSVM - *It looks like the scripts/java binaries have not been copied into the SSVM ? See this thread http://markmail.org/thread/niu2qwsdydkuh2f for how it is supposed to work (response from Alex)
###Available secondary storage space is low 
shows only ~2 gb. Generally this is because the secondary storage is not mounted correctly. Run the step 2 above to verify that. One of the reasons could be "It's due to IP access-list settings which was turned on by default on NAS4Free.  Have allowed the SSVM's IP address to mount to the NAS and it's now working fine."
SSVM nics - SSVM basically has four nics, they are:


- eth0: link local nic used for ssh login from host
- eth1: private nic used as management interface between mgmt server and SSVM
- eth2: public nic used as interface that can reach outside internet
- eth3: storage nic used as interface to access secondary storage share like NFS

CloudStack sets route for each nic, however, the most important route 'default' is set to public nic which is eth2.

That means a healthy SSVM should have default route like(by command 'ip route'):
default via public_gateway_ip_address dev eth2

this also implies communication between SSVMs happen thru public nic even both SSVMs are in the same private subnet.

SSVM templates physical location - find the mount point by typing command "mount" . Go to the directory and under template/tmpl you will find all the templates.

SSVM Apache server - For 2.2 onwards the system vms are debian based. Type "service apache2 status" to find the status. Apache root is at /www/html/
Run script of java process /usr/local/cloud/systemvm/run.sh
Increasing log level - 1) Edit the file /usr/local/cloud/systemvm/conf/log4j-cloud.xml 2) For the log file cloud.log change the threshold to info:  <param name="Threshold" value="WARN"/>  to  <param name="Threshold" value="INFO"/>  3) Change com.cloud to INFO:  <category name="com.cloud"> <priority value="INFO"/> </category>  If you're not getting sufficient logging, you can also try setting it to  DEBUG.

Multiple secondary storages feature has been added since some time now. The private templates are copied to one of the secondary storages and public to all of them for higher availability.All the public templates will be replicated to all the secondary storage for redundancy but private templates and uploaded volumes would be kept in one of the randomly chosen secondary storage. Snapshots are randomly copied to one of the secondary storage (the chain of incremental snapshots are kept on the same secondary storage.) Currently these algorithms are hardcoded and there is no way to tweak it. In case you need flexibility please open an enhancement for the same. During template sync the consistency is again checked and download triggered for templates keeping in mind the algorithm above.
SSVM and CPVM do not have agents running AND  SSVM health check runs fine. cloud.log on SSVM/CPVM keeps showing the following message: 
2014-06-16 08:08:01,108 INFO  [utils.nio.NioClient] (Agent-Selector:null) Connecting to 10.102.192.247:8250
2014-06-16 08:08:01,872 ERROR [utils.nio.NioConnection] (Agent-Selector:null) Unable to initialize the threads.
java.io.IOException: SSL: Fail to init SSL! java.io.IOException: Connection closed with -1 on reading size.
at com.cloud.utils.nio.NioClient.init(NioClient.java:84)
at com.cloud.utils.nio.NioConnection.run(NioConnection.java:108)
at java.lang.Thread.run(Thread.java:701)

###Solution :
 
1. rm ./client/target/cloud-client-ui-4.3.0.0/WEB-INF/classes/cloudmanagementserver.keystore ./client/target/conf/cloudmanagementserver.keystore ./client/target/generated-webapp/WEB-INF/classes/cloudmanagementserver.keystore
 
2. remove root entry from cloud.keystore;
 
3. remove ssl.keystore from cloud.configuration where description like '%key%';
 
4. restart MS  agent in ssvm

Download Complete 100% but getting error like this Failed post download script: /usr/sbin/vhd-utilvhd tool check /mnt/SecStorage/33e2e9f5/template/tmpl/345/447/dnld1469110483936142751tmp_ failed - Many reasons for this but amongst them are wrong OS selection, vhd corruption.

Test this in the lab by copying the template to one of the hosts then on that host run

 - vhd-util check -n filename.vhd
 - vhd-util scan filename.vhd

SSVM RAM - Set the param secstorage.vm.ram.size to in change the ram size of the vm. Default in the code is 256.


For each secondary storage there is a corresponding row created in the host table. 

- HTTP Server returned 403 (expected 200 OK) - For copy templates. 

	Try to see the first log for this template initiation ? It should be logged with DownloadCommand and should have the url of the source ssvm's template. Then you can try going to the destination SSVM and try downloading that url. 
	See what issues you get. I would also check the iptable rules to see if the destination ssvm is blocked from accessing the source ssvm and also if there is any .htaccess file in the apache directories forbidding the download of template
	
	One of the problems as was as follows.*The problem is that we're using basic networking & have the private network setup with the same gateway & subnet as the public network.  When the storage VM comes up the public network gets setup first but then when the private network comes up on eth2 it clobbers the gateway & sets it to the eth2 interface.  So when the copy is initiated between the storage VMs it happens across the private network but the /var/www/html/copy/.htaccess file only allows the public IP of the other SSVM, thus the 403 errors. 
