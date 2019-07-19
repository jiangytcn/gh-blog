---
layout: post
title: "Docker   Enter a Running Container With New TTY"
description: ""
category: "sysadmin"
tags: [docker]
---

Generic Method - Docker Enter
============================

Since docker 1.3, you can use docker exec -it [container-id] bash. To make this easier, I create a file at/usr/bin/docker-enter with the following contents.
<pre><code>
#!/bin/bash 

EXPECTED_NUM_ARGS=1;

if [ "$#" -ne $EXPECTED_NUM_ARGS ]; then
    CONTAINER_ID=`/usr/bin/docker ps -q --no-trunc | /bin/sed -n 1p`
    /usr/bin/docker exec -it $CONTAINER_ID bash
else
    /usr/bin/docker exec -it $1 bash
fi
</code></pre>

Now you can just run docker-enter $container-id or if you just want to enter the last container you spawned, just run the command docker-enter
