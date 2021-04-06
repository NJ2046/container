# 概论
由于集群主节点在个人电脑上,不方便管理和使用,现将集群主节点迁移至服务器,并且升级k8s版本.
### 集群简介
1. ssh hello@172.16.1.10  123         主节点
2. ssh yons@172.16.0.21   root12300.  从节点
### 兼容性
1.  [k8s和kubeflow的兼容行列表](https://www.kubeflow.org/docs/started/k8s/overview/)
2.  [k8s和dashboard的兼容行列表](https://github.com/kubernetes/dashboard/releases)
3.  经过比对结论此处使用:1.15
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
3. 添加docker官方的GPG key。什么是GPG，为什么要安装GPG。
4. 安装docker引擎。关键步骤。
5. 验证docker是否安装成功。 
6. 启动docker且设置为开机启动。
### [安装 Kube\*](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
```
1. apt-get install kubelet=1.15.0-00 kubeadm=1.15.0-00 kubectl=1.15.0-00
2. systemctl start kubelet && systemctl enable kubelet
```
出现问题,说找不到指定的安装包
```
cat <<EOF >/etc/apt/sources.list.d/docker-k8s.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable
EOF
```
## 主节点
### 初始化
1. 使用init命令来初始化主节点，其中包含主节点IP地址、拉取镜像的仓库地址、pod的网络地址
```
kubeadm init \
	--apiserver-advertise-address=172.16.1.10 \
	--image-repository registry.aliyuncs.com/google_containers \
	--service-cidr=10.1.0.0/16 \
	--pod-network-cidr=10.244.0.0/16
```
上述命令执行成功会输出（部分）
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube && \
  cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  && \
  chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.1.10:6443 --token 2h4aw9.f4my7fzl9a7n3ym4 \
    --discovery-token-ca-cert-hash sha256:78da3081d89ec9f2b7ca61d132946813291666d7cc667032e6d34a1e878c11cb 
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
kubectl apply -f flannel.yml
```
2. 验证操作
```
kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
k8s-nodex                 Ready    master   6m    v1.15.0
```
3. 出现问题,说拉取镜像失败
```
/etc/docker/daemon.json

 "registry-mirrors" : [
	    	"https://kfwkfulq.mirror.aliyuncs.com",
		"https://2lqq34jg.mirror.aliyuncs.com",
		"https://pee6w651.mirror.aliyuncs.com",
		"https://registry.docker-cn.com",
		"http://hub-mirror.c.163.com",
    		"https://mirror.baidubce.com"
	    ],

写入镜像源
依旧失败,查看日志,发现DNS转化不过来,执行如下
echo nameserver 114.114.114.114 >> /etc/resolv.conf 

```
## 子节点
### 加入主节点
1. 在主节点初始化成功的时候会输出子节点加入主节点的命令和参数，其中包括主节点IP、Token以及TokenHash
```
kubeadm join 172.16.1.10:6443 --token 2h4aw9.f4my7fzl9a7n3ym4 \
    --discovery-token-ca-cert-hash sha256:78da3081d89ec9f2b7ca61d132946813291666d7cc667032e6d34a1e878c11cb 


output
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
2. 验证操作
在主节点执行以下命令，输出子节点信息表示操作成功
```
kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
k8s-nodex                 Ready    master   8d    v1.15.0
k8s-child                 NotReady <none>   6d    v1.15.0
```
3. 出现问题,重启节点就连接不到了
```
感觉是节点上的k8s没有删除清理成功,执行如下
kubectl drain b-node2 --delete-local-data --force --ignore-daemonsets

kubectl delete node b-node2

kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig docker0 down
ip link delete cni0
ip link delete flannel.1
systemctl start docker


kubeadm reset
apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*
apt-get autoremove
rm -rf ~/.kube

apt-get install kubelet=1.16.0-00 kubeadm=1.16.0-00 kubectl=1.16.0-00
正在处理用于 systemd (237-3ubuntu10.38) 的触发器 ...
正在处理用于 man-db (2.8.3-2ubuntu0.1) 的触发器 ...
正在处理用于 ureadahead (0.100.0-21) 的触发器 ...
ureadahead will be reprofiled on next reboot
发现安装的时候提示如上
重启机器
systemctl start kubelet && systemctl enable kubelet

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
## install dashboard
参考目录:k8s->dashboard
## install kubeflow






