# Kubernetes


## Pod

Pod 的本质，是虚拟机，而容器，是进程。

比如：凡是调度、网络、存储，以及安全相关的属性，基本上是 Pod 级别的。这么去理解。

Pod 的实现需要一个 Infra 容器 作为初始化容器，而其他容器都是通过 Join Network Namespace 的方式 与 Infra 容器 关联在一起。

同一个 Pod 里面的所有用户容器来说，它们的进出流量，也可以认为都是通过 Infra 容器完成的。

> 13-深入剖析Kubernetes 13


NodeSelector：是一个供用户将 Pod 与 Node 进行绑定的字段。

NodeName：为 Pod 完全指定 Node。

HostAliases：定义 Pod 的 Host文件。

shareProcessNamespace：内部所有容器共享 PID Namespace。

hostNetwork、hostIPC、hostPID：共享宿主机资源。

ImagePullPolicy：镜像拉取策略。

Lifecycle：生命周期Hooks。


> 13-深入剖析Kubernetes 14

Projected Volume：投射数据卷，Kubernetes 把数据投射到容器目录上，比如 Secret、ConfigMap、Downward API、ServiceAccountToken。

一旦其对应的 Etcd 里的数据被更新，这些 Volume 里的文件内容，同样也会被更新。这是 kubelet 组件在定时维护这些 Volume。但更新可能会有一定的延时，所以在编写应用程序时，在发起数据库连接的代码处写好重试和超时的逻辑，绝对是个好习惯。

Downward API：让 Pod 里的容器能够直接获取到这个 Pod API 对象本身的信息，而且只能启动之前就确定的信息。

Service Account：Kubernetes 系统内置的一种“服务账户”，它是 Kubernetes 进行权限分配的对象。可以实现 Pod 安装 Client 控制集群。原理就是通过 Projected Volume 得到 crt 认证文件，然后通过 认证文件 去访问。

livenessProbe：健康检查

restartPolicy：Pod 恢复机制，默认是 Always。

readinessProbe：检查结果的成功与否，决定的这个 Pod 是不是能被通过 Service 的方式访问到。

PodPreset：Pod预设置，就不需要开发人员填写那么多字段了。

PodPreset 里定义的内容，只会在 Pod API 对象被创建之前追加在这个对象本身上，而不会影响任何 Pod 的控制器的定义。意思就是 Pod 只能作用于 Pod资源文件，不能改变比如 Deployment。

> 13-深入剖析Kubernetes 15

> 13-深入剖析Kubernetes 16

## 控制器

Kubernetes 通用编排模式：控制循环。

控制循环就是：获得对象X的实际状态，获得对象X的期望状态，进行比较，如果相同不做，如果不同就指定编排动作，达到期望状态，然后循环。

> 13-深入剖析Kubernetes 17

## StatefulSet

StatefulSet 是解决 Deployment 无状态的问题。

StatefulSet 的核心功能，就是通过某种方式记录这些状态，然后在 Pod 被重新创建时，能够为新 Pod 恢复这些状态。

记录这些状态：1、拓扑状态，谁先启动，谁后启动，尤其是网络标识。2、存储状态，每个容器存储的不一样。

首先，StatefulSet 的控制器直接管理的是 Pod。
其次，Kubernetes 通过 Headless Service，为这些有编号的 Pod，在 DNS 服务器中生成带有同样编号的 DNS 记录。
最后，StatefulSet 还为每一个 Pod 分配并创建一个同样编号的 PVC。

> 13-深入剖析Kubernetes 18
> 13-深入剖析Kubernetes 19
> 13-深入剖析Kubernetes 20

## API对象

Kubernetes 使用 组Group、版本Version、资源Resource 来标识一个API对象。

CRD（Custom Resource Definition，自定义API资源）解决了 1.7版本 之前添加资源困难的问题，具体方法：

1. 使用 CustomResourceDefinition 定义API资源的信息。

2. 写代码：定义资源属性。

3. 写控制器。

> 13-深入剖析Kubernetes 24/25


## RBAC

所谓角色（Role），其实就是一组权限规则列表。而我们分配这些权限的方式，就是通过创建 RoleBinding 对象，将被作用者（subject）和权限列表进行绑定。

与之对应的 ClusterRole 和 ClusterRoleBinding，则是 Kubernetes 集群级别的 Role 和 RoleBinding，它们的作用范围不受 Namespace 限制。

> 13-深入剖析Kubernetes 26


### Operator

Operator 的工作原理，实际上是利用了 Kubernetes 的自定义 API 资源（CRD），来描述我们想要部署的“有状态应用”；然后在自定义控制器里，根据自定义 API 对象的变化，来完成具体的部署和运维工作。

> 13-深入剖析Kubernetes 27


## 持久化

Persistent Volume（PV），描述的是持久化存储数据卷，运维人员提供，比如 NFS 类型。

Persistent Volume Claim（PVC），Pod 所希望使用的持久化存储的属性，开发人员提供，比如 存储大小、读写权限。

可以理解：PVC提供接口，PV 提供实现。

Volume Controller：如果 PVC 暂时找不到匹配的 PV，就会启动失败，这个时候运维人员赶紧提供了对应的 PV，这个时候希望能再次进行绑定，PV 和 PVC 一旦绑定，就可以使用，而这个绑定就是 VC 做的，内部有个循环专门监听处理。

绑定：将这个 PV 对象的名字，填在了 PVC 对象的 spec.volumeName 字段上。

容器的Volume：将一个宿主机上的目录，跟一个容器里的目录绑定挂载在一起。而持久化 Volume，就是这个宿主机上的目录，具备“持久性”。

Volume处理机制：
1、Attach，连接到 远程磁盘，本地认为是一个块设备。
2、Mount，挂在到宿主机目录上，然后传递给 CRI容器 挂在到对应目录上。

StorageClass：创建 PV 的模板。因为一个集群 PVC 成千上万，不可能每次让运维人员提交满足条件的 PV，这就是 Dynamic Provisioning。StorageClass 定义了两部分内容，一个是 PV 属性，比如 存储类型、Volume大小等，一个是 PV 需要的存储插件，比如 Ceph 等。

创建完之后，Kubernetes 就能够根据用户提交的 PVC，找到一个对应的 StorageClass，Kubernetes 就会调用该 StorageClass 声明的存储插件，创建出需要的 PV。

如果你的集群已经开启了名叫 DefaultStorageClass 的 Admission Plugin，它就会为 PVC 和 PV 自动添加一个默认的 StorageClass；否则，PVC 的 storageClassName 的值就是“”，这也意味着它只能够跟 storageClassName 也是“”的 PV 进行绑定。

> 13-深入剖析Kubernetes 28

---

PV、PVC 是更好的提供可扩展性。

Local Persistent Volume：适用于对IO性能非常敏感的应用，比如 分布式数据库、分布式文件系统。

Local Persistent Volume 不能使用本地文件磁盘作为存储，因为完全不可控，隔离也没有了，关键是影响系统本身，所以 LPV 的存储介质应该是一个额外挂载的磁盘，而不是主硬盘。

Kubernetes 在调度的时候如何考虑Volume 分布，毕竟不是每个节点都有磁盘。方法就是 nodeAffinity 字段，保证在特定节点上进行调度。

StorageClass 里的 volumeBindingMode=WaitForFirstConsumer 表示延迟绑定，推迟到调度Pod的时候，具体不懂。

> 13-深入剖析Kubernetes 29

---

在 Kubernetes 中，存储插件的开发有两种方式：FlexVolume 和 CSI。

TODO：

> 13-深入剖析Kubernetes 30
> 13-深入剖析Kubernetes 31


## 网络

容器 的网络在自己的  Network Namespace，包括 网卡（Network Interface）、回环设备（Loopback Device）、路由表（Routing Table）和 iptables 规则，也可以直接使用宿主的网络，唯一的好处就是非常好的性能，但缺点就是占用端口，受宿主网络环境的影响。

不同 Network Namespace 的进程 通过 网桥（Bridge）连接，其工作在数据链路层，通过 MAC地址把数据发送到端口上。

Docker 自动创建了 docker0 网桥。容器通过 Veth Pair 这个虚拟设备跨 Namespace 互通，让宿主机充当中间网桥。如果容器访问外部网络，

Veth Pair 设备的特点：创建的时候总是以两张网卡出现，从一个网卡发出的数据包，在另一个网卡出现，哪怕是不同的 Namespace。

容器通过Veth Pair访问外部机器，如果要访问外部机器的容器网络，就需要通过软件构建 Overlay Network（覆盖网络）来转发数据包。

不同宿主容器间的通信问题，称作为 “跨主问题”，基于 Overlay 的思想，社区有很多解决方案选择。

> 13-深入剖析Kubernetes 32

---

Flannel 有三种后端实现，VXLAN、host-gw、UDP（废弃）

UDP 的原理比较简单：使用 TUN设备（内核态和用户态之间传递IP包）把数据包通过软件打包成 UDP 发送到对应节点上，然后用相反的操作用 TUN设备 通过内核态发送到容器网络上。缺点就是内核态和用户态切换太多，性能非常差。

VXLAN，即 Virtual Extensible LAN（虚拟可扩展局域网），是 Linux 内核本身就支持的一种网络虚似化技术，可以完全在内核态实现上述封装和解封装的工作，解决了 UDP方案性能痛点。

VXLAN原理：在现有的三层网络之上，“覆盖”一层虚拟的、由内核 VXLAN 模块负责维护的二层网络，使得连接在这个 VXLAN 二层网络上的“主机”（虚拟机或者容器都可以）之间，可以像在同一个局域网（LAN）里那样自由通信。

为了能够在二层网络上打通“隧道”，VXLAN 会在宿主机上设置一个特殊的网络设备作为“隧道”的两端。这个设备就叫作 VTEP，即：VXLAN Tunnel End Point（虚拟隧道端点）。

> 13-深入剖析Kubernetes 33

---

无论 UDP方案，还是 VXLAN方案，都是通过特殊的设备（TUN 和 VTEP）来处理数据包，所以 K8S 网络插件的作用就是联通各个网络设备，所以把 docker0网桥 换成了 CNI 网桥。

CNI 的设计思想：Kubernetes 在启动 Infra 容器之后，就可以直接调用 CNI 网络插件，为这个 Infra 容器的 Network Namespace，配置符合预期的网络栈。

> 13-深入剖析Kubernetes 34

---

host-gw 利用 “下跳地址”（把对方宿主机当作路由器/网关 Gateway）来实现通信，所以叫 正如其名。这些子网和主机信息，只要保存在 ETCD 中通过 WATCH 动态更新即可。

这种模式免除了额外的封包和解包带来的性能损耗，host-gw 的性能损失大约在 10% 左右，而其他所有基于 VXLAN“隧道”机制的网络方案，性能损失都在 20%~30% 左右。

host-gw 模式必须要求集群宿主机之间是二层连通的，而宿主机之间二层不连通的情况也是广泛存在的。比如宿主机分布在了不同的子网（VLAN）里。

Calico 也提供了类似于 host-gw 的方案，唯一不同并非用 ETCD 记录路由主机信息，而是使用 BGP（Border Gateway Protocol，边界网关协议）

边界网关：一个独立大系统的路由器 保存着 其他大系统路由器的信息，当两个系统间通信的时候，负责把数据包发送给对方，所以叫边界的网关。

BGP：在 边界网关 的基础上，解决各种复杂的对外连接，尤其是大规模的网络。所以 BGP 是在大规模网络中实现节点路由信息共享的一种协议。

Calico 的做法是把所有节点当作边界路由器处理，这样的问题就是，每个路由器相互交换路由信息（Node-to-Node Mesh），复杂度 N^2，超过100个节点压力就很大，这个时候可以指定几个节点专门去学习，然后其余节点去获取信息，把复杂度控制在 N 上，这就是 Route Reflector 模式。

Calico 也是需要二层连通，如果没有可以使用 IP隧道（IP tunnel），也是 IPIP模式，原理就是 把 容器 到 Node2 的IP包，伪装成 Node1 到 Node2 的IP包，可以直接通过三层转发。

Calico IPIP模式 和 Flannel VXLAN 模式 性能差不多，实践中，最后把所有节点放在一个子网里。

在大多数公有云环境下，宿主机（公有云提供的虚拟机）本身往往就是二层连通的，所以这个需求也不强烈。

私有部署：使用一个或多个独立组件负责搜集整个集群里的所有路由信息，然后通过 BGP 协议同步给网关。而我们前面提到，在大规模集群中，Calico 本身就推荐使用 Route Reflector 节点的方式进行组网。所以，这里负责跟宿主机网关进行沟通的独立组件，直接由 Route Reflector 兼任即可。

> 13-深入剖析Kubernetes 35

---

网络隔离：NetworkPolicy

需要 K8S 的 CNI网络插件 支持 NetworkPolicy，目前 Calico、Weave 和 kube-router 等多个项目，但是并不包括 Flannel 项目。

如果想要在使用 Flannel 的同时还使用 NetworkPolicy 的话，你就需要再额外安装一个网络插件，比如 Calico 项目，来负责执行 NetworkPolicy。

Kubernetes 网络插件对 Pod 进行隔离，其实是靠在宿主机上生成 NetworkPolicy 对应的 iptable 规则来实现的。

```go
for dstIP := range 所有被 networkpolicy.spec.podSelector 选中的 Pod 的 IP 地址
  for srcIP := range 所有被 ingress.from.podSelector 选中的 Pod 的 IP 地址
    for port, protocol := range ingress.ports {
      iptables -A KUBE-NWPLCY-CHAIN -s $srcIP -d $dstIP -p $protocol -m $protocol --dport $port -j ACCEPT 
    }
  }
} 
```

### iptables

iptables 是一个操作 Linux 内核 Netfilter 子系统的“界面”。顾名思义，Netfilter 子系统的作用，就是 Linux 内核里挡在“网卡”和“用户态进程”之间的一道“防火墙”。

IP 包“一进一出”的两条路径上，有几个关键的“检查点”，它们正是 Netfilter 设置“防火墙”的地方。在 iptables 中，这些“检查点”被称为：链（Chain）。

流程：进入路由之前，设置了 PREROUTING 检查点，处理完，经过路由，第一种本机处理（进入 INPUT 检查点，继续上上层协议栈流动，进入用户空间处理，处理完，进入路由表进行路由，然后过 OUTPUT 检查点，之后过 POSTROUTING 检查点），第二种直接过 POSTROUTING 检查点（不会经过传输层，包转发，直接过 FORWARD，进入 POSTROUTING 检查点）。

POSTROUTING 检查点是两条路径的最终检查点。

每一个检查点都有一些表，比如 raw、nat、filter 等，作用就是按顺序执行检查动作。

> 13-深入剖析Kubernetes 36

--- 

Service 解决：1、Pod的IP不固定；2、负载均衡。

Service 原理：kube-proxy 组件，加上 iptables 来共同实现的，其中 iptables 实现随机模式。

基于 iptables 的 Service 实现性能不高，制约了Pod数量。

基于 IPVS 的 Service 在 Service 宿主机上设置 ipvs虚拟网卡，添加响应的 Pod虚拟主机，通过 Netfilter 的 NAT 模式进行转发，完全只通过内核，就不需要用户态频繁修改 iptables。所以，非常建议为 kube-proxy 设置–proxy-mode=ipvs 来开启这个功能，它为集群规模提升是非常巨大的。

 Kubernetes 里，/etc/hosts 文件是单独挂载的，这也是为什么 kubelet 能够对 hostname 进行修改并且 Pod 重建后依然有效的原因。

> 13-深入剖析Kubernetes 37

---

从外界连通 Service 与 Service 调试“三板斧”

NodePort：访问到任何一个节点，SNAT操作（不让 Node2 知道 Client，不然 Node2 直接转发给 Client 会报错），通过 iptables 转发。

可以将 Service 的 spec.externalTrafficPolicy 字段设置为 local，这就保证了所有 Pod 通过 Service 收到请求之后，一定可以看到真正的、外部 client 的源地址。但是，流量只能在 Local本地宿主机上的 Pod，就不会垮 Node 转发了，如果选择的 Node 没有响应 Pod，请求会被丢弃。

LoadBalancer：适用于公有云上的 Kubernetes 服务，Kubernetes 会调用 CloudProvider（转接层，来跟公有云本身的 API 进行对接） 在公有云上为你创建一个负载均衡服务，并且把被代理的 Pod 的 IP 地址配置给负载均衡服务做后端。

ExternalName：域名转换，https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#externalname  

理解了 Kubernetes Service 机制的工作原理之后，很多与 Service 相关的问题，其实都可以通过分析 Service 在宿主机上对应的 iptables 规则（或者 IPVS 配置）得到解决。

进一步问题解决，参阅 13-深入剖析Kubernetes 38。

> 13-深入剖析Kubernetes 38

---

Service 与 Ingress

缘起：Service 有个 LoadBalancer 就是让云服务商通过 Cloud Provider 提供负载均衡，如果每个 Service 都有负载均衡服务，成本太高了，所以需要一个内置的全局负载均衡器，通过 URL，就能转发到不同的后端 Service。所以 Ingress服务 出来了。

所谓 Ingress，就是 Service 的“Service”。

所谓 Ingress资源文件，就是一个反向代理的描述文件，用户不需要关心具体实现。

Ingress Controller：Ingress 具体实现，常用的是 Nginx Ingress Controller。

Nginx Ingress Controller 的本质就是 根据 Ingress对象变化 和 代理后端 Service 的变化，自动生成 Nginx 配置文件（/etc/nginx/nginx.conf），然后启动 Nginx 服务。

PS：如果被代理的 Service 对象被更新，nginx-ingress-controller 通过 Nginx Lua 实现Nginx Upstream 的动态配置。

之后，需要一个 Service 把 Ingress Controller 暴露出去，要么公有云的 LoadBalancer，要么 NodePort。

如果匹配不到规则，默认返回 Nginx 的 404页面，或者指定 –default-backend-service 添加一个 Service 专门处理 404 页面。

> 13-深入剖析Kubernetes 39
