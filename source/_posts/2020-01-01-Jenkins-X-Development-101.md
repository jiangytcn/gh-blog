---
title: Jenkins-X-Development-101
date: 2020-01-01 13:37:49
tags: CI/CD
---

[Jenkins-X](https://jenkins-x.io/) is an awesome CI/CD tool that working best with Kubernetes, it is also the fundation of [CI/CD as a Service](https://www.cloudbees.com/products/cloudbees-ci-cd/overview) company of CloudBees. All of the buzz words and the new techs adopted. GitOps, Istio, Multi-Cluster, Approval mechanism...

But the project is still under developing, some features may not implemented or have minior issues need to be fixed.

The article outlined the tasks need to hack the jenkins-x with your own requirment and environment.

<!--more-->

### Prerequisite

|Resource | Version|
|---|---|
|Jenkins X| A JX instance deployed with `jx boot`
|Docker Registry | - |
|Helm Chart Repository | chartmuseum|
|Golang Environment| 1.11 or 1.12|

and also forked the following repository into your own org
1. https://github.com/jenkins-x/jenkins-x-boot-config
2. https://github.com/jenkins-x/jenkins-x-platform
3. https://github.com/jenkins-x/jenkins-x-versions
4. the codebase repository that you're trying to hack

### Persona

Assuming I want to make the Jenkins X works on the kube api 1.16+. 
So if you followed the docs to install jenkins X on a on premise kube cluster like k3s distribution `v1.16.3-k3s.2`, you should hit the issue like below:

```
configmap/config-entrypoint created
configmap/config-logging created
unable to recognize "/var/folders/62/l5t1s2lj17jd5nwlzw70st1r0000gn/T/helm-template-workdir-114960915/jenkins-x/output/namespaces/jx/env/charts/jenkins-x-platform/charts/chartmuseum/templates/part0-deployment.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
unable to recognize "/var/folders/62/l5t1s2lj17jd5nwlzw70st1r0000gn/T/helm-template-workdir-114960915/jenkins-x/output/namespaces/jx/env/charts/jenkins-x-platform/charts/heapster/templates/part0-deployment.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
unable to recognize "/var/folders/62/l5t1s2lj17jd5nwlzw70st1r0000gn/T/helm-template-workdir-114960915/jenkins-x/output/namespaces/jx/env/charts/jenkins-x-platform/charts/nexus/templates/part0-deployment.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
unable to recognize "/var/folders/62/l5t1s2lj17jd5nwlzw70st1r0000gn/T/helm-template-workdir-114960915/jenkins-x/output/namespaces/jx/env/charts/lighthouse/templates/part0-deployment.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
unable to recognize "/var/folders/62/l5t1s2lj17jd5nwlzw70st1r0000gn/T/helm-template-workdir-114960915/jenkins-x/output/namespaces/jx/env/charts/lighthouse/templates/part0-tide-deployment.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
unable to recognize "/var/folders/62/l5t1s2lj17jd5nwlzw70st1r0000gn/T/helm-template-workdir-114960915/jenkins-x/output/namespaces/jx/env/charts/tekton/templates/part0-controller.yaml": no matches for kind "Deployment" in version "apps/v1beta1"'
error: failed to interpret pipeline file jenkins-x.yml: failed to run '/bin/sh -c jx step helm apply --boot --remote --name jenkins-x --provider-values-dir ../kubeProviders' command in directory 'env', output: '
```

The logs above shows that there are issues in the helm charts: `jenkins-platform`, `lighthouse` and `tekton`, so let's see what's the issue it is and  forked these repos. 

Let's take lighthouse for example:

https://github.com/jiangytcn/lighthouse/blob/master/charts/lighthouse/templates/deployment.yaml#L3

The deployment is using wrong resource version which not compatible with my kube cluster in https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/, so did with other repos. 

Ok, so we got the root cause of the problems and the fixes for that, how can I applied these changes to the jx environment ? 

### Hacks

Cause JX is using helm charts for the deployment, before we apply changes in it, let udpate the helm charts of problematic repos. 

For the development purpose, I setup the helm repository locally with chartmusem without auth.

1. build customized charts

```
$ tree . -L 1
.
├── Makefile
├── jenkins-x-platform
├── jenkins-x-versions
├── lighthouse
├── platform-config
└── tekton
```
After update the code in ligthouse, tekton upload the charts to local registry

```bash
CHART_REPO := ${CHART_REPO}
NAME := ${NAME}
OS := $(shell uname)

init:
	helm init --client-only

setup: init
	helm repo add jenkinsxio http://chartmuseum.jenkins-x.io

build: clean setup
	helm dependency build ${NAME}
	helm lint ${NAME}

clean:
	git checkout "${NAME}/Chart.yaml"
	rm -rf ${NAME}/charts
	rm -rf ${NAME}/${NAME}*.tgz
	rm -rf ${NAME}*.tgz
	rm -rf ${NAME}/requirements.lock
	curl -w '\n' -X DELETE $(CHART_REPO)/api/charts/${NAME}/${VERSION}

release: clean build
ifeq ($(OS),Darwin)
	sed -i -e "s/version:.*/version: $(VERSION)/" "${NAME}/Chart.yaml"
else ifeq ($(OS),Linux)
	sed -i -e "s/version:.*/version: $(VERSION)/" ${NAME}/Chart.yaml
else
	exit -1
endif
	helm package ${NAME}
	curl -w '\n' --fail --data-binary "@$(NAME)-$(VERSION).tgz" $(CHART_REPO)/api/charts
	rm -rf ${NAME}*.tgz
	git checkout "${NAME}/Chart.yaml"

delete-from-chartmuseum:
	curl --fail -X DELETE $(CHART_REPO)/api/charts/$(NAME)/$(VERSION)
```
Copy the content above to Makefile.

Build chart separately and upload to registry with:
```
cd tekton
NAME=tekton VERSION=100.0.0 CHART_REPO=http://chartmuseum.int.caas.ystacks.com:8080 make -f ../Makefile release

cd lighthouse/charts
NAME=lighthouse VERSION=100.0.0 CHART_REPO=http://chartmuseum.int.caas.ystacks.com:8080 make -f ../../Makefile release

cd jenkins-x-platform
NAME=jenkins-x-platform VERSION=100.0.0 CHART_REPO=http://chartmuseum.int.caas.ystacks.com:8080 make -f ../Makefile release

```

Then
Add your local helm charts registry to the `jenkins-x-versions` for searching the customized charts
https://github.com/jiangytcn/jenkins-x-versions/blob/chore-compatible-with-1.16/charts/repositories.yml#L12
and the version for other repos
https://github.com/jiangytcn/jenkins-x-versions/blob/chore-compatible-with-1.16/charts/jenkins-x/lighthouse.yml#L2
https://github.com/jiangytcn/jenkins-x-versions/blob/chore-compatible-with-1.16/charts/jenkins-x/tekton.yml#L2

after finished the change, bump a new tag for hack branch - v100.0.0

### Deploy with the fix
1. update the jx deployment file `jx-requriement.yml`
change the section to your own forked url with valid ref
```
versionStream:
  ref: v100.0.0
  url: https://github.com/jiangytcn/jenkins-x-versions.git
```

and `jx boot`

### Reference:
[concepts](https://jenkins-x.io/docs/concepts/)
[contributing](https://jenkins-x.io/docs/contributing/)

Drop me messages for any issue and happy hacking :)
