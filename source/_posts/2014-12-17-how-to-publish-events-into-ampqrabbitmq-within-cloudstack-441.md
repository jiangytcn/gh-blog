---
title: "How To Publish Events Into AMPQ/Rabbitmq Within Cloudstack 4.4.1"
description: ""
category: 
tags: [Cloudstack]
---

#### Currently within cloudstack all events are stored in DB and the only to retrive events is using APIS which is not so convient.Since 4.1.0, Apache CloudStack began to support publishing interesting events from the management servers onto a message queue. The current implementation of that feature uses RabbitMQ as the message broker.Fowllowings are the steps to enable such feature.

<!-- more -->

---

### Install Rabbitmq-server


- Following the instructions from here:http://www.rabbitmq.com/download.html to setup rabbitmq-server
- Make sure that rabbitmq-server running, and (/or) enable rabbitmq-management webui plugin

### Configure Cloudstack
Assuming you have setup cloudstack management server and have mq server running with correct user credential(Default user is ***guest/guest***)

- In cloudstack management server 
	- Create spring event bus related configuration file
	<pre><code>
	mdir -p /usr/share/cloudstack-management/webapps/client/WEB-INF/classes/META-INF/cloudstack/core
	[root@cs-mgmt core]# cat spring-event-bus-context.xml
	< beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:util="http://www.springframework.org/schema/util"	
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context-3.0.xsd
	http://www.springframework.org/schema/util
	http://www.springframework.org/schema/util/spring-util-3.0.xsd"
	>
	< bean id="eventNotificationBus" class="org.apache.cloudstack.mom.rabbitmq.RabbitMQEventBus">
	    < property name="name" value="eventNotificationBus" />
	    < property name="server" value="localhost" />
	    < property name="port" value="5672" />
	    < property name="username" value="guest" />
	    < property name="password" value="guest" />
	    < property name="exchange" value="cloudstack-events" />
	  < /bean>
	< /beans>
	</code></pre>
	- restart cloudstack management service
		- /etc/init.d/cloudstack-management restart

###Check Exchange status

Using rabbitmqctl to check the status exchanges of current node.
    
<pre><code>
[root@cs-mgmt core]# rabbitmqctl list_exchanges
Listing exchanges ...
	direct
amq.direct	direct
amq.fanout	fanout
amq.headers	headers
amq.match	headers
amq.rabbitmq.log	topic
amq.rabbitmq.trace	topic
amq.topic	topic
cloudstack-events	topic
...done.
[root@cs-mgmt core]# 
</code></pre>

We can see that cloudstack has connected to rabbitmq server.
