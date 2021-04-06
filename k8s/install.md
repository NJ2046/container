# 概论
基于容器技术的分布式架构领先方案,基于Docker的大规模容器化分布式系统解决方案,实现资源管理的自动化,以及跨多个数据中心的资源利用率的最大化,负载均衡器的选型和部署实施问题,复杂的服务治理框架,服务监控和故障处理模块
### 软件的安装方式
1. 传统安装方式:源码、二进制以及依托于安装工具，包括但不限于yum、apt等
2. 基于容器的安装方式:k8s特有的安装方式，以配置文件(yaml or json)为调度资源的控制台，使用kubectl apply -f .yam 完成软件安装
### 需要用的软件
1. docker\<18.03: CRI,容器运行时的一种软件。
2. kube\*: kubelet、kubeadm=、kubectl
3. flannel:CNI,容器网络接口的一种软件
4. [systemd](https://www.cnblogs.com/zwcry/p/9602756.html):linux管理的一套命令, systemctl, jorunalctl</br>
其中docker和kubelet、kubeadm、kubectl必须使用传统安装方式, CNI可以使用传统安装方式也可以使用基于容器的安装方式，推荐使用后者。systmed用来调试软件安装出现的问题、监控软件运行的状态。
### 软件介绍
1. docker。k8s所必须的，k8s里的docker是指符合容器运行时的一种软件，当然还有其它容器运行时软件。理解的话，可以用编译器比较，比如C语言会有多种编译器，常用的是GCC，但也有VC、TC等。相同的C语言语法，用不同编译器，都可以正常编译出可运行文件。k8s虽然是一个解决方案、框架或者说是服务，但从编程语言角度理解，k8s的确定义了自己的DSL, 它以资源为基本单元，定义了一套操作资源的领域专用语言。
2. kubelet、kubeadm、kubectl。k8s提供了几种安装方式命令工具安装和二进制文件安装。这里采用第一种方式安装k8s，其中kubeadm用来初始化集群，kubelet用来在集群中的每个节点启动pod和容器等，kubectl用来与集群通信。建议使用yum或apt来安装，工具版本使用1.14.0，kubeflow对k8s的版本1.14.0支持最好，一般的工具版本指定后，如果没有的指定k8s版本，那么k8s的版本会跟随安装工具kubeadm版本。
3. flannel。负责k8s中pod通信的软件。可以通过传统方式安装，也可以通过基于容器的方式安装，推荐后者。安装完成后，集群内节点将处于ready状态，ifconfig的output中会多一个flannel的网卡信息。
4. systemctl,jorunalctl。如果在安装过程中出现错误，可以使用这两个工具来调试错误。systemclt status serverName，查看服务的运行状态。jorunalctl serverName查看服务的日志
# 操作步骤
安装软件包含主节点和子节点，请依照标题的上下顺序进行安装，有一些软件存在依赖性，请严格按照顺序安装，标题的先后顺序是基于软件依赖性进行的。主&子节点是主和子节点都需要进行的操作，主节点和子节点是分别对应的不同操作。
## 主&子节点
### 前置操作
关闭防火墙、selinux、swapoff
```
systemctl stop firewalld && \
systemctl disable firewalld && \
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config  && \ 
setenforce 0 && \
swapoff -a && \
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab 
```
防火墙：可能会遇到服务不存在的提示，不用理会</br>
selinux：可能会遇到文件不存在的提示，不用理会</br>
### [安装 Docker](https://docs.docker.com/engine/install/)
安装示例为ubuntu系统，centos以及其它OS请参考官方安装文档。
```
1. apt-get purge docker-ce docker-ce-cli containerd.io && rm -rf /var/lib/docker
2. apt-get update &&\
	apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
3. curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
4. apt-get install docker-ce docker-ce-cli containerd.io
5. docker version
6. systemctl start docker && systemctl enable docker
```
1. 安装前检查是否已经安装docker，删除已有的docker。如果你已经装好了可以不用安装。
2. 更新apt-get工具，下载相关软件。没看明白，感觉是自己去下载一些软件，为了后续GPG的事情。
3. 添加docker官方的GPG key。什么是GPG，为什么要安装GPG。GPG是一种软件加密方式，知道的加密方式还是有RSA。安装GPG是为了保证软件的安全可靠性，这一步不是非必须。
4. 安装docker引擎。关键步骤。
5. 验证docker是否安装成功。 
6. 启动docker且设置为开机启动。
### [安装 Kube\*](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
kubeflow推荐使用版本为1.14.0的k8s，因此安装示例使用的是1.14.0。
```
1. apt-get install -y kubelet=1.14.0-00 kubeadm=1.14.0-00 kubectl=1.14.0-00
2. systemctl start kubectl && systemctl enable kubectl
```
## 主节点
### 初始化
1. 使用init命令来初始化主节点，其中包含主节点IP地址、拉取镜像的仓库地址、pod的网络地址
```
kubeadm init \
	--apiserver-advertise-address=172.17.7.152 \
	--image-repository registry.aliyuncs.com/google_containers \
	--service-cidr=10.1.0.0/16 \
	--pod-network-cidr=10.244.0.0/16
```
上述命令执行成功会输出（部分）
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.17.7.152:6443 --token zi78n3.0ro76mwadng8k8jy \
    --discovery-token-ca-cert-hash sha256:b2618e08860356992f14d58a6b52c5add541d2b22173389a8fa8e1e5e6febcfb
```
接着执行如下操作，让当前用户正常使用kube
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
2. 验证操作
```
kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
k8s-nodex                 NotReady master   1m    v1.15.0
```
输出节点名称、状态以及角色说明安装成功
### 配置网络
使用kubectl指令，加载网络配置的yaml文件，已经存储在本地，在static目录中，文件名称为kube-flannel.yml
```
kubectl apply -f kube-flannel.yml
```
2. 验证操作
```
kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
k8s-nodex                 Ready    master   6m    v1.15.0
```
## 子节点
### 加入主节点
1. 在主节点初始化成功的时候会输出子节点加入主节点的命令和参数，其中包括主节点IP、Token以及TokenHash
```
kubeadm join 172.17.7.152:6443 --token zi78n3.0ro76mwadng8k8jy \
     --discovery-token-ca-cert-hash sha256:b2618e08860356992f14d58a6b52c5add541d2b22173389a8fa8e1e5e6febcfb
```
2. 验证操作
在主节点执行以下命令，输出子节点信息表示操作成功
```
kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
k8s-nodex                 Ready    master   8d    v1.15.0
k8s-child                 NotReady <none>   6d    v1.15.0
```
### 配置网络
使用kubectl指令，加载网络配置的yaml文件，已经存储在本地，在static目录中，文件名称为kube-flannel.yml
```
kubectl apply -f kube-flannel.yml
```
2. 验证操作
```
kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
k8s-nodex                 Ready    master   8d    v1.15.0
k8s-child                 Ready    <none>   6d    v1.15.0
```
### 其他
从已有集群中脱离且加入到另一个集群中
```
kubeadm reset
apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*
apt-get autoremove
rm -rf ~/.kube
```
## 经验
1. 文档尽可能查看官方，上述安装并非严格按照官方进行，链接附有官方安装教程
2. 根据日志提示解决问题，百度搜索不到，换stackoverflow，再没有QQ技术论坛求教
3. k8s基于容器，使用etcd做分布式存储，网络插件选你喜欢，但大部分使用了flannel，flannel负责pods间的通讯
4. 常用调试命令:查看服务状态、查看pod描述、查看pod日志、查看pod状态
```
systemctl status kubelet
kubectl describe pod nj2008-0  -n zykj
kubectl logs pod/profiles-deployment-55bbfcc7fb-w2p95 -n kubeflow
kubectl get pod -n kubeflow
```
