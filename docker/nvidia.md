# Ubuntu
## Uninstall
### Nvidia Drive
```
sudo apt-get --purge remove nvidia*
sudo apt autoremove
```
### Cuda
```
sudo apt-get --purge remove "*cublas*" "cuda*"
sudo apt autoremove
```
## Install
### Nvidia Drive
```
ubuntu-drivers devices
select driver version
apt-get install nvidia-driver-440-server

nvidia-smi 
Mon Dec 21 13:12:14 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 450.80.02    Driver Version: 450.80.02    CUDA Version: 11.0     |
|-------------------------------+----------------------+------------------
```
### Cuda
```
download cuda package
https://developer.nvidia.com/zh-cn/cuda-downloads

cd /cudapackage
sudo sh cuda_9.0.176_384.81_linux.run
choose yes or no depending on the situation
        
echo export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-9.0/lib64 >> ~/.bashrc
echo export PATH=$PATH:/usr/local/cuda-9.0/bin >> ~/.bashrc
echo export CUDA_HOME=$CUDA_HOME:/usr/local/cuda-9.0 >> ~/.bashrc

nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Sat_Aug_25_21:08:01_CDT_2018
Cuda compilation tools, release 10.0, V10.0.130

rm -r /cudapackage
```
### Cudnn
```
download cudnn package
cd /cudnnpackage
tar -xf cudnn-10.0-linux-x64-v7.6.4.38.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn.h
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*

cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
#define CUDNN_MAJOR 7
#define CUDNN_MINOR 6
#define CUDNN_PATCHLEVEL 4
--
#define CUDNN_VERSION (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)

#include "driver_types.h"
```
# Tip
## cuda和nvidia的版本对应关系 
[参考链接](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)
## tensflow与cuda以及cudnn版本对应关系
[参考链接](https://tensorflow.google.cn/install/source)



# docker-GPU安装
## 设置stable存储库和GPG密钥：
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
&& curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
&& curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```   
## 更新软件包清单后，安装NVIDIA运行时软件包（及其依赖项）：
```
sudo apt-get update \&& 
sudo apt-get install -y nvidia-docker2
```
## 修改添加：  /etc/docker/daemon.json
```
{
   "default-runtime": "nvidia",
   "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
      }
   }
}
```
## 重新启动Docker守护程序以完成安装
```
sudo systemctl restart docker
```

# 参考信息
```
ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:02.0/0000:02:00.0 ==
modalias : pci:v000010DEd00001B06sv000010DEsd0000120Fbc03sc00i00
vendor   : NVIDIA Corporation
model    : GP102 [GeForce GTX 1080 Ti]
driver   : nvidia-driver-418-server - distro non-free
driver   : nvidia-driver-450-server - distro non-free
driver   : nvidia-driver-415 - third-party free
driver   : nvidia-driver-440-server - distro non-free
driver   : nvidia-driver-455 - distro non-free recommended
driver   : nvidia-driver-390 - distro non-free
driver   : nvidia-driver-410 - third-party free
driver   : nvidia-driver-450 - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin

nvidia-smi 
Mon Dec 21 13:12:14 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 450.80.02    Driver Version: 450.80.02    CUDA Version: 11.0     |
|-------------------------------+----------------------+------------------


cat /usr/local/cuda/version.txt
CUDA Version 10.0.130

nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Sat_Aug_25_21:08:01_CDT_2018
Cuda compilation tools, release 10.0, V10.0.130


cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2

#define CUDNN_MAJOR 7
#define CUDNN_MINOR 6
#define CUDNN_PATCHLEVEL 4
--
#define CUDNN_VERSION (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)

#include "driver_types.h"
```
## ssh root@172.16.0.18
```
root@yons-Super-Server:~# ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:02.0/0000:02:00.0 ==
modalias : pci:v000010DEd00001B06sv00001458sd0000376Bbc03sc00i00
vendor   : NVIDIA Corporation
model    : GP102 [GeForce GTX 1080 Ti]
driver   : nvidia-driver-430 - third-party free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin

nvidia-smi 
Mon Dec 21 13:19:08 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 430.26       Driver Version: 430.26       CUDA Version: 10.2   




cat /usr/local/cuda/version.txt
CUDA Version 10.0.130

cat /usr/local/cuda/version.txt
CUDA Version 10.0.130


nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Sat_Aug_25_21:08:01_CDT_2018
Cuda compilation tools, release 10.0, V10.0.130

```
## ssh root@172.16.0.15
```
root@zoneyet-zhanting:~# ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:1c.4/0000:03:00.0 ==
modalias : pci:v000010DEd00001B06sv000010DEsd0000120Fbc03sc00i00
vendor   : NVIDIA Corporation
model    : GP102 [GeForce GTX 1080 Ti]
driver   : nvidia-driver-440 - third-party free recommended
driver   : nvidia-driver-390 - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin

nvidia-smi 
Mon Dec 21 13:01:00 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.100      Driver Version: 440.100      CUDA Version: 10.2  


cat /usr/local/cuda/version.txt
CUDA Version 10.0.130

nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Sat_Aug_25_21:08:01_CDT_2018
Cuda compilation tools, release 10.0, V10.0.130




cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
#define CUDNN_MAJOR 7
#define CUDNN_MINOR 6
#define CUDNN_PATCHLEVEL 4
--
#define CUDNN_VERSION (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)
```
