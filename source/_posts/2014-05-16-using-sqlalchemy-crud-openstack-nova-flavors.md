---
title: "OpenStack Nova 使用SQLAlchemy 操作Flavor(Mysql backend)"
description: ""
category: openstack
tags: [neutron]
---

##SQLAlchemy [简介]

The SQLAlchemy Object Relational Mapper presents a method of associating user-defined Python classes with database tables, and instances of those classes (objects) with rows in their corresponding tables. It includes a system that transparently synchronizes all changes in state between objects and their related rows, called a unit of work, as well as a system for expressing database queries in terms of the user defined classes and their defined relationships between each other.

我的理解是，SQLAlchemy 是实体/关系映射的一种操作数据的方式， 实体是由Python所编写的类， 这个类的每一个实体，就对应于数据库中的每一条元组（行数据）。其中对这个类的实体的操作，直接映射（影响）到数据库中对应的元组， 这种方式叫做 工作单元（操作单元，有点类似于一个原子操作）

<!-- more -->


自己写的查询Flavor例子
<pre><code>
 1 from sqlalchemy.orm import sessionmaker
 2 from sqlalchemy import create_engine
 3 from sqlalchemy.ext.declarative import declarative_base
 4 from sqlalchemy import Column, Integer, String, Float, Boolean
 5
 6 sql_connection = "mysql://root:root@localhost/nova?charset=utf8"
 7 engine = create_engine(sql_connection, echo=True) #echo=True 设置该项后，可以打印出sql的执行过程
 8 Session = sessionmaker(bind=engine)
 9 session = Session()
10
11 class NovaBase():
12     pass
13
14 BASE = declarative_base() # 用户创建与DB映射的基类
15
16 class InstanceTypes(BASE, NovaBase):
17     __tablename__ = "instance_types" # 数据库中表的名称
18     id = Column(Integer, primary_key=True) #Column 用于定义数据表 "instance_types”的字段， 其参数是该字段的具体的类型
19     name = Column(String(255))
20     memory_mb = Column(Integer)
21     vcpus = Column(Integer)
22     root_gb = Column(Integer)
23     ephemeral_gb = Column(Integer)
24     flavorid = Column(String(255))
25     swap = Column(Integer, nullable=False, default=0)
26     rxtx_factor = Column(Float, nullable=False, default=1)
27     vcpu_weight = Column(Integer, nullable=True)
28     disabled = Column(Boolean, default=False)
29     is_public = Column(Boolean, default=True)
30
31 flavors = session.query(InstanceTypes).all() #查询数据表
32 for flavor in flavors:
33     print flavor.id, flavor.name
</code></pre>
代码输出

<pre><code>
2013-05-15 18:41:10,928 INFO sqlalchemy.engine.base.Engine SELECT DATABASE()
2013-05-15 18:41:10,929 INFO sqlalchemy.engine.base.Engine ()
2013-05-15 18:41:10,933 INFO sqlalchemy.engine.base.Engine SHOW VARIABLES LIKE 'character_set%%'
2013-05-15 18:41:10,934 INFO sqlalchemy.engine.base.Engine ()
2013-05-15 18:41:10,935 INFO sqlalchemy.engine.base.Engine SHOW VARIABLES LIKE 'lower_case_table_names'
2013-05-15 18:41:10,936 INFO sqlalchemy.engine.base.Engine ()
2013-05-15 18:41:10,937 INFO sqlalchemy.engine.base.Engine SHOW COLLATION
2013-05-15 18:41:10,937 INFO sqlalchemy.engine.base.Engine ()
2013-05-15 18:41:10,946 INFO sqlalchemy.engine.base.Engine SHOW VARIABLES LIKE 'sql_mode'
2013-05-15 18:41:10,946 INFO sqlalchemy.engine.base.Engine ()
2013-05-15 18:41:10,948 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2013-05-15 18:41:10,950 INFO sqlalchemy.engine.base.Engine SELECT instance_types.id AS instance_types_id, instance_types.name AS instance_types_name, instance_types.memory_mb AS instance_types_memory_mb, instance_types.vcpus AS instance_types_vcpus, instance_types.root_gb AS instance_types_root_gb, instance_types.ephemeral_gb AS instance_types_ephemeral_gb, instance_types.flavorid AS instance_types_flavorid, instance_types.swap AS instance_types_swap, instance_types.rxtx_factor AS instance_types_rxtx_factor, instance_types.vcpu_weight AS instance_types_vcpu_weight, instance_types.disabled AS instance_types_disabled, instance_types.is_public AS instance_types_is_public
FROM instance_types
2013-05-15 18:41:10,950 INFO sqlalchemy.engine.base.Engine ()

1 m1.medium
2 m1.tiny
3 m1.large
4 m1.xlarge
5 m1.small
6 m1.nano
7 m1.micro
</code></pre>

###Nova 中所对应的代码结构

nova/openstack/common/db/sqlalchemy/models.py 定义了SQLAlchemy 的基类， 所有的对数据的操作，通过该类中定义的方法来实现， 其子类，只需要调用相关的方法即可

nova/db/sqlalchemy/models.py 定义了nova数据中的所有的表,以InstanceTypes 为例

<pre><code>
BASE = declarative_base() # 与上一个类子中的方法一样

class NovaBase(models.SoftDeleteMixin, models.ModelBase): #区别在这里， models 为1中定义的模块
      pass
class InstanceTypes(BASE, NovaBase):
     __tablename__ = "instance_types" # 数据库中表的名称
     id = Column(Integer, primary_key=True) #Column 用于定义数据表 "instance_types”的字段， 其参数是该字段的具体的类型
     name = Column(String(255))
     memory_mb = Column(Integer)
     vcpus = Column(Integer)
     root_gb = Column(Integer)
     ephemeral_gb = Column(Integer)
     flavorid = Column(String(255))
     swap = Column(Integer, nullable=False, default=0)
     rxtx_factor = Column(Float, nullable=False, default=1)
     vcpu_weight = Column(Integer, nullable=True)
     disabled = Column(Boolean, default=False)
     is_public = Column(Boolean, default=True)
</code></pre>

[简介]:http://docs.sqlalchemy.org/en/rel_0_8/orm/tutorial.html
