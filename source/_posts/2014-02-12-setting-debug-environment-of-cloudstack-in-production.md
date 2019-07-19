---
title: "Setting Debug Environment Of Cloudstack In Production"
description: ""
category: "Cloudstack"
tags: [Cloudstack,Debug]
---

This introduction will outline specifically how setting debug environment in production of Cloudstack, and how building a single project into a patch.

### Configure Tomcat
<pre><code>$ vim /usr/sbin/tomcat6</code></pre>
- **Adding -server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8787 into the file above**
<pre><code> 35     -Djava.io.tmpdir="$CATALINA_TMPDIR" \
 36     -Djava.util.logging.config.file="${CATALINA_BASE}/conf/logging.properties" \
 37     -Djava.util.logging.manager="org.apache.juli.ClassLoaderLogManager" \
 38     -server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8787 \
 </code></pre>

 Now configure you eclipse to the server, go Debugging.

### Building Single Project
- **Some reasons you modified a project, for instance cloud-server, and you want your new code to be taken into effect, you should build a new jar file, and then applying the patch.**
<pre><code>cd ~/cloudstack4.1.0
mvn clean
mvn -pl :cloud-server
</code></pre>

After a while when you saw something like below, then you got a new patch.

<pre><code> ------------------------------------------------------------------------
 BUILD SUCCESS
 ------------------------------------------------------------------------
 Total time: 3:01.915s
 Finished at: Wed Feb 12 14:56:24 CST 2014
 Final Memory: 26M/233M
</code></pre>

And replace the old cloud-server-4.1.0.jar with new one , restart cloudstack-management, go hacking.
