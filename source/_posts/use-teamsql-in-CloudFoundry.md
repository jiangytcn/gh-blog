---
title: Use teamsql in CloudFoundry
date: 2019-08-05 19:13:38
tags: CloudFoundry
---


Most of cloudfoundry based platform have postgres service in their catalog, applications can use the credentials bounded to their running environment for accessing, but sometimes the administrator need to access the database instance for whatever reason, debugging or checking specific record.

TeamPostgreSQL is a project of Webworks ([webworks.dk](webworks.dk)), which is excellent for the webui based database management, using a single browser, user could manage the postgresql database.

<!-- more -->
download release
```
/usr/bin/curl -# http://cdn.webworks.dk/download/teampostgresql_multiplatform.zip \
    -o teampostgresql_multiplatform.zip && \
unzip teampostgresql_multiplatform.zip && rm -f teampostgresql_multiplatform.zip
```
generate deploy manifest file for teampostgresql
The application would use java buildpack for compiling and hosting, and need to specify JBP_CONFIG_JAVA_MAIN otherwise the http service won't start correctly

```
---
applications:
- name: psqlgui
  memory: 1G
  buildpack: https://github.com/cloudfoundry/java-buildpack.git
  host: psqlgui
  env:
    JBP_CONFIG_JAVA_MAIN: '{arguments: "-cp WEB-INF/lib/log4j-1.2.17.jar-1.0.jar:WEB-INF/classes:WEB-INF/lib/* dbexplorer.TeamPostgreSQL $PORT"}'

```
Deploy
using the following script to deploy application, before executing, need to obtain an account of cloudfoundry

```SHELL
#!/bin/bash

WORK_DIR=`dirname "${0}"`
RUN_DIR=$TMPDIR

cf --version > /dev/null

if [[ $? -gt 0 ]];
    then
    echo "CloudFoundry cli not installed"
    exit 1
fi

cf push -f $WORK_DIR/manifest.yml -p $RUN_DIR/teampostgresql/webapp
```

After the deployment, the application could be access through webui and then configure database with credential of postgresql database instance
