# 容器常用命令
## docker
```
docker image ls
docker ps
docker run -itd --name=container_name -p f8088:i8088 -v flocal:ilocal --network=network_name image:tag
docker exec -it container_name bash
docker build -t image_name:image_tag .
docker container prune

删除所有镜像
docker rmi $(docker images -q) -f
删除所有容器
docker stop $(docker ps -q) & docker rm $(docker ps -aq)
```
## k8s
#### kubectl
```
systemctl status kubelet
kubectl apply -f kube-flannel.yml
kubectl describe pod nj2008-0  -n zykj
kubectl logs pod/profiles-deployment-55bbfcc7fb-w2p95 -n kubeflow
kubectl get pod -n kubeflow
kubectl get pod --all-namespaces
kubectl get pod -n kube-system -o wide
kubectl delete pod dashboard-metrics-scraper-76679bc5b9-4f8dl -n kubernetes-dashboard
kubectl get nodes
systemctl status kubelet
systemctl start kubelet
systemctl stop kubelet

平滑驱逐到别的可用节点上。
kubectl drain b-node2 --delete-local-data --force --ignore-daemonsets

彻底删除一个节点
kubectl delete node b-node2

> kubeadm reset
> systemctl stop kubelet
> systemctl stop docker
> rm -rf /var/lib/cni/
> rm -rf /var/lib/kubelet/*
> rm -rf /etc/cni/
> ifconfig cni0 down
> ifconfig flannel.1 down
> ifconfig docker0 down
> ip link delete cni0
> ip link delete flannel.1
> systemctl start docker

kubeadm reset
apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*
apt-get autoremove
rm -rf ~/.kube

journalctl -xeu kubelet
```
#### kubeadm
```
kubeadm reset
apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*
apt-get autoremove
rm -rf ~/.kube
```
## kubeflow
```
nohup kubectl port-forward -n istio-system svc/istio-ingressgateway 8088:80 --pod-running-timeout=720h --address=0.0.0.0 >nohup.out 2>&1 &
```
