---
title: "cloud storage swift ceph"
description: ""
category: "Storage"
tags: []
---

###_概述_
许多人对对象存储与像ISCSI、FC这类块存储混同，但是他们之间存在很大的差别。像fc这类的块存储只能提供块设备如系统中的sdb，对象存储只能通过特殊的客户端来访问，像百度网这一类

块存储对于云环境下是重要的一部分，
主要用于存储
虚拟机的镜像文件或者存储用户的文件，包括所有进行备份的数据，文件，图像等。对象存储的主要优势在于比企业级的商业存储更加的节约，并且保证规模扩展性和数据的冗余.

<!-- more -->

###_Openstack Swift_
###软件架构

OpenStack 对象存储(Swift)利用标准服务器组建集群，提供了冗余的，可扩展的分布式对象存储。分布式意味着数据的每一分份在集群中的存储节点上复制。复制的份数是可配置的但是对于生产环境最少是3份。

Swift 中的数据可以通过REST
接口访问，根据需要可以对存储的数据进行相应的操作。
每一个对象的访问路径包含三个元素：
/account/container/object
object存储了用户的实际的输入的数据

Accounts and containers提供了组织对象的方式，不允许嵌套的accounts 和 containers。

Swift 软件以组件的方式进行开发，包括account servers, container servers, 和object servers 除此之外，proxy server哟用于接受用户的api请求
Account servers 对账户提供container listings的操作 Container servers 对于一个给定的容器提供object listings 的操作 Object servers返回数据本身

###_Rings_

由于用户的数据在集群中分布，所以非常的有必要记录数据存放的位置。 Swift通过维护一个叫做_rings_的数据结构来完成数据的定位操作。Rings 在集群中的各个节点复制包括存储节点以及代理节点，这种方式使得swift避免了大部分存储系统依赖于集中单一的meta date服务器的弊端。由于ring文件存储的是集群中node的关系，而不是一个集中的数据map，所以在存储或者删除object是，不需要更新ring文件.这样有益于IO操作，极大程度的减少了访问的带来的不便

对于acount数据库、container数据库、和单独的object都有独立的rings文件，但是都已相同的方式进行工作。简单来说就是，对于一个给定的Account，container，或者object，ring返回的是它在物理存储节点上的位置，从技术的角度来说，这一过程包括[一致性hash算法](http://en.wikipedia.org/wiki/Consistent_hashing)。在mirantis上有对ring工作原理的相关介绍 [under-hood-of-swift-ring](http://mirantis.blogspot.com/2012/02/under-hood-of-swift-ring.html) 和 [openstack-swift-consistency-analysis](https://julien.danjou.info/blog/2012/openstack-swift-consistency-analysis).

![Swift-architecture](http://jiangyt-github-io.qiniudn.com/Swift_Arch.png)

###_Proxy Server_
代理服务器暴露出public API并且接受对存储实体的请求。对于每一个请求，代理节点都会通过ring文件查找account，container以及object所在的物理位置。根据位置将请求转发.Objects 在代理节点和客户端间直接的以流的的方式进行传递，并且没有缓存

###_Object server_
这是一个简单的blob的存储服务器，可以存储数据、查询、删除数据。对象以二进制文件的方式在存储节点中，对象的metadata信息存储在xattrs（file’s extended attributes）上，这就要求存储object的节点必须支持xattrs

每一个object利用对象名的hash值以及操作的时间戳来进行存储路径的存储，最新的写操作会在记录中的最新位置（包括在分布式的场景下，创建的请求需要全局的同步时钟）以保证响应对象的最新的一个版本。删除操作也会记录为文件的一个版本，这就保证了已经删除的对象在集群间正确的复制，老的版本不会出现在用户的请求中

###_Container server_
Container 服务器用于查询object信息（listings）。并不知道具体的object的位置，但是对于一个给定的container可以查询到其包含的objects。listings 数据存储在sqlite3数据库中，并且向object一样，在集群间复制。象object总数、container的存储使用情况的统计值都会记录下来.

在所服务的节点上，会有一个特殊的进程swift-container-updater 不间断的想container数据库中填充数据，在一个container中的数据变化时，并对其数据库进行更新。通过ring来定位需要更新的account。


###_Account server_
与container server雷系，但是是处理container listings 的操作.

###_Features and functions_

- Replication: The number of object copies that can be configured manually.
- Object upload is a synchronous process: The proxy server returns a “201 Created” HTTP code only if more than half the replicas are written.
- Integration with OpenStack identity service (Keystone): Accounts are mapped to tenants.
- Auditing objects consistency: the md5 sum of an object on the file system compared to its metadata stored in xattrs.
- Container synchronization: This makes it possible to synchronize containers across multiple data centers.
- Handoff mechanism: It makes it possible to use an additional node to keep a replica in case of failure.
- If the object is more than 5 Gb, it has to be split: These parts are stored as separate objects and could be read simultaneously.

##Ceph

Ceph is a distributed network storage with distributed metadata management and POSIX semantics. The Ceph object store can be accessed with a number of clients, including a dedicated cmdline tool, FUSE, and  Amazon S3 clients (through a compatibility layer, called “S3 Gateway“).  Ceph is highly modular – different sets of features are provided by different components which one can mix and match. Specifically, for object store accessible via s3 API it is enough to run three of them: object server, monitori

![ceph-architecture](http://jiangyt-github-io.qiniudn.com/Ceph_arch.png)


###_Monitor server_
ceph-mon is a lightweight daemon that provides a consensus for distributed decision-making in a Ceph cluster. It also is the initial point of contact for new clients, and will hand out information about the topology of the cluster. Normally there would be three ceph-mon daemons, on three separate physical machines, isolated from each other; for example, in different racks or rows.

###_Object server_
The actual data put onto Ceph is stored on top of a cluster storage engine called RADOS, deployed on a set of storage nodes.

ceph-osd is the storage daemon that runs on every storage node (object server) in the Ceph cluster. ceph-osd contacts ceph-mon for cluster membership. Its main goal is to service object read/write/etc. requests from clients, It also peers with other ceph-osds for data replication. The data model is fairly simple at this level. There are multiple named pools, and within each pool there are named objects, in a flat namespace (no directories). Each object has both data and metadata. The data for an object is a single, potentially big, series of bytes.  The metadata is an unordered set of key-value pairs. Ceph filesystem uses metadata to store file owner, etc. Underneath, ceph-osd stores the data on a local filesystem. We recommend Btrfs, but any POSIX filesystem that has extended attributes should work.

###_CRUSH algorithm_
While Swift uses rings (md5 hash range mapping against sets of storage nodes) for consistent data distribution and lookup, Ceph uses an algorithm called CRUSH for this.  In short, CRUSH is an algorithm that can calculate the physical location of data in Ceph, given the object name, cluster map and CRUSH rules as input. CRUSH describes the storage cluster in a hierarchy that reflects its physical organization, and thus can also ensure proper data replication on top of physical hardware. Also CRUSH allows data placement to be controlled by policy, which allows CRUSH to adapt to changes in the cluster membership.

###_Rados Gateway_
radosgw is a FastCGI service that provides a RESTful HTTP API to store objects and metadata on the Ceph cluster.

###_Features and functions_
- Partial or complete reads and writes
- Snapshots
- Atomic transactions with features like append, truncate, and clone range
- Object level key-value mappings
- Object replicas management
- Aggregation of objects (series of objects) into a group, and mapping the group to a series of OSDs
- Authentication with shared secret keys: Both the client and the monitor cluster- have a copy of the client’s secret key
- Compatibility with S3/Swift API

###_Feature summary_
<pre><code>
Swift Ceph
Replication Yes Yes
Max. obj.size 5gb(bigger objects segmented) Unlimited
Multi DCinstallation Yes (replication on the container level only,but a blueprint proposed for full inter dc replication) No (demands asynchronous eventual consistency
replication, which Ceph does not yet support)
Integration/w Opentsack Yes Partial(lack of Keystone support)
Replicasmanagement No Yes
Writingalgorithm Synchronous Synchronous
Amazon S3 compatible API Yes Yes
Data placement method Ring (static mapping structure) CRUSH (algorithm)
</code></pre>

###_参考文章_

http://www.mirantis.com/blog/object-storage-openstack-cloud-swift-ceph
