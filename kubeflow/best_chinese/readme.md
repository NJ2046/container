# KubeFlow
安装KubeFlow并且汉化为中业科技公司
# Install
1. cd setp/one run kubectl apply -f .
2. cd setp/two run kubectl apply -f .
3. cd setp/three run kubectl apply -f .


概述： 
kubeflow高级汉化及安装
步骤：
前提需要安装好k8s的环境，在主节点执行以下操作
1. 下载yaml文件夹、local-path-storage.yaml配置文件
2.  创建持久化卷：
          打开local-path-storage.yaml，按照第103行的路径在主节点上创建相应的路径
           持久化卷：kubectl apply -f local-path-storage.yaml
3. kubectl apply -f kubeflow.yaml
3.  进入yaml文件夹：cd yaml
4.  kubeflow初始化：kubectl  apply  -f .        （因为要下载镜像，安装时间比较长，需等待一小时左右）
5. nohup启动：nohup kubectl port-forward -n istio-system svc/istio-ingressgateway 20470:80 --pod-running-timeout=720h --address=0.0.0.0 >nohup.out 2>&1 &
6. 访问：http://172.16.1.10:20470


# command

kubectl get pod -n kubeflow -o wide
