## 1 Kubernetes介绍
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

## 2 开始使用Kubernetes和Docker
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

## 3 pod：运行于k8s中的容器
  * 为何需要pod
    * 多个容器比单容器多进程好
  * 了解pod
    * 由于不能将多个进程聚集在单独容器中，我们需要另一种更高级的结构来将容器绑定到一起，并将它们作为一个单元进行管理，这就是pod的根本原理
    * 同一pod中容器之间的部分隔离
      * 容器之间是彼此完全隔离的，但我们期望的是隔离容器组，而不是单个容器，并让每个容器组内的容器共享一些资源。
      * k8s通过配置docker来让一个pod内的所有容器共享相同的Linux命名空间。
      * pod中的所有容器共享相同的主机名和网络接口。
      * 这些容器也都在相同的IPC命名空间下运行，能够通过IPC进行通信。
      * 共享相同的PID命名空间，默认不激活
      * 涉及到文件系统时，每个容器的文件系统与其他容器完全隔离。
    * 介绍平坦pod间网络
      * k8s集群中所有的pod都在同一个共享网络地址空间中，这意味着pod之间可以通过其它pod的IP地址来互相访问
    * 通过pod合理管理容器
      * 将多层应用分散到多个pod中
      * 基于扩缩考虑分割到多个pod中
  * 以YAML或者JSON描述文件创建pod
    * metadata: 名称、命名空间、标签和关于该容器的其它信息
    * spec: 包含pod内容的实际说明，例如pod的容器、volume和其它数据
    * status: 包含运行中的pod当前信息，例如pod所处的条件、每个容器的描述和状态，以及内部IP和其它基本信息
  * 使用标签组织pod
    * 介绍标签
      * 可以组织pod和其它kubernetes资源。
      * 标签是可以附加到资源的任意键值对，用于选择具有该确切标签的资源
      * 创建pod时指定标签：在metadata下面labels使用key: value表示
      * kubectl get pods --show-labels
    * 通过标签选择器列出pod子集
      * 包含（或不包含）使用特定键的标签
      * 包含特定键值的标签
      * 包含特定键的标签，但其值与我们指定的不同
  * 使用标签和选择器来约束pod调度
    * 使用场景：工作节点的硬件设施不同，需要把某些需求特定硬件的app部署到对应节点
    * 使用标签来分类工作节点
      * kubectl label node node-1 gpu=true
    * 将pod调度到特定节点
      * spec.nodeSelector.gpu: "true"
    * 调度到一个特定节点
      * 通过主机名label
      * 如果该主机不可用，将造成pod不可调度
      * 所以应该使用标签选择器来调度
  * 注解pod
    * 和标签一样，也是键值对
    * 不能用注解来分组
    * 注解可以容纳更多信息，主要用于工具使用
  * 使用命名空间对资源进行分组
    * 将对象分割成完全独立且不重叠的组
    * 了解对命名空间的需求
      * 可以将包含大量组件的复杂系统拆分为更小的不同组
      * 命名空间之间是否提供网络隔离取决于kubernetes所使用的网络解决方案
  * 停止和移除pod
    * 在删除pod的过程中，实际上我们在指示k8s终止该pod中的所有容器，k8s向进程发送一个SIGTERM信号并等待一定秒数（默认30s），使其正常关闭。如果没有及时关闭，则通过SIGKILL终止该进程。
    * 通过标签选择器删除pod
    * 通过删除整个命名空间来删除pod
    * 删除所有pod，保留ns
      * kubectl delete po --all
    * 删除命名空间中的所有资源
      * kubectl delete all --all


## 4 副本机制和其它控制器：部署托管的pod
### 4.1 保持pod健康
* 只要将pod调度到某个节点，该节点上得到Kubelet就会运行pod的容器，从此只要该pod存在，就会保持运行。如果该容器的主进程崩溃，Kubelet将重启容器。如果应用中有个导致它每隔一段时间就崩溃的Bug, Kubernetes会自动重启应用程序。
* 即使进程没有崩溃，有时应用程序也会停止正常工作，例如Java的OOM，或者有无限循环和死锁，进程会一直运行。
* 为确保应用在这类情况下可以重启，必须从外部检查其运行状况。
#### 4.1.1 介绍存活探针
* Kubernetes可以使用liveness probe检查容器是否还在运行。如果探测失败，Kubernetes将定期执行探针并重启容器。
* 三种探测容器的机制
  * HTTP GET: 如果返回错误或者无响应，会被认为失败，容器将被重新启动
  * TCP套接字： 尝试与容器指定端口建立TCP连接
  * Exec探针：在容器内执行任意命令，如果返回状态码非0，就认为失败
  
#### 4.1.2 HTTP存活探针
* yaml: spec.containers.livenessProbe.httpGet
  
#### 4.1.3 使用存活探针
* 获取容器崩溃日志：kubectl logs mypod --previous
* 通过kubectl describe来了解为什么重启容器
* 当容器被强行终止时，会创建一个全新的容器，而不是重启原来的容器

#### 4.1.4 配置存活探针的附加属性
* delay: 容器启动后多长时间开始探测
* timeout: 必须在多长时间内返回响应
* period: 间隔多长时间探测一次
* initialDelaySeconds: 初始延时
* 退出码
  * 137: 进程被外部信号终止(SIGKILL, 128 + 9)
  * 143(SIGTERM, 128 + 15)

#### 4.1.5 创建有效的存活探针
* 对于在生产环境运行的pod，一定要定义一个存活探针。没有探针的话，Kubernetes无法直到你的应用是否还活着。
* 存活探针应该检查什么
  * 将探针配置为请求特定的URL路径（/health）
  * 让应用从内部运行的所有重要组件执行状态检查
  * 确保/health路径不需要认证
  * 一定要检查应用内部，没有任何外部因素影响。例如前端因为数据库挂掉返回失败，重启前端不会有任何作用。
* 保持探针轻量
  * 如果运行Java程序，确保使用Http探针，因为使用Exec探针启动全新JVM非常耗时
* 无须在探针中实现重试循环
  * 探针的失败阈值是可配置的，并且通常在容器被终止之前探针必须失败多次。
  * 即使将阈值设为1，Kubernetes为了确认一次探测失败，会尝试若干次。
* 存活探针小结
  * 容器崩溃或探测失败后，该节点上的kubelet会重启容器保持运行，主节点的Kubernetes Control Plane组件不会参与此过程。
  * 如果节点本身崩溃，无法执行任何操作，所以需要ReplicationController或类似机制管理pod

### 4.2 了解ReplicationController
* ReplicationController是一种K8s资源，可以确保pod始终保持运行状态。如果pod因为任何原因消失，ReplicationController会注意到并创建替代pod。
#### 4.2.1 ReplicationController的操作
* ReplicationController的操作会持续监控正在运行的pod列表，并保证相应“类型”的pod数目与期望相符，少增多删。
* ReplicationController的三个部分
  * label selector: 确定ReplicationController的作用域里有哪些pod。
  * replica count: 应运行的pod数量。
  * pod template: 拥有创建新的pod副本。
* 更改控制器的标签选择器或pod模版的效果
  * 对现有pod没有影响
  * 更改标签选择器会使现有pod脱离ReplicationController的管理范围
* 使用ReplicationController的好处
  * 确保pods持续运行
  * 集群节点发生故障时，它将为故障节点上运行的所有受它管理的pod创建替代副本
  * 能轻松实现pod的水平伸缩
  * pod的实例永远不会被安置到另一个节点，相反，ReplicationController会创建一个全新的pod，它与正在替换的实例无关。
#### 4.2.2 创建一个ReplicationController

### 4.3 使用ReplicaSet代替ReplicationController
* 新一代的ReplicationController, 用来完全代替ReplicationController
* 通常不会直接创建它，而是创建在更高层级的Deployment资源

#### 4.3.1 比较ReplicaSet和ReplicationController
* 行为完全相同，但是ReplicaSet的标签选择器功能更强
  * 可匹配多个标签
  * 可匹配不符的标签

### 4.4 使用DaemonSet在每个节点上运行一个pod
* 用例：在每个节点上部署一个日志搜集器和监控器
#### 4.4.2 使用DaemonSet在特定节点上运行一个pod
* 通过pod模版中的nodeSelector
### 4.5 运行执行单个任务的pod
* 运行完成工作后就终止任务的情况
#### 4.5.1 Job资源
* pod内部进程成功结束时，不重启容器。一旦任务完成，pod就被认为处于完成状态。
* 在发生节点故障时，会按照replicaSet的模式安排到其他节点。
* 如果进程本身异常退出，可以将Job配置为重启容器
#### 4.5.2 定义Job资源
* kind: Job
* spec.template.spec.restartPolicy: onFailure

### 4.6 安排Job定期运行或者将来运行一次
#### 4.6.1 CronJob
* kind: CronJob
* spec.schedule: "0,15,30,45 * * * *"

## 5 服务：让客户端发现pod并与之通信
* 为什么需要服务
  * pod是短暂的
  * 客户端不能提前知道提供服务的pod的IP地址
  * 水平伸缩意味着多个pod提供相同的服务
### 5.1 介绍服务
* 一种为一组功能相同的pod提供单一不变的接入点资源。
* 客户端不必知道pod的地址
  
#### 5.1.1 创建服务
* 通过kubectl expose
* 通过yaml
* 从集群内部测试服务
  * pod的ip是集群内部地址，只能在集群内部访问
  * 创建一个pod，它将请求发送到服务的集群IP并记录相应。可以通过pod日志检查响应。
  * 使用ssh远程登陆到一个k8s节点上使用curl。
  * 通过kubectl exec在已存在的pod中执行curl。
* 在运行的容器中远程执行命令
  * kubectl exec mypod -- curl -s http://localhost:8080/health
  * 双横杠--代表kubectl命令项的结束。之后是pod内部需要执行的命令
* 配置服务上的会话亲和性
  * 多次执行同样的命令，每次会执行在随机的pod上
  * 特定客户端的请求每次都指向同一个pod：设置spec.sessionAffinity: ClientIP
* 同一服务暴露多个端口
  * spec.ports 配置多个
* 使用命名的端口
  * 在服务中的port可以使用pod定义的containerPort的name
* 在pod中运行shell
  * kubectl exec -it mypod -- bash
#### 5.1.2 服务发现
* 通过环境变量发现服务
  * kubectl exec mypod env
* 通过DNS发现服务
* 通过FQDN发现服务


#### 5.2 连接集群外部的服务
#### 5.2.1 endpoint
* 服务并不和pod直接相连，它们中间有endpoint
* endpoint暴露一个服务的IP和端口列表
#### 5.2.2 手动配置服务的endpoint
* 创建没有选择器的服务
  * spec里面没有选择器
* 为没有选择器的服务创建Endpoint资源

#### 5.2.3 为外部服务创建别名

#### 5.3 将服务暴露给外部客户端
* NodePort
  * 集群节点在节点本身上打开一个端口，并将在该端口上接收到的流量重定向到基础服务。
  * 该服务仅在内部集群IP和端口上才可访问，但也可以通过所有节点上的专用端口访问
* LoadBalancer
  * NodePort的一种扩展，使得服务可以通过一个专用的负载均衡器来访问，这是由Kubernetes中正在运行的云基础设施提供的。
  * LoadBalancer将流量重定向到跨所有节点的节点端口。
  * 客户端通过负载均衡器的IP连接到服务
* 创建一个Ingress资源
  * 一个完全不同的机制，通过一个IP地址公开多个服务，它运行在HTTP层（网络协议第7层）上，因此可以提供比工作在第四层的服务更多的功能。

#### 5.3.1 使用NodePort类型的服务
#### 5.3.2 使用LoadBalancer类型的服务
* 通过负载均衡器连接服务
  * 会话亲和性和Web浏览器
    * 每次浏览器访问会遇到同一个pod，即使sessionAffinity设置为None
    * 浏览器使用keep-alive连接，并通过单个连接发送所有请求，而curl每次会打开一个新的连接
  
#### 5.3.3 了解外部连接的特性
* 了解防止不必要的网络跳数
  * 随机选择的pod不一定在service所在节点上，需要额外的网络跳转
  * 在服务中设置spec.externalTrafficPolicy: Local
  * 这样服务会选择本地的pod，如果没有本地pod，连接会被挂起，需确保本地至少有一个pod
* 记住客户端IP是不记录的
  * 后端的pod无法看到实际客户端的ip

### 5.4 通过Ingress暴露服务
* 为什么需要Ingress
  * 每个LB需要自己的负载均衡器，以及独立的公网地址，而Ingress只需要一个公网IP就能为许多服务提供访问。
  * Ingress会更加请求的主机名和路径决定请求的转发服务
* Ingress Controller是必不可少的
  * 只有Ingress Controller在集群中运行，Ingress才能正常工作。

#### 5.4.1 创建Ingress资源

#### 5.4.2 通过Ingress访问服务
* 获取Ingress的IP地址
* 确保Ingress中配置的Host指向Ingress的IP地址
* 通过Ingress访问pod
* 了解Ingress的工作原理
  * 客户端首先对域名进行DNS查找，DNS服务返回了Ingress控制器的IP.
  * 客户端项Ingress控制器发送Http请求，并在Host请求头中指定该域名。
  * 控制器从该头部确定客户端尝试访问哪个服务，通过与该服务关联的Endpoint对象查看pod IP, 并将客户端的请求转发给其中一个pod。
  * Ingress Controller不会将请求转发给该服务，只用它来选择一个pod。

#### 5.4.3 通过同一个Ingress访问多个服务
* 将不同的服务映射到相同主机的不同路径
  * spec.host.http.paths下设置多个path
* 将不同的服务映射到不同主机上
  * spec.rules下设置多个host

#### 5.4.4 配置Ingress处理TLS传输以支持HTTPS
* 为Ingress创建TLS认证
  * 需要将证书和私钥附加到Ingress，这两个必须资源存储在Secret资源中，然后在Ingress manifest中引用它

### 5.5 pod就绪后发出信号
* 如果pod没有准备好，不希望服务立刻转发请求给它

#### 5.5.1 介绍就绪探针
* 就绪探测会定期调用，并确定特定的pod是否接收客户端的请求。
* 类型
  * Exec
  * HTTP GET 
  * TCP socket
* 了解就绪探针的操作
  * 启动容器时，可以为Kubernetes配置一个等待时间，经过等待时间后才可以执行第一次准备就绪检查。
  * 之后它会周期性的探测，根据结果采取行动。
  * 与存活探针不同，如果检测不通过，不会终止或重启。

#### 5.5.2 向pod添加就绪探针
* spec.template.spec.containers.readinessProbe

#### 5.5.3 了解就绪探针的实际作用
* 务必定义就绪探针
  * 如果没有将就绪探针添加到pod中，它们计划会立即成为服务端点。如果应用启动比较慢，请求会被转发到未启动完成的pod，返回连接被拒绝的错误。
* 不需要将停止pod的逻辑纳入就绪探针中
  * 只要删除该pod, k8s会从所有服务中移除该pod
### 5.6 使用headless服务来发现独立的pod
* 客户端需要连接到所有的pod，或者后端的pod需要连接到所有其它pod
* Kubernetes运行通过DNS查找发现pod IP
#### 5.6.1 创建headless服务
* 将spec.clusterIP设为None

### 5.7 排查服务故障
* 如果无法通过服务访问pod
  * 确保从集群内连接到服务的集群IP，而不是外部。
  * 不要通过ping服务IP来判断，服务的集群IP是虚拟IP，无法ping通。
  * 如果以及定义了就绪探针，确保它返回成功。
  * 确认某个pod是服务的一部分，使用kubectl get endpoints来检查。
  * 如果通过FQDN或其中一部分来访问，检查是否可以用集群IP访问成功。
  * 检查是否连接到服务的公开端口，而不是目标端口。
  * 尝试直接连接到pod IP以确认pod正在接收正确端口上的连接。
  * 如果甚至无法通过pod IP来访问，请确保应用不是仅绑定到本地主机。
## 6 卷：将磁盘挂载到容器
* 在某些场景下，我们可能希望新的容器可以在之前容器结束的位置继续运行，比如在物理机上重启进程。可能不需要整个文件系统被持久化，但又希望能保存实际数据的目录。
### 6.1 介绍卷（volume）
* Kubernetes的volume是pod的一个组成部分，因此像容器一样在pod规范中定义。
* volume不是独立的Kubernetes对象，不能单独创建或删除。
* pod中所有容器都可以使用volume，但必须先将它挂载在每个需要访问它的容器中。

#### 6.1.1 volume的应用示例
#### 6.1.2 volume的类型
* emptyDir: 用于存储临时数据的简单空目录。
* hostPath: 用于将目录从工作节点的文件系统挂载到pod中。
* gitRepo: 通过检出Git仓库的内容来初始化volume。
* nfs: 挂载到pod的NFS共享volume。
* configMap, secret, downwardAPI: 用于将Kubernetes部分资源和集群信息公开给pod的特殊volume。
* persistentVolumeClaim: 一种使用预置或者动态配置的持久存储类型。
* awsElasticBlockStore, azureDisk, gcePersistentDisk: 用于挂载云服务商提供的特定存储类型。
* cinder, cephfs, iscsi, flocker, glusterfs, quobyte, rbd, flexVolume, vsphere-Volume, photoPersistentDisk, scaleIO: 用于挂载其它类型的网络存储。

### 6.2 通过volume在容器之间共享数据
#### 6.2.1 使用emptyDir卷
* 从一个空目录开始，运行在pod内的应用可以写入它需要的任何文件。
* 因为volume的生命周期和pod关联，删除pod后volume的内容会消失。
* 可用于同一个pod中的多个容器共享数据。
* 也可以被单个容器用于将数据临时写入磁盘，例如超大数据集的排序。

#### 6.2.2 使用Git仓库作为存储卷
* 基本上也是一个emptyDir volume, 它通过克隆Git仓库并在pod启动时（创建容器之前），检出特定的版本来填充数据。
* 并不能和repo保持同步，但如果这个pod由ReplicaSet管理，删除这个pod后创建的新pod包括最新版本。

### 6.3 访问工作节点文件系统上的文件
* 大多数的pod应该忽略它们的主机节点，因此它们不应该访问节点文件系统上的任何文件。
* 某些系统级别的pod，通常由DeamonSet管理，确实需要读取节点的文件或者使用节点的文件系统。通过hostPath volume可以实现

#### 6.3.1 介绍hostPath volume
* 指向文件系统上的特定文件或目录。
* 在同一个节点上运行并在其hostPath卷中使用相同路径的pod可以看到相同的文件。
* hostPath是持久化的，不像emptyDir或Git类型会随着pod的删除而消失。
* 会使pod对节点规划很敏感，如果数据库pod被安排到另一个节点，会找不到数据。

#### 6.3.2 检查使用hostPath卷的系统pod

### 6.4 使用持久化存储
* 如果存储数据的pod可以随意分配到任意节点，需要借助某种类型的网络存储。
* 使用NFS volume: 集群运行在一组自有服务器上时。

### 6.5 从底层存储解耦pod
* 理想的情况是，在Kubernetes上部署应用程序的开发人员不需要知道底层使用的是哪种存储技术。
#### 6.5.1 PersistentVolume & PersistentVolumeClaim
* 开发人员无须像它们的pod添加特定类型的volume，而是由集群管理员设置底层存储，然后通过Kubernetes API服务器创建PV并注册。
* 当用户需要在其pod中使用持久化存储时，他们首先创建PersistentVolumeClaim清单，指定所需要的最低容量和访问模式。然后提交到Kubernetes API服务器，Kubernetes将找到可匹配的持久卷并将其绑定到持久卷声明。
#### 6.5.2 创建PersistentVolume
* 容量需求
* 单节点或多节点
* 只读或可写
* 如何处理被删除的绑定
* 实际存储类型、位置和其它属性
* 不属于任何ns

#### 6.5.3 创建PersistentVolumeClaim
* PVC状态
  * RWO: ReadWriteOnce
  * ROW: ReadOnlyMany
  * RWX: ReadWriteMany
  
#### 6.5.4 在pod中使用PersistentVolumeClaim

#### 6.5.5 PV和PVC的好处
* 对于开发人员更简单
* 不依赖特定技术设施

#### 6.5.6 回收PV
* 手动回收
  * 设置为Retain，删除和重新创建PV
* 自动回收
  * Recycle: 删除volume的内容并可再被用于PVC
  * Delete: 删除底层存储。

### 6.6 持久卷的动态卷配置
#### 6.1.1 通过StorageClass资源定义可用存储类型
#### 6.1.2 请求PersistentVolumeClaim的存储

## 7 ConfigMap和Secret: 配置应用程序
### 7.1 配置容器化应用程序
* 无论是否使用ConfigMap, 以下方法都可以用作配置应用
  * 向容器命令行传递参数
  * 为每个容器设置自定义环境变量
  * 通过特殊类型的volume将配置文件挂载到文件中
### 7.2 向容器传递命令行参数
* Kubernetes可在pod的容器中定义并覆盖命令以满足运行不同的可执行程序。
#### 7.2.1 在Docker中定义命令与参数
* 了解ENTRYPOINT与CMD
  * ENTRYPOINT定义容器启动时被调用的可执行程序
  * CMD指定传递给ENTRYPOINT的参数
  * 正确的做法是借助ENTRYPOINT指令，仅仅用CMD指定所需的默认参数，这样景象可以直接使用docker run \<image> 运行，无须添加参数
* shell与exec形式的区别
  * shell形式: ENTRYPOINT node app.js
  * exec形式: ENTRYPOINT ["node", "app.js"]
  * 区别在于指定的命令是否在shell中被调用
  * shell进程往往是多余的，因此通常可以直接采用exec形式。

#### 7.2.2 在Kubernetes中覆盖命令和参数
* 镜像的ENTRYPOINT和CMD均可被覆盖，只需在容器定义中设置属性command和args的值。
* spec.containers.command/args

### 7.3 为容器设置环境变量
* 容器化应用通常会使用环境变量作为配置源
* Kubernetes允许为pod中的每个容器都指定自定义的环境变量集合
#### 7.3.1 在容器中定义中指定环境变量
* spec.containers.env.-name
* spec.containers.env.value
* Java: System.getenv()
* Python: os.environ[]

#### 7.3.2 在环境变量中引用其他环境变量
* 使用$(VAR)语法

#### 7.3.3 hard coded环境变量的不足
* 为了能在多个环境复用pod的定义，需要将配置从pod定义描述解耦出来。
* 使用ConfigMap资源，用valueFrom代替value字段

### 7.4 利用ConfigMap解耦配置
#### 7.4.1 ConfigMap介绍
* 本质上就是一个Key/Value pair
* app无须直接读取ConfigMap，甚至不需要知道其是否存在。映射的内容通过环境变量或者volume形式传递给容器，而并非直接传递给容器。
* 也可以通过Kubernetes Rest API按需直接读取，但除非需求如此，应尽可能使你的应用保持对Kubernetes的无感知。
* 不同环境(dev/test/prod ns)可以使用同名ConfigMap

#### 7.4.2 ConfigMap创建
* kubectl create cm
* 使用yaml
* 从文件内容创建
* 从文件夹创建
* 合并不同的创建方式

#### 7.4.3 给容器传递ConfigMap条目作为环境变量
* spec.containers.env.valueFrom.configMapKeyRef.name/key
* 在pod中引用不存在的ConfigMap
  * K8s会正常调度pod并尝试运行所有的容器，但是引用不存在的ConfigMap的容器会启动失败。如果之后创建了正确的ConfigMap，失败容器会自动启动，无须重新创建pod。
* 可以标记对ConfigMap的引用是可选的
  * spec.containers.env.valueFrom.configMapKeyRef.optional: true

#### 7.4.4 一次性传递ConfigMap的所有条目作为环境变量
* spec.containers.envFrom.-prefix

#### 7.4.5 传递ConfigMap条目作为命令行参数
* 在pod.spec.containers.args中无法直接引用ConfigMap的条目。
* 可以利用ConfigMap条目初始化某个环境变量，然后再在参数字段中引用
* 在pod.spec.containers.args: ["$(ConfigMapKey)"]

#### 7.4.6 使用ConfigMap volume将目录暴露为文件
* pod.spec.containers.volumeMounts
* pod.spec.volumes
* 仅挂载指定条目到特定文件
  * pod.spec.containers.volumeMounts.subPath

#### 7.4.7 更新配置且不重启应用
* 将ConfigMap暴露为volume可以达到配置热更新的效果。
* ConfigMap更新后，volume中引用它的所有文件也会相应更新。

### 7.5 使用Secret给容器传递敏感数据
#### 7.5.1 介绍Secret
* 使用方法和ConfigMap相同
* Kubernetes通过仅将Secret分发到需要的pod所在节点来保障其安全性
* 只存在于内存中，永不写入物理存储

#### 7.5.2 介绍默认Token Secret
#### 7.5.3 创建Secret
#### 7.5.4 对比ConfigMap和Secret
* Secret的条目内容会被Base64格式编码，ConfigMap直接以文本展示
* Secret的大小上限为1M
* Secret通过stringData字段设置条目的纯文本值
* 在pod中读取Secret会自动解码

#### 7.5.5 在pod中使用Secret 
## 8 从应用访问pod元数据以及其他资源

## 9 Deployment: 声明式地升级应用

## 10 StatefulSet: 部署有状态的多副本应用

## 11 了解Kubernetes机理

## 12 Kubernetes API服务器的安全防护

## 13 保障集群内节点和网络安全

## 14 计算资源管理

## 15 自动横向伸缩pod与集群节点

## 16 高级调度

## 17 开发应用的最佳实践

## 18 Kubernetes应用扩展