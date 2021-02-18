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
### 8.1 通过Downward API传递元数据
* pod创建前不能知道的数据：IP，host，pod名称等
* Downward API允许我们通过环境变量或者文件(在downward API卷中)传递pod元数据。
* 这种方式主要是将在pod定义和状态中取得的数据作为环境变量和文件的值。
#### 8.1.1 了解可用的元数据
* pod的名称
* pod的IP
* pod所在的namespace
* pod所在的节点名称
* pod运行所归属的服务账户名称
* 每个容器请求的CPU和内存使用量
* 每个容器可以使用的CPU和内存限制
* pod标签
* pod注解
#### 8.1.2 通过环境变量暴露元数据
* spec.containers.env.valueFrom.fieldRef.fieldPath
#### 8.1.3 通过downwardAPI v暴露元数据

### 8.2 与Kubernetes API服务器交互
* 集群中pod之外的信息
#### 8.2.1 探究Kubernetes REST API
* kubectl cluster-info
* 通过kubectl proxy访问API服务器

#### 8.2.2 从pod内部与API服务器交互
* 三件事情
  * 确定API服务器的位置
  * 确保是与API服务器进行交互，而不是一个冒名者
  * 通过服务器的认证，否则不能查看任何内容以及进行任何操作
#### 8.2.3 通过ambassador容器简化与API服务器交互
## 9 Deployment: 声明式地升级应用
### 9.1 更新运行在pod内的应用
* pod创建后，不允许直接修改镜像，只能通过删除原有pod并使用新的镜像创建新的pod替换。
* 两种方法
  * 直接删除现有所有pod，然后创建新的pod。
  * 创建新的pod，等待它们运行成功后，再删除旧的pod。
    * 可以先创建所有新的pod，然后一次性删除所有旧pod。
    * 按顺序一一创建新pod替换旧pod。
  * 第一种方法会导致你的程序在一定时间内不可用
  * 第二种方法你的应用需要支持两个版本同时服务。
#### 9.1.1 删除旧pod，使用新pod替换
* 修改ReplicationController的镜像，然后删除旧的pod，它会检测到当前没有pod匹配它的标签选择器，便会创建新的实例。
#### 9.1.1 创建新的pod，再删除旧的pod
* 需要更多的硬件资源，实现稍微复杂
* 从旧版本立即切换到新版本
  * pod通常通过Service来暴露。在运行新版本的pod之前，Service只将流量切换到初始版本的pod。一旦新版本的pod被创建并且运行正常之后，就可以修改服务的标签选择器并将service的流量切换到新的pod。
* 执行滚动升级操作
### 9.2 使用ReplicationController实现自动的滚动升级
#### 9.2.1 运行第一个版本的应用
#### 9.2.2 使用kubectl来执行滚动式升级
* 使用同样的Tag推送更新过的镜像
  * 需要将imagePullPolicy属性设置为Always
  * 如果是latest的tag，默认的imagePullPolicy就是Always, 其他的tag默认为IfNotPresent
#### 9.2.3 为什么kubectl rolling-update 已经过时
* 它会直接修改创建的对象。直接更新pod和ReplicationController并不符合之前的预期，
* kubectl只是执行滚动升级过程中所有这些步骤的客户端。
* 如果中途失去了网络连接，升级过程会被中断，pod和ReplicationController会停留在中间状态。

### 9.3 使用Deployment声明式地升级应用
* Deployment是一种更高级的资源，用于部署应用程序并以声明的方式升级应用。
* 在升级应用时，需要引入额外一组ReplicationController，并协调两个RC，使它们在根据彼此不断修改，而不会造成干扰。Deployment资源就是负责来处理这个问题的。
#### 9.3.1 创建一个Deployment
#### 9.3.2 升级Deployment
* 只需修改Deployment资源中定义的pod template
* 不同的升级策略
  * 默认是滚动更新，RollingUpdate
  * Recreate: 一次性删除所有旧版本pod，然后创建新的pod
* 减慢滚动升级速度
  * minReadySeconds
* 触发滚动升级
  * 修改pod镜像
  * kubectl set image deploy name image
* 修改Deployment或其他资源的不同方式
  * kubectl edit: 使用默认编辑器打开资源配置，修改保存并推出编辑器
  * kubectl patch: 修改单个资源属性
  * kubectl apply: 应用yaml中的新值来修改对象
  * kubectl replace: 替换yaml中定义的对象，之前必须有yaml存在
  * kubectl setimage: 修改镜像
#### 9.3.3 回滚Deployment
* kubectl rollout undo deployment name
* 显示Deployment的滚动升级历史
  * kubectl rollout history deployment name
* 回滚到特定的Deployment版本
  * kubectl rollout undo deployment name --to-revision=1
#### 9.3.4 控制Deployment升级速率
* maxSurge和maxUnavailable
  * maxSurge: 升级时最多能超出Deployment里规定的副本量，可以是百分比或绝对值
  * maxUnavailable: 升级时最多不可用pod的数量，可以是百分比或绝对值


#### 9.3.5 暂停滚动升级
* kubectl rollout pause deployment name
* 一个新的pod会被创建，与此同时所有旧的pod还在运行，服务的一部分请求会被转发到新的pod。这样相当于发布了一个金丝雀版本。
* 恢复滚动升级
  * kubectl rollout resume deployment namespace
#### 9.3.6 阻止出错版本的滚动升级
* minReadySeconds的主要功能是避免部署出错版本的应用，而不是单纯地减慢部署速度。
* minReadySeconds指定新创建的pod至少成功运行多久之后，才能将其视为可用。在pod可用之前，滚动升级的过程不会继续。
## 10 StatefulSet: 部署有状态的多副本应用
### 10.1 复制有状态pod
* ReplicaSet通过一个模版创建多个副本。这些副本除了它们的名字和IP不同之外，没有别的差异。
* 如果pod模版里描述了一个关联到特定持久卷声明的数据卷，那么ReplicaSet的所有副本都将共享这个持久卷声明，也就是绑定到同持久卷声明和持久卷。
#### 10.1.1 运行每个实例都有单独存储的多副本
* 手动创建pod
* 一个pod实例对应一个ReplicaSet
* 使用同一volume里的不同目录
#### 10.1.2 每个pod都提供稳定的标识
* pod随时可能被删掉替换，尽管新的pod使用的是同一volume，但它有全新的主机名和IP。这可能会在一些应用中出现问题。
* 每个pod实例配置单独的Service（不是一个好的解决方案）
### 10.2 了解StatefulSet
* 可以创建一个Statefulset资源替代ReplicaSet来运行这些pod。它是专门定制的一类应用，这类应用中每一个实例都是不可替代的实体，都拥有稳定的名字和状态。
#### 10.2.1 对比Statefulset和ReplicaSet
* ReplicaSet管理的应用都是无状态的。
* Statefulset管理的有状态应用挂掉后，这个pod实例需要重建，但是新的实例必须与被替换掉的实例具有相同的名称、网络标识和状态。
* Statefulset创建的pod副本并不是完全一样的。每个pod都可以拥有一组独立的数据卷，并且它们的名字都是固定有规律的。

#### 10.2.2 提供稳定的网络标识
* 控制服务
  * 对于有状态的pod来说，因为它们都是彼此不同的，通常希望操作的是其中特定的一个。
  * Statefulset通常要求你创建一个用来记录每个pod网络标记的headless service。通过这个是svc，每个pod将独立拥有DNS记录，这样集群里它的伙伴或者客户端可以通过访问主机名找到它。
* 替换消失的pod
  * 当一个statefulset管理的一个pod实例消失后，statefulset会保证重启一个新的pod实例替换它，但是新的pod会拥有与之前pod完全一致的名称和主机名。
* 扩缩容Statefulset
  * 扩容会使用到下一个还没用到的顺序索引值创建一个新的pod实例。
  * 缩容会最先删除最高索引的实例，在有实例不健康的情况下，是不允许缩容操作的。
#### 10.2.3 为每个有状态的实例提供稳定的专属存储
* 在pod模版中添加卷声明模版
  * volume claim template会在创建pod之前创建pvc，并绑定到一个pod实例上
* 持久卷的创建和删除
  * 让你需要释放特定持久卷时，需要手动删除对应的pvc
* 重新挂载持久卷声明到相同pod的新实例上
#### 10.2.4 Statefulset的保障
* 稳定标识和独立存储的影响
### 10.3 使用StatefulSet
#### 10.3.1 创建应用和容器镜像
#### 10.3.2 通过Statefulset部署应用
* 需要创建2个（或3个）不同类型的对象
  * volume
  * Statefulset必需的一个控制Service
  * Statefulset本身
* 创建PV
* 创建控制Service
  * 必须是headless模式，spec.clusterIP: None

#### 10.3.3 使用你的pod

### 10.4 在Statefulset中发现伙伴节点
* 可以通过API服务器通信来获取，但是会违反应用无感知的原则。
* 使用SRV记录
  * 用来指向提供指定服务的服务器的主机名和端口号。K8s通过一个headless service创建SRV记录来指向pod主机名。
#### 10.4.1 通过DNS实现伙伴间彼此发现
## 11 了解Kubernetes机理
### 11.1 了解架构
* Kubernetes集群的两部分
  * Kubernetes控制平面
  * (工作)节点
* 控制平面的组件，用来存储、管理集群状态，但它们不是运行应用的容器
  * etcd分布式持久化存储
  * API服务器
  * 调度器
  * 控制器管理器
* 工作节点的组件
  * Kubernetes DNS服务器
  * 仪表盘
  * Ingress控制器
  * Heapster(容器集群监控)
  * 容器网络接口插件
#### 11.1.1 Kubernetes组件的分布式特性
* 有些组件无须其他组件，可以独立工作
* 检查控制平面组件状态
  * kubectl get componentstatuses
* 组件之间如何通信
  * 只能通过API服务器通信，不能之间通信。
  * API服务器是和etcd通信的唯一组件，其他组件不会直接和etcd通信，而是通过API服务器来修改集群状态。
* 单组件运行多实例
  * 控制平面组件可以被简单地分割到多台服务器上
  * 其每个组件有多个实例
  * etcd和API服务器的多个实例可以同时并行工作
  * 调度器和控制器管理在给定时间内只能有一个实例起作用，其他实例处于待命模式。
* 组件是如何运行得到
  * 直接部署在系统上或者作为pod来运行
  * kubelet是唯一一直作为常规系统组件来运行的组件，它把其他组件作为pod来运行。
#### 11.1.2 Kubernetes如何使用etcd
* K8s里创建的所有对象持续化存储到etcd。
* etcd是一个响应快、分布式、一致的key-value存储。
#### 11.1.3 API服务器做了什么
* 以RESTful API的形式提供了可以查询、修改集群状态的CRUD接口
#### 11.1.4 API服务器如何通知客户端资源变更
#### 11.1.5 了解调度器
* 利用API服务器的监听机制等待创建新的pod，然后给每个新的、没有节点集的pod分配节点。
* 调度器不会命令选中的节点去运行pod。它做的就是通过API服务器更新pod的定义，然后API服务器再去通知Kubelet该pod已被调度过。当目标节点上的Kubelet发现pod被调度到本节点，他就会创建并且运行pod的容器。
* 默认的调度算法
  * 过滤所有节点，找出可分配给pod的可用节点列表。
  * 对可用节点按优先级排序，找出最优节点。如果有多个同等最优，循环平均分配。
* 查找可用节点
  * 节点能否满足pod对硬件资源的请求
  * 节点是否耗尽资源
  * pod是否要求被调度到指定节点，是否是当前节点
  * 节点是否有和pod规格定义里的节点选择器一致的标签？
  * 如果pod要求绑定指定的主机端口，那么这个节点上的这个端口是否已经被占用？
  * 如果pod要求有特定类型的卷，该节点是否能为此pod加载此卷
  * pod能否容忍节点的污点
  * pod是否定义了节点、pod的亲缘性及非亲缘性规则？
* 为pod选择最佳节点
* pod高级调度
  * pod多个副本期望分布到不同节点上
* 使用多个调度器
  * 可以自定义调度器让后在pod特性中设置它用哪个调度器调度
#### 11.1.6 介绍控制器管理器中运行的控制器
#### 11.1.7 Kubelet做了什么
  * 了解Kubelet的工作内容
    * 负责所有运行在工作节点上内容的组件
    * 运行容器存活探针的组件
#### 11.1.8 Kubernetes Service Proxy的作用

#### 11.1.9 介绍Kubernetes插件

≈
#### 11.2.1 了解涉及哪些组件
* 控制器
* 调度器
* Kubelet
#### 11.2.2 事件链
* Deployment控制器生成ReplicaSet
* ReplicaSet控制器创建pod资源
* 调度器分配节点给新创建的pod
* Kubelet运行pod容器
#### 11.2.3 观察集群事件
### 11.3 了解运行中的pod是什么
### 11.4 跨pod网络
#### 11.4.1 网络应该是什么样的
#### 11.4.2 深入了解网络工作原理
#### 11.4.3 引入容器网络接口
### 11.5 服务是如何实现的
#### 11.5.1 引入kube-proxy
#### 11.5.2 kube-proxy如何使用iptables
### 11.6 运行高可用集群
#### 11.6.1 让你的集群变得高可用
#### 11.6.2 让Kubernetes控制平面变得高可用
## 12 Kubernetes API服务器的安全防护
### 12.1 了解认证机制
* API服务器可以配置一到多个认证的插件。API服务器接收到的请求会经过一个认证插件的列表，列表中的每个插件都可以检查这个请求和尝试确定谁在发送这个请求。
* 有几个认证插件是直接可用的。它们用下列方法获取客户端的身份认证
  * 客户端证书
  * 传入在HTTP头中的认证token
  * 基础的HTTP认证
  * 其他
#### 12.1.1 用户和组
* 认证插件会返回已经认证过的用户名和组。Kubernetes不会在任何地方存储这些信息，这些信息被用来验证用户是否被授权执行某个操作。
* 了解用户
  * Kubernetes区分了两种连接到API服务器客户端
    * 真实的用户（人）
    * pod（运行在pod的用户）
  * 用户被管理在外部系统中，例如单点登陆系统SSO
  * pod使用service account机制，该机制被创建和存储在集群中作为ServiceAccount资源。
* 了解组
  * 正常用户和ServiceAccount都可以属于一个或多个组。组可以一次给多个用户授权。
#### 12.1.2 ServiceAccount介绍
* pod通过发送/var/run/secrets/kubernetes.io/serviceaccount/token文件内容来进行身份认证。这个文件通过加密卷挂载进每个容器的文件系统中。
* 每个pod都与一个ServiceAccount相关联，它代表了运行在pod中应用程序的身份证明。
* token文件持有ServiceAccount的认证token。应用程序使用这个token连接API服务器时，身份认证插件会对ServiceAccount进行身份认证，并将ServiceAccount的用户名传回API服务器内部。
* ServiceAccount用户名格式
  * system:serviceaccount:\<namespace>:\<service account name>
* 了解ServiceAccount资源
  * 作用在单个命名空间
  * 每个pod都与一个ServiceAccount相关联，但是多个pod可以使用一个SA
* ServiceAccount如何和授权绑定
  * 在pod的manifest文件中，可以用指定账户名称的方式将一个SA赋值给一个pod。
  * 如果不显示地指定SA名称，pod会使用在当前namespace的默认SA。
  * API服务器通过管理员配置好的系统级别认证插件来获取这些信息。其中一个现成的授权插件是基于角色控制的插件（RBAC）。
#### 12.1.3 创建ServiceAccount
* 每个pod只应该读取它们所需要的东西。
* 创建SA
  * kubectl create serviceaccount foo
* 了解SA上的可挂载密钥
  * 默认情况下pod可以挂载任何密钥
  * 设置只允许挂载SA中列出的密钥
    * SA加入注解：kubernetes.io/enforce-mountable-secrets="true"
#### 12.1.4 将SA分配给pod
  * 在pod定义文件spec.serviceAccountName字段设置SA名称，必须在创建时设置，后续不能被修改。
  * 使用自定义的SA token和API服务器通信

### 12.2 了解认通过角色的权限控制加强集群安全
#### 12.2.1 介绍RBAC授权插件
* Kubernetes API服务器可以配置使用一个授权插件来检查是否允许用户请求得到的动作执行。因为API服务器对外暴露了REST接口，用户可以通过向服务器发送HTTP请求来执行动作，通过在请求中包含认证凭证来进行认证。
* 了解动作
  * 获取pod
  * 创建服务
  * 更新密钥
* RBAC这样的授权插件运行在API服务器中，它会决定一个客户端是否允许在请求的资源上执行请求的动词。
* 了解RBAC插件
  * 将用户决策作为决定用户能否执行操作的关键因素。
  * 主体（一个人，一个SA或者一组用户或一组SA）和一个和等多个角色相关联，每个角色被允许在特定资源上执行特定的动词。
#### 12.2.2 RBAC资源
* RBAC授权规则通过四种资源来进行配置，它们可以分为两个组
  * Role/ClusterRole: 指定了在资源上可以执行哪些动词。
  * RoleBinding/ClusterRoleBinding: 将上述角色绑定到特定用户、组或SA上。
* 角色定义了可以做什么操作，而绑定定义了谁可以做这些操作
* Role & RoleBinding: 命名空间级别资源
* ClusterRole & ClusterRoleBinding: 集群级别资源
#### 12.2.3 使用Role和RoleBinding
* 绑定Role到SA
  * 通过创建一个RoleBinding资源来实现将角色绑定到主体。
* 在角色绑定中使用其他命名空间的SA
  * 修改在foo命名空间中的RoleBinding并添加另一个pod的SA

#### 12.2.4 使用ClusterRole和ClusterRoleBinding
* 使用场景
  * 跨不同命名空间访问资源
  * 访问跨命名空间资源（Node, PV, NS等等）
#### 12.2.5 了解默认的ClusterRole和ClusterRoleBinding
* Kubernetes提供了一组默认的ClusterRole和ClusterRoleBinding，每次API服务器启动时都会更新它们。这保证了在你错误删除了角色和绑定或者Kubernetes的新版本使用了不同得到集群角色和绑定配置时，所有的默认角色和绑定都会被重新创建。
* view, edit, admin and cluster-admin是最重要的角色，他们应该绑定到用户定义pod中得到SA上。
* 用view ClusterRole允许对资源的只读访问。
* 用edit ClusterRole允许对资源的修改。
* 用admin ClusterRole赋予一个命名空间全部的控制权。
* 用cluster-admin ClusterRole得到完全的控制。
* 了解其他默认的CR
  * 以system为前缀
#### 12.2.6 理性地授予授权权限
* 为每个pod创建特定的SA
* 假设你的应用会被入侵
## 13 保障集群内节点和网络安全
### 13.1 在pod中使用宿主节点的Linux命名空间
#### 13.1.1 在pod中使用宿主节点的网络命名空间

#### 13.1.2 绑定宿主节点上的端口而不使用宿主节点的网络命名空间


#### 13.1.3 使用宿主节点的PID与IPC命名空间
### 13.2 配置节点的安全上下文
#### 13.2.1 使用指定用户运行容器
#### 13.2.2 阻止容器以root用户运行
#### 13.2.3 使用特权模式运行pod
#### 13.2.4 为容器单独添加内核功能
#### 13.2.5 在容器中禁用内核功能
#### 13.2.6 阻止对容器根文件系统的写入
#### 13.2.7 容器使用不同用户运行时共享存储卷

### 13.3 限制pod使用安全相关的特性
#### 13.3.1 PodSecurityPolicy资源介绍
#### 13.3.2 了解runAsUser, fsGroup和supplementalGroup策略
#### 13.3.3 配置允许、默认添加、禁止使用的内核功能
#### 13.3.4 限制pod可以使用的存储卷类型
#### 13.3.5 对不同的用户与组分配不同的PodSecurityPolicy
### 13.4 隔离pod的网络
#### 13.4.1 在一个命名空间中启动网络隔离
#### 13.4.2 允许同一命名空间中的部分pod访问一个服务端pod
#### 13.4.3 在不同Kubernetes命名空间之间进行网络隔离
#### 13.4.4 使用CIDR隔离网络
#### 13.4.5 限制pod的对外访问流量
## 14 计算资源管理
### 14.1 为pod中的容器申请资源
* 创建一个pod时，可以指定容器对CPU和内存资源请求量(requests)，以及资源限制量(limits)。
* 不在pod里定义，而是每个容器单独指定。
#### 14.1.1 创建包含资源requests的pod
* spec.containers.resources.requests
#### 14.1.2 资源requests如何影响调度
* 通过资源requests指定了pod对资源需求的最小值。如果节点的未分配资源量小于pod需求量，调度器不会把pod调度到该节点
* 调度器如何判断一个pod是否适合调度到某个节点
  * 调度器不关心各类资源在当前时刻的实际使用量，只关心节点上部署的所有pod的资源申请量之和。
* 调度器如何选择最佳节点
  * 调度器首先会对节点列表进行过滤，排除那些不满足需求的节点，然后根据预先配置的优先级函数对其余节点进行排序。
  * 基于资源请求量的优先级排序
    * LeastRequestedPriority
      * 优先将pod调度到请求量少的节点上
    * MostRequestedPriority
      * 优先将pod调度到请求量多的节点上
      * 在为每个pod提供足够的CPU/内存资源的同时，确保K8s使用尽可能少的节点。在云平台上可以节省开销。
  * 查看节点资源总量
    * kubectl describe nodes
  * 资源不足时pod无法被调度
    * kubectl describe pod
    * kubectl describe node
  * 释放资源让pod正常调度
#### 14.1.3 CPU requests如何影响CPU时间分配
* CPU requests 不仅仅在调度时起作用，他还决定这剩余的CPU时间如何在pod之间分配
* 如果一个容器能跑满CPU，另一个容器该时段空闲，那么前者可以使用整个CPU。当第二个容器需要CPU的时候能获取到，同时第一个容器会被限制回来。

#### 14.1.4 定义和申请自定义资源
### 14.2 限制容器的可用资源
#### 14.2.1 设置容器可使用资源量的硬限制
* CPU是可压缩资源，意味着可以在不对容器内运行的进程产生不利影响的同时，对其使用量进行限制。
* 内存是不可压缩资源，一但系统为进程分配了一块内存，在进程主动释放这块内存之前无法被回收。因此我们要对容器最大内存分配量做限制。
* 创建一个带有资源limits的pod
  * spec.containers.resources.limits
* 可超卖的limits
  * limits不受节点可分配资源量的约束，它们的总和可以超过资源总量的100%
#### 14.2.2 超过limits
* 因为CPU是可压缩资源，所以CPU的limits超限后，进程只会分不到比限额更多的CPU而已。
* 当进程尝试申请比限额更多的内存时会被杀掉（OOMKilled）。如果pod的重启策略为Always或OnFailure，进程将会立即重启。但是如果它继续超限并被杀死，Kubernetes会再次尝试重启，并开始下次重启的间隔时间。
* 这种情况pod处于CrashLoopBackOff状态
  * 每次崩溃等待重启时间以几盒倍数增长，最终收敛在300秒。

#### 14.2.3 容器中的应用如何看待limits
* 在容器内看到的始终是节点的内存，而不是容器本身的内存
* 容器内同样可以看到节点所有的CPU核
* 需要使用Downward API将CPU限额传递到容器并使用这个值。
* 也可以通过cgroup系统直接获取配置得到CPU限制。
  * /sys/fs/cgroup/cpu/cpu.cfs_quota_us
  * /sys/fs/cgroup/cpu/cpu.cfs_period_us
### 14.3 了解pod QoS等级(Quality of Service)
* 当新启动的pod时，其所需的内存超过节点剩余内存资源时，哪个pod应该被杀掉呢，是新启动的还是现有的pod？
* 3种QoS等级
  * BestEffort(Lowest)
  * Burstable
  * Guaranteed(Highest)
#### 14.3.1 定义pod的QoS等级
* 并非来源于独立的字段定义，来源于pod资源requests和limits配置。
* BestEffort
  * 分配给那些没有设置request和limit的pod
  * 最坏情况下它们分配不到任何CPU，内存不足时它们最先被杀死
* Guaranteed
  * CPU和内存都要设置requests和limits
  * 每个容器都需要设置资源量
  * requests和limits必须相等
    * 如果没有设置requests, 默认它和limits相等，所以只需要设置limits就可以有Guaranteed等级
* Burstable
  * requests小于limits

#### 14.3.2 内存不足时哪个容器会被杀死

### 14.4 为命名空间中的pod设置默认的requests和limits
#### 14.4.1 LimitRange资源
#### 14.4.2 LimitRange创建
#### 14.4.3 强行进行限制
#### 14.4.4 应用资源requests和limits默认值

### 14.5 限制命名空间中的可用资源总量
#### 14.5.1 ResourceQuota资源
#### 14.5.2 为持久化存储指定配额
#### 14.5.3 限制可创建对象的个数
#### 14.5.4 为特定pod状态或QoS等级指定配额
### 14.6 监控pod资源使用量
#### 14.6.1 收集、获取实际资源使用情况
#### 14.6.2 保存并分析历史资源的使用统计信息
## 15 自动横向伸缩pod与集群节点
### 15.1 pod的横向自动伸缩
#### 15.1.1 了解自动伸缩过程
* 三个步骤
  * 获取被伸缩资源对象所管理的所有pod度量
  * 计算使度量数值到达或接近指定目标数值所需的pod数量
  * 更新被伸缩资源的replicas字段
#### 15.1.2 基于CPU使用率进行自动伸缩
* 基于CPU使用率创建HPA
* 观察第一个自动伸缩事件
* 触发第一次扩容
* 观察Autoscaler扩容Deployment
* 了解伸缩操作的最大速率
#### 15.1.3 基于内存使用进行自动伸缩
#### 15.1.4 基于其他自定义度量进行伸缩
* 了解Resource度量类型
* 了解Pods度量类型
* 了解Objects度量类型
#### 15.1.5 确定哪些度量适合用于自动伸缩
#### 15.1.6 缩容到0个副本

### 15.2 pod的纵向自动伸缩
#### 15.2.1 自动配置资源请求
#### 15.2.2 修改运行中pod的资源请求
### 15.3 集群节点的横向伸缩
#### 15.3.1 Cluster Autoscaler介绍
* 从云端基础价格请求新节点
* 归还节点
#### 15.3.2 启动Cluster Autoscaler
#### 15.3.3 限制集群缩容时的服务干扰
## 16 高级调度
### 16.1 使用污点和容忍度阻止节点调度到特定节点
#### 16.1.1 介绍污点和容忍度
* 显示节点的污点信息
  * kubectl describe node 
* 显示pod的污点容忍度
  * kubectl describe pod
* 了解污点的效果
  * NoSchedule: 如果pod没有容忍这些污点，pod则不能被调度到包含这些污点的节点上。
  * PreferNoSchedule: NoSchedule的一个宽松版本，表示尽量阻止pod被调度到哦这个节点上，如果没有其他可用节点，还是会调用到这里。
  * NoExecute: 前两者只会在调度期间起作用，NoExecute会影响正在节点上运行的pod，如果节点上添加了NoExecute污点，在节点上没有容忍度的pod会从这个节点上去除。
#### 16.1.2 在节点上添加自定义污点
* kubectl taint node

#### 16.1.3 在pod上添加污点容忍度
* spec.template.spec.tolerations

#### 16.1.4 了解污点和污点容忍度的使用场景
* 在调度时使用污点和容忍度
* 配置节点失效后的pod重新调度最长等待时间

### 16.2 使用节点亲缘性将pod调度到特定节点上
* 对比节点亲缘性和节点选择器
* 检查默认的节点标签
  * failure-domain.beta.kubernetes.io/region: 节点所在的地理位置
  * failure-domain.beta.kubernetes.io/zone: 节点所在的可用性区域
  * kubernetes.io/hostname: 节点主机名
#### 16.2.1 指定强制性节点亲缘性规则
* spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoreDuringExecution.nodeSelectorTerms
* 较长的节点亲缘性属性名的意义
  * requiredDuringScheduling: 为了使pod能调度到该节点上，明确指出了该节点必须包含的标签。
  * IgnoreDuringExecution: 不会影响已经在该节点上运行的pod
* 了解节点选择器条件

#### 16.2.2 调度pod时优先考虑某些节点
* preferredDuringSchedulingIgnoredDuringExecution
* 给节点加上标签
* 指定优先级节点亲缘性规则
* 了解节点优先级如何工作
* 在一个包含两个节点的技巧中部署节点

### 16.3 使用pod亲缘性与非亲缘性对pod进行协同部署
#### 16.3.1 使用pod间亲缘性将多个pod部署在同一个节点上
* spec.template.affinity.podAffinity.preferredDuringSchedulingIgnoredDuringExecution.[topologyKey:kubernetes.io/hostname].labelSelector.matchedLabels.app:backend
  * 要求pod被调度到和其他包含app=backend标签的pod所在的相同节点上
* 部署包含pod情缘性的pod
* 了解调度器如何使用pod亲缘性规则
#### 16.3.2 将pod部署在同一机柜、可用性区域或者地理区域
* 在同一个可用性区域
  * topologyKey: failure-domain.beta.kubernetes.io/zone
* 同一地域
  * topologyKey: failure-domain.beta.kubernetes.io/region
* 了解topologyKey是如何工作的

#### 16.3.3 表达情缘性优先级取代强制需求
#### 16.3.4 利用pod的非亲缘性分开调度pod
* podAntiAffinity
## 17 开发应用的最佳实践
### 17.1 集中一切资源
* 典型Kubernetes应用
  * 在开发者编写的应用manifests中定义
    * Deployment
      * Pod template
        * Container
          * Health probes
          * Env variables
          * Volume mounts
          * Resource requests/limits
      * Volume
        * PersistentVolumeClaim
        * ConfigMap
    * StatefulSet
    * DaemonSet
    * Job
    * CronJob
    * Ingress
    * Service
  * 由集群管理员提前创建
    * Service account
    * Secret
    * Storage Class
    * LimitRange
    * ResourceQuota
  * 在运行时自动创建
    * ReplicaSet
      * pod (label selector)
    * Persistent Volume
    * Endpoint

### 17.2 了解pod的生命周期
* 可以将pod比作只运行单个应用的虚拟机，但pod可能随时被杀死。
#### 17.2.1 应用必须料到会被杀死或者重新调度
* 预料到本地IP和主机名会发生变化
  * 大部分无状态应用都可以处理这种场景同时不会有副作用。
  * 有状态的服务通常不能
    * 使用StatefulSet
    * 不应该依赖IP地址来构建彼此的关系，使用主机名
* 预料到写入磁盘的数据会丢失
* 使用存储卷来跨容器持久化数据
#### 17.2.2 重新调度死亡或部分死亡的pod
* ReplicaSet不关心pod是否死亡，只关心副本数量是否匹配
#### 17.2.3 以固定顺序启动pod
* 了解pod是如何启动的
  * 你可以阻止一个主容器的启动，直到它的预置条件被满足。通过在pod中包含一个叫做init的容器来实现。
* init容器介绍
  * 可以用来初始化pod，通常意味着向容器存储卷中写入数据，然后将这个存储卷挂载到主容器中。
  * 一个pod可以挂载任意数量的init容器。init容器是按顺序执行的，并且仅当最后一个init容器执行完毕才会去启动主容器。
* 将init容器加入pod
  * spec.initContainers
* 处理pod内部依赖的最佳实践
  * 构建一个无依赖应用
  * 使用Readiness探针

#### 17.2.4 增加生命周期钩子
* 两种生命周期钩子
  * 启动后(Post-start)
  * 启动前(Pre-stop)
* 都可以
  * 在容器内部执行一个命令
  * 向一个URL发送HTTP GET请求
* 使用启动后容器生命周期钩子
  * spec.containers[image.lifecycle.postStart.exec.command[""]]
* 使用停止前容器生命周期钩子
* 生命周期钩子是针对容器而不是pod
#### 17.2.5 了解pod的关闭
* pod的关闭是通过API服务器删除pod对象来触发的。当接收到HTTP DELETE请求后，API服务器还没有删除pod对象，而是给pod设置了一个deleteTimeStamp值。拥有这个值的pod就开始停止了。
* 当Kubelet意识到需要终止pod的时候，它开始终止pod中的每个容器。它会给每个容器一定的时间来优雅地停止。这个时间叫作终止宽限期(Termination Grace Period)，每个pod可以单独配置。
* 当终止进程开始之后，计时器就开始计时，按照顺序执行以下事件
  * 执行停止前钩子（如果有），然后等待它执行完毕
  * 向容器的主进程发送SIGTERM信号
  * 等待容器优雅地关闭或者等待终止宽限期超时
  * 如果容器主进程没有优雅地关闭，使用SIGKILL信号强制终止进程
* 指定终止宽限期
  * 终止宽限期可以通过pod中的spec.terminationGracePeriod
* 在应用中合理地处理容器关闭操作
* 将重要的关闭流程替换为专注于关闭流程的pod

### 17.3 确保所有的客户端请求都得到了妥善处理
* 确保所以的客服端请求都得到了妥善处理
#### 17.3.1 在pod启动时避免客户端连接断开
#### 17.3.2 在pod关闭时避免客户端连接断开
* Kubernetes的组件都是运行在不同机器上的不同进程，并不是在一个庞大的单一进程。
* 当API服务器接收到删除pod请求后，它首先修改了etcd中的状态并且把删除事件通知给观察者。其中的两个观察者就是Kubelet和端点控制器
* 问题：在发送终止信号给pod后，pod仍然可以接收客户端请求。
* 解决问题
  * 给pod添加就绪探针
  * 等待足够长的时间让所有的kube-proxy完成它们的工作。
* 妥善关闭一个应用的步骤
  * 等待几秒钟，然后停止接收新的连接
  * 关闭所有没有请求过来的长连接
  * 等待所有请求完成
  * 完全关闭应用

### 17.4 让应用在Kubernetes中方便运行和管理
#### 17.4.1 构建可管理的容器镜像
* 部署全新pod速度快要求镜像足够小且不包含任何无用的东西。
* 如果使用GoLang构建应用，你的镜像除了应用可执行的二进制文件外不需要任何东西。
* 在实践中，最小化构建应用非常难调试，因为缺少像ping, dig, curl等工具。
#### 17.4.2 合理地给镜像打标签，正确地使用ImagePullPolicy
* 使用latest标签会导致你无法回退到之前得到版本。
* 使用latest或者其他可更改镜像的标签，imagePullPolicy必须设为Always, pod每次启动都要去连接镜像中心，并且当镜像中心无法连接到的时候，这个策略会导致pod无法启动。
#### 17.4.3 使用多维度的资源标签
* 资源所属的应用或者为服务的名称
* 应用层级(前端，后段等)
* 运行环境(开发、测试、预发布、生产等)
* 版本号
* 发布类型(稳定版、金丝雀等等)
* 租户
* 分片

#### 17.4.4 通过注解描述每个资源
* 资源至少应该包括一个描述资源的注解和一个描述资源负责人的注解
* 在微服务架构中，pod应该包含一个注解来描述pod所依赖的其他服务名称


#### 17.4.5 给进程终止提供更多的信息
* 使用Kubernetes特性在pod状态中显示出容器终止的原因，让容器中的进程向容器文件系统中指定文件写入一个终止消息。
* 默认路径为/dev/termination-logs，可以通过spec.containers.terminationMessagePath定义

### 17.4.6 处理应用日志
* 应用应该将日志写到标准输出终端而不是文件中。这样可以很容易地通过kubectl logs命令来查看应用日志。
* 如果日志写入了文件系统
  * kubectl exec \<pod> cat \<logfile>
* 将日志或者其他文件复制到容器或者从容器中复制出来
  * kubectl cp foo-pod:/var/log/foo.log foo.log
  * kubectl cp localfile foo-pod:/etc/remotefile
* 使用集中式日志记录
* 处理多行日志输出

### 17.5 开发和测试的最佳实践
#### 17.5.1 开发过程中在k8s之外运行应用
* 连接到后台服务
* 连接到API服务器
* 在开发过程中在容器内部运行应用

#### 17.5.2 在开发过程中使用Minikube
* 将本地文件挂载到Minikube VM然后再挂载到容器中
* 在Minikube VM中使用Docker Daemon来构建镜像
* 在本地构建镜像然后直接复制到Minikube VM中
* 将Minikube和Kubernetes集群结合起来

#### 17.5.3 发布版本和自动部署资源清单
#### 17.5.4 使用Ksonnet作为编写YAML/JSON manifest文件的额外选择

#### 17.5.5 利用持续集成和持续交付

## 18 Kubernetes应用扩展
### 18.1 定义自定义API对象
#### 18.1.1 CustomResourceDefinitions介绍
* 开发者只需向Kubernetes API服务器提交CRD对象，即可定义新的资源类型。
* CRD范例介绍
  * 集群用户不必处理pod、服务以及其他Kubernetes资源，甚至只需要确认网站域名以及网站中的文件(HTML, CSS, PNG等)就能以最简单的方式运行静态网站。
  * 需要一个Git仓库当作这些文件的来源。当用户创建网站资源实例时，你希望Kubernetes创建一个新的web服务器pod，并通过服务将它公开。
* 创建一个CRD对象
* 创建自定义资源实例
* 检索自定义资源实例
* 删除自定义资源实例

#### 18.1.2 使用自定义控制器自动定制资源
* 了解网站控制器的功能
* 将控制器作为pod运行
* 观察运行中的控制器

#### 18.1.3 验证自定义对象
#### 18.1.4 为自定义对象提供自定义API服务器
* API服务器聚合

### 18.2 使用Kubernetes服务目录扩展Kubernetes
#### 18.2.1 服务目录介绍
#### 18.2.2 服务目录API服务器与控制器管理器介绍
#### 18.2.3 Service代理和OpenServiceBroker API
* OpenServiceBroker API
* 在服务目录中注册代理
* 罗列集群中的可用服务
#### 18.2.4 提供服务与使用服务
* 提供服务实例
* 绑定服务实例
* 在客户端pod中使用新创建的Secret
#### 18.2.5 解除绑定与取消配置
#### 18.2.6 服务目录给我们带来了什么

### 18.3 基于Kubernetes搭建的平台
#### 18.3.1 RedHat OpenShift
* OpenShift容器平台中其他可用资源
  * 除了Kubernetes中提供的所有可用API之外，OpenShift还提供了一些额外的API对象
    * Users&Groups
    * Projects
    * Templates
    * Buildconfigs
    * DeploymentConfigs
    * ImageStreams
    * Routes
* 使用构建配置通过源码构建镜像
  * 通过将OpenShift集群指向包含应用程序源代码的Git存储库，用户能在OpenShift集群中快速构建和部署应用程序，不必亲自构建容器镜像。

#### 18.3.2 Deis Workflow与Helm
* Deis Workflow
  * 可以部署到任何现有的Kubernetes集群中，在你运行Workflow时，它会创建一组Service和ReplicationController。
  * 只需通过git push deis master推送更改就可以触发应用程序新版本更新，而剩余的工作都会由Workflow完成。
* 通过Helm部署资源
  * 一个Kubernetes包管理器，由两部分组成
    * helm cli
    * Tiller, Kubernetes集群内pod运行的服务组件
  * 