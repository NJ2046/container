# 虚拟化技术
## 背景
公司电脑大抵是Ubuntu系统,当编写文档的时候,Office软件对于Ubuntu系统兼容并没有Windows系统那样优秀,因此决定采用虚拟化技术,来实现在Linux上安装Windows系统.由于部分项目必须使用Windows平台部署,比如热成像系统,因此采用虚拟化技术来实现Windows系统构建,采用VNC协议进行连接.
## KVM
Kernel-based Virtual Machine的简称，是一个开源的系统虚拟化模块，自Linux 2.6.20之后集成在Linux的各个主要发行版本中。它使用Linux自身的调度器进行管理，所以相对于Xen，其核心源码很少。KVM已成为学术界的主流VMM之一
## Server
```
ssh yons@172.16.0.21
123
```
## Install
首先尝试在Docker里安装,用来做到测试以及隔离.
#### docker
```
docker run -itd -p 20469:20469 --mount type=bind,source=/root/nj/winimage,target=/app --name=kvm  ddy:v3
docker exec -it kvm bash
apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager -y
```

### 开发机
```
apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager


virsh edit win7

virsh start win7

virsh destroy win7

virt-manager

无损添加硬盘
qemu-img resize /kvm/CentOS7.4-Test.qcow2 +100G

```
