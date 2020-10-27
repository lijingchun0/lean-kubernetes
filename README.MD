# Kubernetes分享

<!-- GFM-TOC -->
 * [前言](#前言)
 * [初识k8s](#初识k8s)
 * [架构](#架构) 
 * [资源对象](#资源对象)
 * [小结](#小结)
<!-- GFM-TOC -->
## 前言
&emsp;&emsp;随着容器技术的发展，Docker近几年突然崛起，变得炙手可热，已经成为容器技术的事实标准。然而想要在生成环境中成功部署和操作容器的关键是容器编排技术，市场上有各种各样的容器编排工具（如Docker原生的Swarm），其中谷歌公司开发的Kubernetes得到开源社区的全力支持，IBM、惠普、微软、RedHat等业界巨头纷纷加入，Kubernetes已经成为GitHub上的明星开源项目。
<br/>
[CKA认证](https://www.bilibili.com/video/BV1S7411m7vM?p=1)
<div align=center><img src="img/k8smap.jpg"></div>


## 初识k8s
&emsp;&emsp;[Kubernetes](https://kubernetes.io/) 这个名字源于希腊语，是舵手的意思，所以它的Logo既像一张渔网，又像一个罗盘。有意思的是Docker的Logo为驮着集装箱在大海上遨游的鲸鱼，Kubernetes与Docker的关系可见一斑。

<div align=center><img src="img/k8s&docker.png"></div>
<br/>
&emsp;&emsp;Kubernetes（也常称K8s，用8代替8个字符“ubernete”而成的缩写。）是一个全新的基于容器技术的分布式架构方案，它源自Google内部大规模集群管理系统——Borg，也是CNCF（Cloud Native Computing Foundation，今属Linux基金会）最重要的项目之一，旨在让部署容器化的应用简单并且高效。 

&emsp;&emsp;Kubernetes 具备完善的集群管理能力，包括多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和服务发现机制、内建负载均衡器、故障发现和自我修复能力、服务滚动升级和在线扩容、可扩展的资源自动调度机制、多粒度的资源配额管理能力。还提供完善的管理工具，涵盖开发、部署测试、运维监控等各个环节。

## 安装
- [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [kubespray](https://github.com/kubernetes-sigs/kubespray)

## 架构
&emsp;&emsp;Kubernetes使用Go语言开发，集群采用 Master/Node（最初称为Minion，后改名Node） 的结构，Master（主节点）控制整个集群，Node（从节点）为集群提供计算能力。用户可以通过命令行或者Web页面的方式来操作集群。


<br>
<div align=center><img src="img/k8sArchitecture.png" width=650></div>

&emsp;&emsp;Master是Kubernetes集群的大脑，负责公开集群的API，调度部署和管理整个集群。集群至少有一个Master节点，如果在生产环境中要达到高可用，还需要配置Master集群。Master主要包含API Server、Scheduler、Controller三个组件，需要etcd组件来保存整个集群的状态。

- etcd：由CoreOS开发，是一个高可用、强一致性的服务发现存储仓库，为Kubernetes集群提供存储服务，类似于zookeper。

- API Server： kubernetes最重要的核心组件之一，提供资源操作的唯一入口（其他模块通过API Server查询或修改数据，只有API Server才直接操作etcd），并提供认证、授权、访问控制、API注册和发现等机制。

- Scheduler：负责资源的调度，按照预定的调度策略将Pod（k8s中调度的基本单位）调度到相应的Node上。

- Controller：通过API Server来监控整个集群的状态，并确保集群处于预期的工作状态，比如故障检测、自动扩展、滚动更新等。

<br>
<div align=center><img src="img/k8sMaster.png"></div>

&emsp;&emsp;Node是Kubernetes集群的工作节点，可以是物理机也可以是虚拟机。Node需要包含容器、kubelet、kube-proxy等组件。Fluentd用来提供日志收集功能（可选）。

- kubelet：维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理。每个节点上都会运行一个kubelet服务进程，接收并执行Master发来的指令，管理Pod及Pod中的容器。每个kubelet进程会在API Server上注册节点自身信息，定期向Master节点汇报节点的资源使用情况，并通过cAdvisor监控节点和容器的资源。

- kube-proxy： 为 Service 提供集群内部的服务发现和负载均衡，监听 API Server 中 service和endpoint的变化情况，并通过iptables等方式来为服务配置负载均衡。

- pause: 在pod中担任Linux命名空间共享的基础,启用pid命名空间，开启init进程。

- Docker： 每台Node上需要安装Docker来运行镜像，但是Docker不是唯一选择，Kubernetes支持多种容器，比如CoreOS公司的Rkt容器（之前称为Rocket，现更名为Rkt）。

<br>
<div align=center><img src="img/k8sNode.png" width=650></div>

## 资源对象

&emsp;&emsp;API对象是Kubernetes集群中的管理操作单元。集群中的众多技术概念分别对应着API对象，每个API对象都有3大类属性：

- metadata（元数据）：用来标识API对象，包含namespace、name、uid等。

- spec （规范）：描述用户期望达到的理想状态，所有的操作都是声明式（Declarative）的而不是命令式（Imperative），在分布式系统中的好处是稳定，不怕丢操作或运行多次。比如设置期望3个运行Nginx的pod，运行多次也还是一个结果，而给副本数加1的操作就不是声明式的，运行多次结果就错了。

- status（状态）：描述系统当前实际达到的状态，比如期望3个pod，现在实际创建好了2个。


<br>

&emsp;&emsp;在Kubernetes众多的API对象中，Pod是最重要的也是最基础的，是kubernetes中可以创建的最小部署单元。Pod就像是豌豆荚一样，它可以由一个或者多个容器组成，这些容器共享存储、网络和配置项。

&emsp;&emsp;目前Kubernetes中的业务类型可以分为长期伺服型（long-running）、批处理型（batch）、节点后台支撑型（node-daemon）和有状态应用型（stateful application）这四种类型，而这四种类型的业务又可以由不同类型的Pod控制器来完成，分别为：Deployment、Job、DaemonSet和StatefulSet。

- Deployment： 复制控制器（Replication Controller，RC）是集群中最早的保证Pod高可用的API对象，副本集（Replica Set，RS）是它的升级，能支持更多种类的匹配模式。部署(Deployment)又是比RS应用模式更广的API对象，以Kubernetes的发展方向，未来对所有长期伺服型的的业务的管理，都会通过Deployment来管理。

- Service： Deployment保证了Pod的数量，但是没有解决如何访问Pod的问题，一个Pod只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的IP启动一个新的Pod，因此不能以确定的IP和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力，Service可以稳定为用户提供服务。

- Job： 用来控制批处理型任务，Job管理的Pod根据用户的设置把任务成功完成就自动退出了。

- DaemonSet： 后台支撑型服务的核心关注点在集群中的Node，要保证每个Node上都有一个此类Pod运行。比如用来收集日志的Pod。

- StatefulSet： 不同于RC和RS，StatefulSet主要提供有状态的服务，StatefulSet中Pod的名字都是事先确定的，不能更改，每个Pod挂载自己独立的存储，如果一个Pod出现故障，从其他节点启动一个同样名字的Pod，要挂载上原来Pod的存储继续以它的状态提供服务。比如数据库服务MySQL，我们不希望一个Pod故障后，MySQL中的数据即丢失。

## 常用命令

- 自动补全
```shell script
# 安装bash-completion
yum install -y epel-release.noarch
yum install -y bash_completion
# 添加补全脚本
kubectl completion bash >/etc/bash_completion.d/kubectl
```
其他系统[Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

- 查看资源对象
```shell script
# Usage:
kubectl explain RESOURCE [options]
```

- 声明式资源对象管理
```shell script
# Usage:
kubectl apply (-f FILENAME | -k DIRECTORY) [options]
```

- 创建资源
```shell script
kubectl create deployment my-dep --image=busybox        # 创建一个deplpyme
kubectl expose rc nginx --port=80 --target-port=8000    # 创建一个svc，暴露 nginx 这个rc
```

- 更新资源
```shell script
kubectl scale --replicas=3 -f foo.yaml                     # 将foo.yaml中描述的对象扩展为3个
kubectl annotate pods foo description='my frontend'        # 增加description='my frontend'备注,已有保留不覆盖
kubectl label --overwrite pods foo status=unhealthy        # 增加status=unhealthy 标签，已有则覆盖
```

- 删除资源
```shell script
kubectl delete -f xxx.yaml                      # 删除一个配置文件对应的资源对象  
kubectl delete pod,service baz foo              # 删除名字为baz或foo的pod和service  
kubectl delete pods,services -l name=myLabel    # -l 参数可以删除包含指定label的资源对象                            
kubectl delete pod foo --grace-period=0 --force # 强制删除一个pod，在各种原因pod一直terminate不掉的时候很有用
```

- 查看资源状态
```shell script
kubectl get
[(-o|--output=)](TYPE[.VERSION][.GROUP] [NAME | -l label] | TYPE[.VERSION][.GROUP]/NAME ...) [flags] 
[options]
# Examples: 
kubectl get services                          # 列出当前NS中所有service资源
kubectl get pods --all-namespaces             # 列出集群所有NS中所有的Pod
kubectl get pods -o wide                      # -o wide也比较常用，可以显示更多资源信息，比如pod的IP等
kubectl get deployment my-dep                 # 可以直接指定资源名查看
kubectl get deployment my-dep --watch         # --watch 参数可以监控资源的状态，在状态变换时输出。在跟踪服务部署情况时很有用
kubectl get pod my-pod -o yaml                # 查看yaml格式的资源配置，这里包括资实际的status，可以用--export排除
kubectl get pod my-pod -l app=nginx           # 查看所有带有标签app: nginx的pod
```
- 查看资源信息
```shell script
# Usage:
kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | -l label] | TYPE/NAME) [options]
# Examples: 
kubectl describe nodes my-node    # 查看节点my-node的详细信息
kubectl describe pods my-pod      # 查看pod my-pod的详细信息
```

- 日志查看
```shell script
# Usage:
kubectl logs [-f] [-p] (POD | TYPE/NAME) [-c CONTAINER] [options]
# Examples: 
kubectl logs my-pod                              
# 输出一个单容器pod my-pod的日志到标准输出
kubectl logs nginx-78f5d695bd-czm8z -c nginx     
# 输出多容器pod中的某个nginx容器的日志
kubectl logs -l app=nginx                        
# 输出所有包含app-nginx标签的pod日志
kubectl logs -f my-pod                           
# 加上-f参数跟踪日志，类似tail -f
kubectl logs my-pod  -p                          
# 输出该pod的上一个退出的容器实例日志。在pod容器异常退出时很有用
kubectl logs my-pod  --since-time=2018-11-01T15:00:00Z
# 指定时间戳输出日志            
kubectl logs my-pod  --since=1h 
# 指定时间段输出日志，单位s/m/h
```

- 执行命令
```shell script
# 命令作用和参数基本与docker exec一致
# Usage:
kubectl exec POD [-c CONTAINER] -- COMMAND [args...] [options]
# Examples:
kubectl exec my-pod ls                         # 对my-pod执行ls命令
kubectl exec -t -i nginx-78f5d695bd-czm8z bash # 进入pod的shell，并打开伪终端和标准输入
```

- 文件传输
```shell script
# Usage:
kubectl cp <file-spec-src> <file-spec-dest> [options]
# Examples:  
kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir                 # 拷贝宿主机本地文件夹到pod
kubectl cp <some-namespace>/<some-pod>:/tmp/foo /tmp/bar        # 指定namespace的拷贝pod文件到宿主机本地目录
kubectl cp /tmp/foo <some-pod>:/tmp/bar -c <specific-container> # 对于多容器pod，用-c指定容器名
```

-集群信息查看
```shell script
kubectl cluster-info      # 查看master和集群服务的地址
kubectl cluster-info dump # 查看集群详细日志
kubectl version           # 查看Kubernetes集群和客户端版本
```

- 节点管理
```shell script
kubectl cordon my-node       
# 标记 my-node 为 unschedulable,禁止pod被调度过来。注意这时现有的pod还会继续运行，不会被驱逐。
kubectl uncordon my-node 
# 与cordon相反，标记 my-node 为 允许调度。
kubectl drain  my-node
# drain字面意思为排水，实际就是把my-node的pod平滑切换到其他node，同时标记pod为unschedulable，也就是包含了cordon命令。
# 但是直接使用命令一般不会成功，建议在要维护节点时，加上以下参数：
kubectl drain my-node  --ignore-daemonsets  --force  --delete-local-data  
# --ignore-daemonsets 忽略daemonset部署的pod
# --force 直接删除不由workload对象（Deployment、Job等）管理的pod
# --delete-local-data  直接删除挂载有本地目录(empty-dir方式）的pod
```

- k8s中Pod重启方法
```shell script
#有最新的 yaml 文件
kubectl replace --force -f xxxx.yaml
#没有 yaml 文件，但是使用的是 Deployment 对象。
kubectl scale deployment esb-admin --replicas=0 -n {namespace}
kubectl scale deployment esb-admin --replicas=1 -n {namespace}
#由于 Deployment 对象并不是直接操控的 Pod 对象，而是操控的 ReplicaSet 对象，
#而 ReplicaSet 对象就是由副本的数目的定义和Pod 模板组成的。
#所以这条命令分别是将ReplicaSet 的数量 scale 到 0，然后又 scale 到 1，
#那么 Pod 也就重启了。
```

- 允许master节点部署pod
```shell script
kubectl taint nodes --all node-role.kubernetes.io/master-
```



