---
title: "Cloudstack Cli Cloudmonkey Installation"
description: ""
category: "Cloudstack"
tags: [CloudStack,Python]
---

##Cli安装使用分为四部分

1. **安装virtualenv**
2. **安装cloudmonkey**
3. **配置cloudmonkey**
4. **命令使用**

<!-- more -->

## 安装virtualenv

	1. virtualenv提供了一个安装软件时的测试环境，所安装的软件不会安装在系统目录下
	2. 安装python2.7.4 时的依赖软件， 此步骤可选， cloudmonkey 要求python版本在2.5以上即可
	3. yum -y groupinstall "Development tools" zlib-devel bzip2-devel openssl-devel ncurses-devel mysql-devel ibxml2-devel libxslt-devel  unixODBC-devel sqlite sqlite-devel



## 源码安装python2.7.4

<pre><code>wget http://www.python.org/ftp/python/2.7.4/Python-2.7.4.tar.bz2
tar xf Python-2.7.4.tar.bz2
cd Python-2.7.4
/configure --prefix=/opt/python2.7
make && make altinstall`
</code></pre>

##源码安装virtualenv

<pre><code>wget https://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.9.1.tar.gz#md5=07e09df0adfca0b2d487e39a4bf2270a
tar
-xvzf virtualenv-1.9.1.tar.gz
python virtualenv
-1.9.1/setup.py install
</code></pre>

###安装cloudmonkey

1. 建立virtualenv环境
	<pre><code>#mkidr workenv
	#virtualenv worksenv
	#source worksenv/bin/activate
	</code></pre>
2. 安装cloudmonkey两种方式任选其一

	2.1. 通过python库安装

	在系统安装了pip后或者env生效后执行下面命令	
	<pre><code>#$ pip install cloudmonkey</code></pre>

	2.2. 通过源码安装将打包后的源码上传至服务器中解压缩打包后的文件安装
	<pre><code># python setup.py build #
	# python setup.py install</code></pre>


### 配置cloudmonkey
<pre><code>
#cloudmonkey
> set history_file /usr/share/cloudmonkey_history
> set log_file /var/log/cloudmonkey
设置管理服务器
> set host 192.168.56.1
设置端口
> set port 8080
设置使用的apikey
> set apikey {put-your-api-key-for-your-user}
设置使用的secretkey
> set secretkey {put-your-secret-key-for-your-user}
修改cloudmonkey终端提示符
> set prompt smcloudcli>
同步配置
> sync
324 APIs discovered and cached
</code></pre>

配置结束。
通常情况下，所有配置项在 ~/.cloudmonkey/config: 均可见

### 命令使用

tab健可以命令补全
<pre><code>#cloudmonkey
加载用户的key
smcloudcli>sync
smcloudcli>list <tab>
设置数据显示方式为table
smcloudcli>set display table
设置数据显示方式为json
smcloudcli>set display json
smcloudcli>list users
</code></pre>
