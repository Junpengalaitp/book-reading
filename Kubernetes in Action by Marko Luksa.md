### 1 Kubernetes介绍
* 大型的单体应用正在被分解成小的、可独立运行的微服务，彼此之间解耦，所以它们可以被独立开部署、升级、伸缩。
* k8s使开发者可以自主部署应用，并且控制部署的频率，完全脱离运维团队的帮助。
* 同时让运维团队监控整个系统，并且在硬件故障时重新调度应用。
* 抽象了数据中心的硬件基础设施，使得对外暴露的只是一个巨大的资源池。部署和运行组件时，不用关心底层的服务器。
* 为应用程序提供一个一致的环境。
* 迈向持续交付：DevOps和NoOps
* Kubernetes的核心功能
  * 整个系统由一个主节点和若干工作节点组成，集群的规模不会造成差异性
  * 帮助开发者聚焦核心应用功能
  * 帮助运维团队获取更高的资源利用率
* Kubernetes的集群架构
  * 主节点
    * 承载kubernetes控制和管理的整个集群系统的控制面板
  * 工作节点
    * 运行用户实际部署的应用
  * 控制面板
    * 控制集群并使它们工作
      * Kubernetes API服务器：你和其他控制面板组件都要和它通信
      * Scheduler: 调度的你的应用
      * Controller Manager: 执行集群级别的功能
      * etcd: 一个可靠的分布式数据存储，它能持久化存储集群配置
  * 工作节点
    * 运行容器化应用的机器
      * Docker、rtk 或者其他的容器类型
      * Kubelet: 与API服务通信，并管理它所在的节点容器
      * Kubernetes Service Proxy(kube-proxy): 负责组件之间的负载均衡网络流量
* 使用Kubernetes的好处
  * 简化应用程序部署
  * 更好地利用硬件
  * 健康检查和自修复
  * 自动扩容
  * 简化应用部署

### 2 开始使用Kubernetes和Docker
* Pod
  * 一组紧密相关的容器，总是一起运行在同一个工作节点上，以及同一个Linux命名空间中。相反，它使用多个共存容器的理念。
  * 每个pod像一个独立的逻辑机器，拥有自己的IP、主机名、进程等，运行一个独立的应用程序。
  * 一个pod的所有容器都运行在同一个逻辑机器上，其他pod中的容器，即使运行在同一个工作节点上、也会出现在不同的节点上。
* 幕后发生的事情
  * 运行kubectl命令时，它通过像Kubernetes API服务器发送一个REST API请求，在集群中创建一个新的ReplicationController对象。
  * 然后，ReplicationController创建了新的Pod, 调度器将其调度到一个工作节点上。
  * Kubelet看到pod被调度到节点上，就告知Docker从镜像中心拉去指定的镜像，因为本地没有该镜像。
  * 下载镜像后，Docker创建并运行容器。
* 访问Web应用
  * 每个pod的IP地址是集群内部的，不能从外部集群访问。
  * 常规服务(ClusterIP服务)，比如pod，只能从集群内部访问。
  * LoadBalancer服务，可通过负载均衡的公共IP访问pod。
* 系统的逻辑区分
  * ReplicationController、pod和服务是如何组合到一起的
    * 通过kubectl run命令，没有直接创建出任何pod，而是创建了一个ReplicationController, 用于创建pod实例。
    * 为了使该pod能从集群外部访问，需要让Kubernetes将该ReplicationController管理的所有pod由一个服务对外暴露。
  * Pod和它的容器
    * 只包含一个容器，但是通常一个pod可以包含任意数量的容器。
  * ReplicationController
    * 确保始终存在一个运行中的pod实例。
    * 通常，ReplicationController用于复制pod(即创建pod的多个副本)，并让它们保持运行。
  * 为什么需要服务
    * pod的存在是短暂的，一个pod可能随时消失，或许因为故障，或许是有人删除了pod。
    * 上述情况发生后，消失的pod会被ReplicationController替换为新的pod.
    * 当一个服务被创建时，它会得到一个静态的IP, 在服务的生命周期里这个IP不会变化。客户端通过固定IP地址连接到服务，而不是直接连接pod。
    * 服务会确保其中一个pod接收连接，而不关心pod当前运行在哪里，以及它的IP地址是什么。
  * 水平伸缩应用
    * 设置期望的数量让k8s自己决定要采取哪些操作
    * 请求会随机切换到不同的pod
  * 查看应用运行在哪个节点上
    * kubectl get pods -o wide