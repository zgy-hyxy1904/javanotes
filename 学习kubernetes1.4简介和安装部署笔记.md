# 学习kubernetes1.4简介和安装部署笔记

## 第一天 2019/11/19

### 一、**kubernetes简介**

搭建kubernetes集群有四种方式：

1. kubeadm：生产环境，安装比较简单。（快速部署，看不到里面的架构，出现问题很难解决）
2. 二进制：生产环境，安装相对比较复杂，不适合初学者，但是企业中使用最多。（部署较慢，看的到里面的架构，出现问题容易解决）
3. minikude：不适合生产环境，功能比较少，比较单一，快速起一个。
4. yum：使用的少，不是网上流行的安装方式。

#### 1、k8s简介 

Kubernetes（k8s）是Google开源的容器集群管理系统，是一个完备集群管理能力的分布式系统支撑平台。  

先简单概览下官方master/slave集群架构图： 

![img](https://img-blog.csdn.net/20161207141014537?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemptOTEwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

假设图中是我们的三台“主机”，其中右侧两台是放我们应用的服务器，也就是k8s中的Node，在k8s中通过左侧的Master来管理我们的Node。 

先说Master：kubectl是k8s中的命令行操作工具，类似docker命令；Authentication是k8s的认证，Authorization是k8s的授权；API Server提供了集群管理的API接口和负责集群内功能模块的数据交互和通信；Kubernetes以RESTFul形式开放接口；Scheduler是集群中的调度器，负责Pod在集群中节点的调度分配；controller manager是k8s的管理控制中心；etcd是一个高可用的键值存储系统，在k8s中用于存储集群中所有的资源对象的信息并被监控。 

再说Node：Node是Kubernetes集群中相对Master而言的工作主机，可以是一台物理机、虚拟机或者云服务器；kubelet负责本Node上的Pod创建、修改、监控和删除等全生命周期管理，同时定时上报本Node的状态信息；Proxy实现Service的代理及软件模式的负载均衡；cAdvisor是一个用于监控容器运行状态的开源软件，k8s中默认被集成到kubelet组件中。

#### **2、基本概念和术语** 

Kubernetes中Service、Pod、Master、Node、RC、label等概念都可以看作一种资源对象，通过Kubernetes提供的Kubectl工具或API调用进行操作，并保存在etcd中。

**Node：** 
是Kubernetes集群中相对Master而言的工作主机。Node包含的信息：Node的地址（“主机”ip地址）或者Node ID、Node运行状态、Node系统容量描述、Node可用的系统资源、还包括实例信息等，在Node上运行的服务进程包括Kubelet、Kube-proxy和docker daemon。

**Pod：** 
是Kubernetes的最基本操作单元，Pod在Node上被创建、启动或销毁，通过Yaml或者Json格式的配置文件来定义。一个Pod包含一个或多个容器，Pod中的多个容器应用通常是紧耦合的。创建Pod的原因是由于Docker容器之间的通信受到Docker网络机制的限制，虽然容器之间可以通过link的方式访问，但是通过Pod的概念可以将一个Pod当做一个“虚拟机”，Pod中的容器就是这台“虚拟机”中的所有应用，也就是说可以将Node当做我们的“物理机”，然后在我们的“物理机”中创建一个“虚拟机”，“虚拟机”内当然可以通过localhost实现通信，也方便管理，一个Pod中的应用容器可以看到其他应用程序的进程ID。

**以下是简单的Master、Node、Pod和container的关系图：** 

![img](https://img-blog.csdn.net/20161207114916863?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemptOTEwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

**Lable：** （标签）
是Kubernetes系统中的一个核心概念。Lable以key/value键值对的形式附加到各种资源对象上，如Pod、Service、RC、Node等，并且每个对象可以具有多个Lable，在为“对象”定义好Lable后，其他“对象”就可以使用Lable Selector来定义其作用的对象了。Lable Selector分为基于等式或者基于集合的形式，类似sql中的“=”、“！=”、“in”、“not in”语句，或者js中的name标签。

**Replication Controller（RC）：** 
是Kubernates系统中的核心概念，可以通过yaml或json格式的配置文件定义Pod副本的数量，并且可以手动修改，在Master内，Controller Manager进程通过RC的定义和Node的上的某个程序（如Kubelet或Docker）来完成Pod的创建、监控、启停等操作，如果运行了过多的Pod，RC会停掉一些Pod，如果运行了Pod数过少，则RC会启动相应数量的Pod，如果需要删除所有Pod，可以将Pod的数量修改为0，也就是RC可以控制Kubernetes的扩容缩容。

**Job：** 
job类似RC，只是RC是持续管理pod，而Job是一次性创建后就完成结束，删除job会清理掉该job创建的pod。

**Daemon Set：** 
如果需要所有节点（或部分）节点都运行某一个pod（比如日志、监控）可以创建Daemon Set，当这些节点添加到集群中时pod也会被添加，如果某个节点从集群中删除，则这些节点的pod会被垃圾回收，删除Daemon Set将清理它创建的pod。

**Replica Set（RS）：** 
是下一代“RC”Replica Set和Replication Controller的区别是，它支持基于集合的选择器要求，而RC仅支持基于等式的选择器。虽然Replica Set可以单独存在管理pod，但是它主要的功能是配合Deployments来进行管理pod。

**Deployment：** 
对pod和Replica Set进行声明和更新。典型的实例：创建部署得到Replica Set和Pods；检查部署的状态，以查看其是否成功；更新该部署以重新创建Pod（例如，使用新的镜像）；如果当前部署不稳定，回滚到早期的部署版本；暂停和恢复部署。

**Service：** 
是一个虚拟概念，逻辑上代理后端pod，可以看作一组提供相同服务的Pod对外访问接口，拥有一个唯一的指定的名字，一个虚拟IP或端口号，能够提供某种远程服务能力，可以通过Lable Selector来定义Service作用于哪些Pod。如果外部需要访问，可以对外提供Service的type定义：NodePort端口映射和LoadBalancer自定义负载，这样即使运行在不同Node上面相同Lable的Pod，也可以定义Service的Lable Selector和port让外部通过port来访问任何一个该Lable的Pod。如果一个Service需要对外暴露多个端口，可给每个port定义不同的name来区分。

**kube-proxy：** 
每个工作节点都会运行一个kube-proxy服务进程，通过kube-proxy实现流量从service到pod的转发，kube-proxy还可以实现简单的负载均衡功能。kube-proxy有多种代理模式，kube-proxy在工作节点上为每一个服务创建一个临时端口，service的ip:port过来的流量转发到这个临时端口上，kube-proxy会用内部的负载均衡机制（默认是轮寻）选择一个后端pod，然后建立iptables，把流量导入到某个pod里。Kubernetes 支持两种服务发现模式，分别是环境变量和dns。使用环境变量的时候会在pod创建的时候，服务的ip和port信息以环境变量的形式注入到pod里，比如服务ip地址是192.168.1.1，port是8080，则会把下面一系列环境变量注入到pod里，通过这些环境变量获取服务的ip和port。Kubernetes集群内会内置一个dns服务器，service创建成功后，会在dns服务器里导入一些记录，想要访问某个服务，通过dns服务器解析出对应的ip和port，从而实现服务访问。

**结合Lable概念说明：**  

![img](https://img-blog.csdn.net/20161207115132549?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemptOTEwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

图中每个Pod有两个标签，如果在Service的配置中定义Lable Selector属性为name：tomcat，并且ip地址为192.168.1.216，端口号为8080，那么我们可以通过192.168.1.216:8080同时访问Pod1和Pod3；如果在Service的配置中定义Lable Selector属性为version：1，ip地址为192.168.1.216，端口号为80，那么我们可以通过192.168.1.216:80同时访问Pod1和Pod2。 
Pod的概念让我们方便在同一个Pod中通过localhost加端口号来访问不同的容器，而Service让我们方便在同一个Node或外部通过ip地址加端口号来访问不同的“服务器”。

**Volume：** 
是Pod中能够被多个容器访问的共享目录，类似Docker的Volume，只是Pod的Volume生命周期与Pod相同，于容器不相干，并且一个Pod可以有多个Volume。Kubernetes提供两种Volume类型：EmptyDir初始内容为空，同一个Pod中的容器可以读写EmptyDir中的相同文件，如果Pod从Node上移除EmptyDir中的数据也会永远删除；hostPath是在Pod上挂载宿主机上的文件或目录；还可以使用其他存储设备、网络上的永久磁盘或文件系统等作为Volume。

**Namespace：** 
简单的说namespace的定义是为了限制集群范围内的资源访问，即使是同一个用户也不能跨namespace访问资源。

#### 3、服务组件

- **Master：**

**Etcd**：是一个高可用的键值存储系统，在k8s中用于持续化集群中所有的资源对象并被监控。 
**API server**：提供了资源对象的唯一操作入口，其他组件都必须通过它提供的API来操作资源数据。 
**Controller Manager**：集群内部的管理控制中心，其主要目的是实现Kubernetes集群的故障检测和恢复的自动化工作。 
**Scheduler**：集群中的调度器，将待调度的Pod按照特定的调度算法和调度策略绑定到集群中某个合适的Node上。

- **Node：**

**kubelet**：每个Node上都会启动一个Kubelet服务进程，负责本Node节点上管理监控本Node和本Node上的Pod和容器。 
**Proxy**：实现了Service的代理及软件模式的负载均衡器。

- **创建基本对象流程**

kubernetes中基本的对象为Pod、Replication Controller和Service。





```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubectl
```

### 

## 第二天 2019/11/20

### 二、minikube的安装



#### 1.安装过程及步骤

https://blog.csdn.net/boling_cavalry/article/details/91304127



#### 2.学习minikube的相关操作

##### （1）minikube的操作命令

![1574221889736](C:\Users\大仲马\AppData\Local\Temp\1574221889736.png)

##### （2）kubectl的操作命令

![1574232259947](C:\Users\大仲马\AppData\Local\Temp\1574232259947.png)![1574232284491](C:\Users\大仲马\AppData\Local\Temp\1574232284491.png)

