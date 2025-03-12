---
creation date: 2024-11-13 11:22
modification date: 2025-03-04 21:16
tags:
  - K8S
---

参考：
[Kubernetes一小时轻松入门_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Se411r7vY/?spm_id_from=333.337.search-card.all.click&vd_source=dbefe8621d153f1b1aaef0768a993d25)

[前言 | Docker — 从入门到实践](https://yeasy.gitbook.io/docker_practice)

[为什么程序员都应该学用容器技术【让编程再次伟大#26】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1VTruYsEYg/?spm_id_from=333.1007.tianma.2-3-6.click&vd_source=dbefe8621d153f1b1aaef0768a993d25)


# 简介
Kubernetes是⼀个**开源的容器编排引擎**， 可以⽤来**管理容器化的应⽤**， 包括**容器的⾃动化的部署、扩容、缩容、升级、回滚**等等， 它是Google在2014年开源的⼀个项⽬， 它的前身是Google内部的Borg系统。在Kubernetes出现之前， 我们⼀般都是使⽤[[Docker]]来**管理容器化的应⽤**， 但是[[Docker]]只是⼀个单机的容器管理⼯具， 它只能管理单个节点上的容器， 当我们的应⽤程序需要运⾏在多个节点上的时候， 就需要使⽤⼀些其他的⼯具来管理这些节点， ⽐如Docker Swarm、Mesos、**Kubernetes**等等


以下是Kubernetes的主要特性：
1. 容器编排：Kubernetes可以管理大规模的容器部署，使得应用能够在不同的环境中（如物理服务器、虚拟机、云环境等）一致运行。
2. 自动化：Kubernetes自动化处理容器的部署、复制、扩展、负载均衡和自愈等任务。
3. 集群管理：Kubernetes使用一个集群来管理容器应用，一个集群由一个或多个节点组成。每个节点运行容器化的应用。
4. 服务发现和负载均衡：Kubernetes提供内置的服务发现和负载均衡机制，使得服务之间的通信更加方便。
5. 存储编排：Kubernetes支持挂载多种存储系统，如本地存储、云存储、分布式存储等，以满足不同的持久化需求。
6. 扩展性：Kubernetes通过自定义资源和控制器可以方便地扩展，以适应特定的应用需求。

# 核心组件
## 基础组件——Node/Pod/Service/ConfigMap/Secret/Volume
1. 一个**Node**对应一个物理机或一个虚拟机，在其上可以运行若干个**Pod**，Pod是K8S最小调度单元，Pod是一个或多个容器的组合
	- 一般一个Pod只存放一个容器，除非容器高度耦合
- Pod之间通过内部IP地址来访问（集群创建时分配）
	- 这意味着外部容器无法访问
	- Pod容易被销毁然后重建会重新分配IP地址

2. 为解决Pod IP地址不稳定，使用**Service**作为统一入口来访问
	- Service可分为内部Service和外部Service，外部Service可以通过node:port（ip:port）的形式暴露后端API、前端界面等服务给外部，而内部Service仅能在集群内部被访问

3. 在测试和开发阶段使用node:port（ip:port）的形式没问题，在投入生产环境后，一般通过域名访问服务，使用**Ingress**来管理从集群外部访问集群内部服务的入口和方式

4. 为了降低应用程序和配置信息的耦合度，避免停机，使用**ConfigMap**将配置信息封装起来将环境配置信息和容器镜像解耦，便于应用程序的修改

5. ConfigMap是明文存储的，为了避免敏感信息泄露需要使用**Secret**，但其仅做了一层base64编码，也不能保证敏感信息安全，需要使用K8S提供的其他机制来保证安全如：网络安全、访问控制、身份认证等

6. 当容器被销毁或重启时，容器中的数据会消失，对于需要持久化存储的应用程序需要使用**Volume**将数据挂载到本地硬盘或云端存储中

## 高可用——Deployment/StatefulSet
1. Deployment可以定义和管理**应用程序**的**副本数量**以及应用程序的更新策略，具有：副本控制、滚动更新等高级特性

2. StatefulSet 是⽤来管理有状态应⽤（例如数据库对象、缓存）的⼯作负载 API 对象。StatefulSet ⽤来管理某 Pod 集合的部署和扩缩，并为这些 Pod 提供持久存储和持久标识符，也具有：副本控制、滚动更新等高级特性

# 架构
典型的Master-Worker架构
Kubernetes ⾥的 Master 指的是集群控制节点,  每⼀个 Kubernetes 集群⾥ 都必须要有⼀个 Master 节点来负责整个集群的管理和控制. 基本上 Kubernetes 的所有控制命令都发给它, 它来负责具体的执⾏过程。 我们后 ⾯执⾏的所有命令基本都是在 Master 节点上运⾏的


1. Node结点由kubelet、kube-proxy、container-runtime三个组件组成
	1. kubelet负责管理和维护每个节点上的Pod
	2. k-proxy负责网络通信，实现了服务暴露和转发等网络功能
	3. container-runtime为应用程序提供运行时环境
2. Master结点由API Server、scheduler、etcd、control manager四个组件组成
	1. etcd 是兼具⼀致性和⾼可⽤性的键值数据库，可⽤于服务发现以及配置中 ⼼， 采⽤ raft ⼀致性算法， 基于 Go 语⾔实现， 是保存 Kubernetes 所有集群 数据的后台数据库
	2. kube-scheduler 是 kubernetes 系统的核⼼组件之⼀， 主要负责整个集群 资源的调度功能， 根据特定的调度算法和策略， 将 Pod 调度到最优的⼯作 节点上⾯去， 从⽽更加合理、 更加充分地利⽤集群的资源。
	3. Controller Manager 的作⽤简⽽⾔之， 保证集群中各种资源的实际状态 （status） 和⽤户定义的期望状态 （spec） ⼀致。
	4. api-server 是集群的核⼼， 是 k8s 中最重要的组件， 因为它是实现声明式 api 的关键， kubernetes api-server 的核⼼功能是提供了 Kubernetes 各类 资源对象 （pod、 RC 、 service 等） 的增、 删、 改、 查以及 watch 等 HTTP REST 接⼝
![[../../Attachment/Pasted image 20240530153903.png]]

# kubectl
## 前置工作
配置真正的 K8s 集群步骤较为繁琐，为了简化环境配置，我们使用 Kind（Kubernetes In Docker）， 一种使用 Docker 容器作为 node 节点，运行本地 Kubernetes 集群的工具。
Kind安装参考官方文档：[kind – Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/)

安装kubectl：[在 Linux 系统中安装并设置 kubectl | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/)
## 基本命令
```shell
kubectl run nginx --image=nginx # 使用nginx镜像来创建并运行pod nginx
kubectl create -f Deployment.yaml # 根据yaml文件创建资源
kubectl delete -f Deployment.yaml # 根据yaml文件删除资源
kubectl get nodes/pod/svc # 获取资源信息

kubectl create deployment nginx-deployment --image=nginx # 创建Deployment使用nginx镜像，名字为nginx-deployment

kubectl edit deployment nginx-deployment # 编辑Deployment的配置文件

kubectl logs nginx # 查看nginx pod的日志
kubectl exec -it <container-name> --/bin/bash # 通过bash以交互的方式进入容器中
```

注意：
1. 创建Deployment会自动创建管理Pod的replicaset

