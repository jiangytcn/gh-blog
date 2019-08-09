---
title: "PowerDNS Configuration For Single Node"
description: ""
category: "sysadmin"
tags: [system]
---

Configuration For Single Host
==========================================

##### The guide is used for setup a single node powerdns with mysql backend under openstack

<!-- more -->

Installation
------------
1. Launch a powerdns instance using image ubuntu 14.04, associate floatingip
2. ssh into instance then run following commands

    sudo apt-get update
    sudo apt-get install mysql-server mysql-common pdns-server pdns-backend-mysql

    **WHEN INSTALLING MYSQL-SERVER, REMEMBER THE ROOT PASSWORD OF DATABASE FOR LATER USE**

Configuration
-------------
    
- configure mysql-server

    edit /etc/mysql/my.cnf

    `bind-address       = 127.0.0.1`

    restart mysql-server
- Create powerdns database

    Now that you have created a database, you need to add a user that will have access to that database.
    For simplify we are using root user, otherwise create use and grant access to database.

    Next, you create the database required for your install of PowerDNS:

    <pre><code>
    create database pdns;

    use pdns;

    create table domains (
     id              INT auto_increment,
     name            VARCHAR(255) NOT NULL,
     master          VARCHAR(128) DEFAULT NULL,
     last_check      INT DEFAULT NULL,
     type            VARCHAR(6) NOT NULL,
     notified_serial INT DEFAULT NULL,
     account         VARCHAR(40) DEFAULT NULL,
     primary key (id)
    ) Engine=InnoDB;

    CREATE UNIQUE INDEX name_index ON domains(name);

    CREATE TABLE records (
      id              INT auto_increment,
      domain_id       INT DEFAULT NULL,
      name            VARCHAR(255) DEFAULT NULL,
      type            VARCHAR(10) DEFAULT NULL,
      content         VARCHAR(64000) DEFAULT NULL,
      ttl             INT DEFAULT NULL,
      prio            INT DEFAULT NULL,
      change_date     INT DEFAULT NULL,
      primary key(id)
    ) Engine=InnoDB;

    CREATE INDEX nametype_index ON records(name,type);
    CREATE INDEX domain_id ON records(domain_id);

    create table supermasters (
      ip         VARCHAR(64) NOT NULL,
      nameserver VARCHAR(255) NOT NULL,
      account    VARCHAR(40) DEFAULT NULL
    ) Engine=InnoDB;</code></pre>

    Now, leave the MySQL shell with:

    `quit;`

- modify powerdns server

   >cd /etc/powerdns

   >mv pdns.conf pdns.conf.bak

   >cp pdns.d/pdns.local.gmysql.conf  pdns.conf

   then modify pdns.conf change to the db credentials created above.
   Finally, restart your PowerDNS service with the following:

   `sudo service pdns restart`

- **Admin Web UI**

    Poweradmin writen in php give you a web dashboard for operating dns record, much more convinent.

    - Install the prerequisites
    
        >$ sudo apt-get install apache2 libapache2-mod-php5 php5 php5-common 
        
        >php5-curl php5-dev php5-gd php-pear php5-imap php5-mcrypt php5-common 
        
        >php5-ming php5-mysql php5-xmlrpc gettext
        
        >$sudo pear install MDB2
        
        >$sudo pear install MDB2_Driver_mysql

    - Download poweradmin release
        
        >$cd /tmp
        
        >wget https://github.com/downloads/Poweradmin/Poweradmin/Poweradmin-2.1.6.tgz
        
        >tar xvfz Poweradmin-2.1.6.tgz
        
        >mv Poweradmin-2.1.6 /var/www/html/Poweradmin
        
        >touch /var/www/html/Poweradmin/inc/config.inc.php
        
        >chown -R www-data:www-data /var/www/html/Poweradmin/`

        Now open a browser and launch the web-based Poweradmin installer (http://<YOUR POWERDNS FLOATING IP>/poweradmin/install). Following the guide to setup admin ui.

        After installation, install subfolder under /var/www/html/Poweradmin need to be removed or migrated, then go to http://<YOUR POWERDNS FLOATING IP>/Poweradmin//index.php to setup dns records

FAQ
---
- Can not launch powerdns admin installer

  Login to the powerdns machine, disable firewall of add the tcp:80 port to the INPUT chain, restart apache2 service

- Mysql crypto releate issue

    1. Make sure php5-mcrypt installed(yum install php5-mcrypt)
    2. modify configuration for php
    
        >root@powerdns-cli-01:/etc/php5# find / -name mcrypt.so
        
        <pre><code>/usr/lib/php5/20121212/mcrypt.so</code></pre>
        
        >root@powerdns-cli-01:/etc/php5# cat  cli/conf.d/20-mcrypt.ini 
        
        <pre><code>; configuration for php MCrypt module
        extension=/usr/lib/php5/20121212/mcrypt.so`</code></pre>
    
References
----------

  1. https://doc.powerdns.com/md/authoritative/installation/
  2. Google