---
layout: post
title: "Setting Django oscar Environment"
description: ""
category: "commerce"
tags: [e-commerce,django,python,posgres]
---

### PSQL Settings

- **bash-4.1$ psql**
- **createdb -T template_postgis oscar**
- **oscar=# \d**
<pre><code>               List of relations
 Schema |       Name        | Type  |  Owner   
 --------+-------------------+-------+----------
  public | geography_columns | view  | postgres
  public | geometry_columns  | table | postgres
  public | spatial_ref_sys   | table | postgres
  (3 rows)
</code></pre>

### Solr Settings
<pre><code>cd ***/solr-4.6.1/example/solr
cp -rf example oscar
cp -f sites/demo/deploy/solr/schema.xml oscar/conf
cd ..
java -jar start.jar
</code></pre>
Browser  http://_ip_:8983/solr finding your app oscar

### Django-oscar demo settings
<pre><code>cd ***/django-oscar/sites/demo
vim settings.py
</code></pre>
-  **update sor connections**
<pre><code>
    271 HAYSTACK_CONNECTIONS = {
    272     'default': {
    273         'ENGINE': 'haystack.backends.solr_backend.SolrEngine',
    274         'URL': 'http://127.0.0.1:8983/solr/oscar',
    275     },
    276 }
</code></pre>

- cd _dir_/django-oscar
- make demo
After a while everyting ok,cd - and run 
_python manage.py  runserver 0.0.0.0:8000_

Browser _ip_:8000
