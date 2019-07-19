---
layout: post
title: "Django oscar I18N"
description: ""
category: "commerce"
tags: [e-commerce,Django]
---

### Translating Oscar 

Within your project, create a locale folder and a symlink to Oscar so that ./manage.py makemessages finds Oscar’s translatable strings:

<pre><code>mkdir locale i18n
ln -s $PATH_TO_OSCAR i18n/oscar
./manage.py makemessages --symlinks --locale=zh_CN
</code></pre>

Now the i18n files are under  _local_ directory,edit django.po files
and run command to generate django.mo file.

<pre><code>
./manage.py compilemessages --locale=zh_CN
</code></pre>

Restart your django server, vist your website to verify.
