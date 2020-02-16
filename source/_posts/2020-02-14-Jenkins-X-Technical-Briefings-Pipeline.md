---
title: Jenkins-X Technical Briefings (Pipeline)
date: 2020-02-14 17:19:37
tags: [Kubernetes,CI/CD]
---

[Jenkins-X](https://jenkins-x.io/) is an awesome CI/CD tool that working best with Kubernetes, it is also the fundation of CI/CD as a Service offering of CloudBees company.
Just as the functions it says, Continous Integration and Continous Delivery, but how Jenkins-X achieved that goal? what's the mechanism behind of the tool? 

<!--more-->

## Method
Jenkins-X uses the familiar objects that you would ordinarily use to ship the service; kaniko to build container images within kubernetes; helm charts skeleton to illustrate the deployment of the service; Tekton to assemble tasks into a pipeline; lighthouse(developed internally) or prow to handle the chatops activities. So that's the most important parts of the Jenins-X.

Here's a typical tekton pipeline used for a golang application image building,packaging and delivery to kubernentes.
For a given repository, there would be two pipelines involved

![tekton meta pipeline used in jenkins-x ](/images/jx-tekton-meta-pipeline.jpeg)

![tekton pipeline used in jenkins-x ](/images/jx-tekton-pipeline.jpeg)

The tekton pipeline and the related resources are kubernetes CRDs, including:

1. `PipelineResource`, the only supported types are `git` and `image`
2. a `Task` is the unit to completedly run a job, contains a series of task instance
3. `TaskRun` the single task running instance, contains the full lifecyle of the task.
4. `Pipeline` control multiple tasks which assembled into a pipeline template.
5. `PipelineRun` an instance to run a spcific `pipeline`

To make it simple, the `Task` and `Pipeline` are the templates to accomplish a goal, `TaskRun` and `PipelineRun` are the instance to execute these templates at the given time. 

From the diagram above, we can see that an application contians two pipelines: `meta` and `release`. The `meta` pipeline is used to create effective pipeline dynamatically with the build-pack of the application and in the last step to trigger that templated pipeline with application specific parameters.

```bash
$ tree . -L 2                                                                                                                             
.
├── charts
│   ├── awesome-go
│   ├── golang
│   └── preview
├── curlloop.sh
├── Dockerfile
├── jenkins-x.yml
├── main.go
├── Makefile
├── OWNERS
├── OWNERS_ALIASES
├── README.md
├── skaffold.yaml
└── watch.sh

4 directories, 10 files

$ cat jenkins-x.yml              

buildPack: go

```

the jenkins-x.yml in application source code illustrate the build-pack for building. Since we're using kubernetes based installation, all available buildpacks stored in [github](https://github.com/jenkins-x-buildpacks/jenkins-x-kubernetes), you can create your own buildpack with unique requirement.

## Tekton Pipeline Usage

Let's make some simulations of the critical parts in Jenkins-X, the source code is the one used above.
The workflow is
```bash
clone repo 
  build container image and pushing
    build helm release and uploading to local chartmuseum
```
### Prerequisites
1. Image building
we need the the service account for accessing kubernetes and the credentials for image registry to store container images.

create image registry 
```
kubectl create secret docker-registry registrysecret \
    --docker-server=<registry address> \
    --docker-username=<user name> \
    --docker-password=<password>
```

create git repository access token
```
kubectl create secret generic git-credentials --type=kubernetes.io/basic-auth \ 
    --from-literal=username=<username> \
    --from-literal=password=<apitoken>
```

create kubernetes service account k8s-sa.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-bot-sa
secrets:
- name: registysecret
- name: git-credentials
```
```
kubectl apply -f k8s-sa.yaml

```

create the clusterrolebinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tekton-bot-handson
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: tekton-bot-sa
  namespace: jx
```

create helm release registry chartmuseum
```bash
docker run --restart=always -d -it -p 8080:8080 -e DEBUG=1 -e STORAGE=local -e STORAGE_LOCAL_ROOTDIR=/charts -v $PWD/chartmuseum-int:/charts  chartmuseum/chartmuseum:latest

```

### Tekton Pipeline Misc

#### PipelineResources

```yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: jiangytcn-awesome-go-master
  namespace: tekton-handson
spec:
  params:
  - name: revision
    value: v0.0.1
  - name: url
    value: http://github.192.168.1.12.nip.io/jiangytcn/awesome-go.git
  type: git
```
> kubectl apply -f tekton-pipelineresource.yaml 

#### Task
> kubectl apply -f tekton-task-build.yaml  
#### Pipeline
create pipeline that reference to task and hte pipelineresource created above
> kubectl apply -f tekton-pipeline.yaml
#### PipelineRun
create a pipeline instance with required parameters 
> kubectl apply -f tekton-pipelinerun.yaml

**_The pipeline won't run until create the pipelinerun instance_**

![Tekton Pipeline Simulation](/images/jx-tekton-simulation.gif)