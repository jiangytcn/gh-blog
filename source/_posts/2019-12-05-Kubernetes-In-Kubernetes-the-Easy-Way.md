---
title: Kubernetes In Kubernetes the Easy Way
date: 2019-12-05 22:09:00
tags: [CloudNative, OpenStack, Kubernetes, MultiCloud]
---

If you are planing to adopt CloudNative way to ship your products in an efficient way, then [Kubernetes](https://kubernetes.io/) might be a good choice to mondernize IT.After serveral years growth, the product went into an stable state and adpoted by large number of companies in Production environments. All major cloud prvoider players offerred their own kubernretes service with their own addons. Kelsey Hightower wrote an good article [Kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) to help you better understanding the tasks needed to deploy a workable kubernetes cluster. The minimal tasks needed are:

<!--more-->
1. container engine installation
1. networking plugin installation
1. certificate management and distribution
2. storage backend management
3. components configuration, apiserver, controller, scheduller, ...

More steps would be added to the the configureation pipelines as more features needed. To expose your service can be accessed publicly, you need to deploy a ingress service, to host a stateful service, for example, for a 3 tiers architecture, you need to deploy a database in the cluster, then cames the CSI compoennts.

So, how can I get my own kubernetes based platform in an easy and efficient way ? how can I have multiple clusters with mimimal resources needed ? and how to keep up with the relases provided by the community to address the CVE, Bug Fix or new feature needed ? 


There are so many deployment tools around the kubenetes, and from CNCF there're couple of [certified installers](https://landscape.cncf.io/category=certified-kubernetes-installer&format=card-mode&grouping=category) to help your spin up and manage kuberntes cluster.

![cncf certified k8s installer](/images/cncf-certified-k8s-installer.png)


From all of these tools, the uniqueness of [Gardener](https://gardener.cloud/) is that the kubernets can be managed by kubernetes, fully utilized the benefit of kubernetes, CustomResource, Deployment, Ingress, etc

_want to upgrade a new version of kubernetes cluster?_ that's easy, just use the webgui or with kubectl same as manage normal kube resource;

_want to reduce the resources occupied by the control plane of kubernetes?_ that's easy, all end user kubernets cluster's control plane would be running as pod;

_want to have a consistent deployment and fine gained control accross your different kubernets?_ that's easy, gardener using Terraform and couple extensions to help you manage the resource deployed and configured; no human interaction;


### So, how to setup in my environment ?

here's my environment and all components setting, basically, you need to have an workable IaaS, a existed kubernetes cluster to host all gardener components, A DNS service to manage the records created for ingress services, and load balancer service which can be used for your kubernetes cloud provider.Lastly, need to have an object storeage to store the backups for resilence.

|Compoents|Version|Note|
|---|---|---|
|IaaS - OpenStack|stable/rocky| The OpenStack IaaS is setup in my home lab environment backed by a 64G E5-2660 V2 DIY server, installed and configured with [openstack-ansible](https://git.openstack.org/openstack/openstack-ansible)|
|DNS - OpenStack-Designate | stable/rocky |along side the designate, hosted [CoreDNS](https://coredns.io/) as a DNS proxy, this is optional, just for debugging DNS related issue and to answer the requests outside the openstack environment|
|LbaaS - OpenStack Haproxy based | stable/rocky ||
|Base Cluster -- K8S|1.16.2| Base K8S Cluster is setup with [kubespray](https://github.com/kubernetes-sigs/kubespray.git) and my patch for fixing the etcd issue, my [PR](https://github.com/kubernetes-sigs/kubespray/pull/5368) is not merged by community yet, so apply it if you had same [issue](https://github.com/kubernetes-sigs/kubespray/issues/4772)|
|Gardener - [garden-setup](https://github.com/gardener/garden-setup) | 1.17.0 | |

Before start the journey to expore Gardener, couples things need to configured correctly otherwise you will be failed to install gardener. 

1. The base kubernetes cluster  network CNI configuration.
If your're using calico as the network cni for the undercloud cluster, the mtu need to be set correctly, otherwise the gardener components won't connecte with each other correctly. choose approiate value per the guidance from [calico networking MTU Configuration](https://docs.projectcalico.org/v3.10/networking/mtu) based on your own environment. For me, the cluster setup with IPIP mode and no jumboframe configured in my switch, so its `1430`
2. the images can be used in gardener.
After couple hours debugging and supported by gardener community, only a subset images supported by the installation miscs in Gardener, I only tried CoreOS and Ubuntu, gardener itself supported other os distributions, but no judgement about the compatibilities of other oses.
| OS | Version| Compatible |
|--|---| --- |
| Ubuntu | 18.04| Y
| Ubuntu | 16.04 | X
| CoreOS | 2135.6.0 | Y |
| CoreOS | 2023.5.0 | Y |

**the image used for kubernetes worker nodes need to be uploaded to IaaS before the gardener installation start, it won't download the image for you**

3. OpenStack API Access reachable from your kube cluster.
Gardener using terrafrom to create resources on demand in your OpenStack environment, so need to make your IaaS reachable from the pods running in the kubernete cluster, dns, routing, etc.

### Installation under OpenStack

The installation steps is easy, providing the configuration file named `acre.yml` with correct setting along side a single command, that's it!

You can follow this [guide](https://github.com/gardener/garden-setup/blob/1.17.0/README.md) to start the deployment, the processes are running under docker container. If you want to run that under your desktop to see how the majic happened, get the necessary tools from [SOW Dockerfile](https://github.com/gardener/sow/blob/master/docker/Dockerfile) and configure `PATH` correctly.

The basic steps are:
1. get your base kubernetes kubeconfig file
2. configure `acre.yaml` correctly
3. verify the configuration by `sow order -A`
4. deploy the gardener by `sow deploy -A`
5. access the gardener builtin ui by `sow url`


```plain
credentials: &openstack
  username: admin
  password: ae226d1f8b27c60b31088
  tenantName: admin
  domainName: Default
  userDomainName: Default
  authURL: https://api.int.ystacks.com:5000/v3
  region: RegionOne
  bucketName: gardener
landscape:
  name: overcloud-caas-ystacks-com
  domain: overcloud.caas.ystacks.com
  cluster:
    kubeconfig: ./kubeconfig
    domain: overcloud.caas.ystacks.com
    iaas: openstack
    networks:
      pods: 10.44.0.0/15
      nodes: 10.132.0.0/16
      services: 10.233.10.0/24 # change to under actual undercloud service ip cidr
  iaas:
    - name: openstack
      type: openstack
      mode: seed
      region: RegionOne
      zones:
      - nova
      credentials: *openstack
      floatingPools:
      - name: public
      loadBalancerProviders:
      - name: haproxy
      extensionConfig:
        machineImages:
        - cloudProfiles:
          - image: coreos-2135.6.0
            name: openstack       # A
          name: coreos              # B
          version: 2135.6.0         # C
        - cloudProfiles:
          - image: ubuntu-18.04
            name: openstack       # A
          name: ubuntu              # B
          version: "18.04"        # C
      machineImages:
        - name: coreos
          versions:
          - version: 2135.6.0
        - name: openstack
          versions:
          - version: "18.04"
      machineTypes:
        - name: 2C4G50G
          cpu: "2"
          gpu: "0"
          memory: 4Gi
          usable: true
          storage:
            class: default
            type: default
            size: 50Gi
        - name: 4C8G50G
          cpu: "4"
          gpu: "0"
          memory: 8Gi
          usable: true
          storage:
            class: default
            type: default
            size: 50Gi
  etcd:
    backup:
      type: swift
      credentials: *openstack
  dns:
    type: openstack-designate
    credentials: *openstack
  identity:
    users:
      - email: jiangyt.cn@gmail.com
        username: jiangyt.cn@gmail.com
        password: password
```



After deploy you gardener cluster, accessing `sow url` to visit gardener dashboard and login with the user provided in identity above

![Gardener Dashboard after logon in cluster view](/images/gardener-dashboard-clusters.png)

_Note:created two clusters by gardener with different kubernetes versions_



### Useful Commands

#### Edit Machine Image and Type on the fly
The machine image versions and types are stored as CR in the seed kube-apiserver, so targetting the seed kube virtual cluster with exported kubeconfig file to edit resources

>1. kubectl --kubeconfig export/kube-apiserver/kubeconfig edit controllerregistration.core.gardener.cloud/provider-openstack
>2. kubectl --kubeconfig export/kube-apiserver/kubeconfig edit cloudprofiles/openstack
