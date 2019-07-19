---
layout: post
title: "OpenStack Nova 添加扩展API流程"
description: ""
category: "openstack"
tags: Nova
---

例子中涉及到SQLAlchemy 得相关操作，可以参考 [上一随笔]

Openstack 中规定，扩展openstack得api有两种方式

>创建新的WSGI 资源

>扩展原有得WSGI资源得控制器（我得理解是，接受到API请求后，具体得响应逻辑）

>这两种方式中，都要求写一个新的模块来声明控制器类去处理请求和实现扩展。


在一个API模块中，可以有一个或多个得资源和扩展控制器。

根据osapi_compute_extension 得配置， ExtensionManager 由nova/api/openstack/compute/contrib/ 下的__init__.py 文件加载标准的或者新的扩展。

所以扩展的api统一写在nova/api/openstack/compute/contrib/ 目录下

如本例子中得 nova/api/openstack/compute/contrib/documents.py


##扩展API流程

实现控制器，完成对资源的基本操作，如增删改查和其他一些用户自定义的RESTful资源操作；

实现一个extensions.ExtensionDescriptor的子类， 并实现get_resources 或者 get_controller_extensions，来建立新的资源或扩展资源控制器（即改写原有的业务逻辑）。具体实现哪个方法这取决于是否要修改原有的RESTFul业务逻辑， 或者说两个功能都需要；

将控制器和扩展的资源类，写如新创建的的资源中； 规范是资源类是模块（问家名）首字母大写，这样做的目的是使nova.api.openstack.extensions.load_standard_extensions这个类能够是别该新资源，并予以加载；

当添加新的资源（extensions.ResourceExtension的子类）的时候，如果想要去除掉 {tenent_id} 链接，则需要编写自定义的路由访问规则（本例子中没有涉及）

###documents.py 实现

<pre><code>1 # vim: tabstop=4 shiftwidth=4 softtabstop=4
 2
 3 # Author:  Yitao Jiang
 4 # Email:  willierjyt@gmail.com
 5
 6 import webob
 7 from webob import exc
 8
 9 from nova import db
10 from nova import exception
11 from nova.api.openstack import extensions
12 authorize = extensions.extension_authorizer('compute', 'documents')
13
14 # 请求控制器， 即处理对资源的请求，予以响应
15 class DocumentsController():
16         """the Documents API Controller declearation"""
17
18         def index(self, req):
19             import pdb; pdb.set_trace()
20             documents = {}
21             context = req.environ['nova.context']
22             authorize(context)
23
24             documents["key"] =  "helloworld"
25             return documents
26
27         def create(self, req):
28             documents = {}
29             context = req.environ['nova.context']
30             authorize(context)
31
32             documents["key"] =  "helloworld"
33             return documents
34
35         def show(self, req, id):
37             documents = {}
38             context = req.environ['nova.context']
39             authorize(context)
40
41             try:
42                 document = db.document_get(context, id)
43             except :
44                 raise webob.exc.HTTPNotFound(explanation="Document not found")
45
46             documents["document"] = document
47             return documents
48
49         def update(self, req):
50             documents = {}
51             context = req.environ['nova.context']
52             authorize(context)
53
54             documents["key"] =  "helloworld"
55             return documents
56
57         def delete(self, req, id):
58             return webob.Response(status_int=202)
59 # 根据命名规范， 模块（python源文件）中的类名是模块名的首字母大写
60 class Documents(extensions.ExtensionDescriptor):
61         """Documents ExtensionDescriptor implementation"""
62
63         name = "documents"
64         alias = "os-documents"
65         namespace = "www.www.com"
66         updated = "2013-05-19T00:00:00+00:00"
67
68         def get_resources(self):
69             """register the new Documents Restful resource"""
70
71             resources = [extensions.ResourceExtension('os-documents',
72                 DocumentsController())
73                 ]
74
75             return resources
</code></pre>

在之后可以由以下几种方式来操作documents 资源

- GET v2/{tenant_id}/ os-documents
- POST v2/{tenant_id}/ os-documents 
- GET v2/{tenant_id}/ os-documents/{document_id}
- PUT v2/{tenant_id}/ os-documents/{document_id}
- DELETE v2/{tenant_id}/ os-documents/{document_id}


###扩展api时所修改的文件

<pre><code>
1  nova/db/api.py
2  nova/db/sqlalchemy/api.py
3  nova/db/sqlalchemy/models.py
</code></pre>

nova/db/api.py 文件内容

####数据操作API提供的方法，由Nova API 根据请求进行相应的操作， 由上面的请求控制器进行调用
<pre><code>
1 def document_get(context, document_id):
2        """Get a document or raise if it does not exist."""
3        return IMPL.document_get(context, document_id)

nova/db/sqlalchemy/api.py 文件内容

# 完成通过由SQLAlchemy操作数据库

 1  @require_admin_context
 2  def document_get(context, document_id):
 3
 4        session = get_session()
 5        with session.begin():
 6                query = model_query(context, models.Document, session=session, read_deleted="yes").filter_by(id=document_id)
 7
 8                result = query.first()
 9
10                if not result or not query:
11                        raise Exception()
12
13                return result
SQLAlchemy 中定义的资源

nova/db/sqlalchemy/models.py（具体使用见上一 篇日志）

1 class Document(BASE, NovaBase):
2        """Represents a document of customized extension."""
3
4        __tablename__ = 'documents'
5        id = Column(Integer, primary_key=True)
6        title = Column(String(255))
</code></pre>

至此，添加添加新的nova API功能完成,重起api服务

调用 curl -v -X GET -H 'X-Auth-Token: 8e5971b3ce0a4f039b895681e7c29361' http://127.0.0.1:8774/v2/9b5903dd2d3443d8bb75ddffac27239a/extensions/os-documents  | python -mjson.tool命令，
可以看到返回，扩展添加成功

<pre><code>
 1 {
 2     "extension": {
 3         "alias": "os-documents",
 4         "description": "Documents ExtensionDescriptor implementation",
 5         "links": [],
 6         "name": "documents",
 7         "namespace": "www.www.com",
 8         "updated": "2013-05-19T00:00:00+00:00"
 9     }
10 }

数据库添加表documents， 结构如下

mysql> desc documents;
+------------+--------------+------+-----+---------+----------------+
| Field      | Type         | Null | Key | Default | Extra          |
+------------+--------------+------+-----+---------+----------------+
| id         | int(11)      | NO   | PRI | NULL    | auto_increment |
| title      | varchar(255) | NO   |     | NULL    |                |
| created_at | datetime     | YES  |     | NULL    |                |
| updated_at | datetime     | YES  |     | NULL    |                |
| deleted_at | datetime     | YES  |     | NULL    |                |
| deleted    | int(11)      | YES  |     | NULL    |                |
+------------+--------------+------+-----+---------+----------------+

6 rows in set (0.05 sec)

mysql> select * from documents;
+----+----------------+------------+------------+------------+---------+
| id | title          | created_at | updated_at | deleted_at | deleted |
+----+----------------+------------+------------+------------+---------+
|  1 | abcdefgiifeife | NULL       | NULL       | NULL       |    NULL |
|  2 | 1qaz2wsx       | NULL       | NULL       | NULL       |    NULL |
+----+----------------+------------+------------+------------+---------+
2 rows in set (0.03 sec)

 1 curl  -X GET -H 'X-Auth-Token: 8e5971b3ce0a4f039b895681e7c29361' http://127.0.0.1:8774/v2/9b5903dd2d3443d8bb75ddffac27239a/os-documents/1  | python -mjson.tool
 2 返回结果
 3 {
 4     "document": {
 5         "created_at": null,
 6         "deleted": null,
 7         "deleted_at": null,
 8         "id": 1,
 9         "title": "abcdefgiifeife",
10         "updated_at": null
11     }
12 }
</code></pre>
至此新添加的资源，API 扩展成功， 可以在此基础上进行进一步的修改，完成需求

###参考文档：

1. [Adding a Method to the OpenStack API](http://docs.openstack.org/developer/nova/devref/addmethod.openstackapi.html)

[上一随笔]:/openstack/2014/05/16/using-sqlalchemy-crud-openstack-nova-flavors/
