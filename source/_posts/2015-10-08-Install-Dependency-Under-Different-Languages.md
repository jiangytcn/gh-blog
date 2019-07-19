---
layout: post
category : Developting
tagline: ""
tags : [Installation,Proxy]
---

This introduction will outline specifically  how to install dependencies behind proxy under different code environment

## Overview
The following commands run on redhat 6.5

1)      python pip
<pre><code># pip install --proxy PROXY_INFOR XXX</code></pre>
2)      Maven
<pre><code># vi $M2_HOME/conf/settings.xml</code></pre>
  Navigate to the proxies settings, and change to your proxy informations

 3) Gradlew
 
>set system properties within your gradlew script by using one the environment variables DEFAULT_JVM_OPTS, JAVA_OPTS or GRADLE_OPTS. Changing to specific information.

Thank you.