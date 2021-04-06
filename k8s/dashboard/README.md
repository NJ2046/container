# Dashboard
k8s-web部署指南
# 操作步骤
1. 在k8s中创建可视化任务，后面的命令验证是否安装成功
```
kubeclt apply -f dashboard.yaml 
kubectl -n kubernetes-dashboard get pods
kubectl -n kubernetes-dashboard get svc
```
2. 创建NodePort访问方式,命令内两个方法都可以，任选其一
```
kubeclt apply -f recommended.yaml 
kubectl  patch svc kubernetes-dashboard -n kubernetes-dashboard \
-p '{"spec":{"type":"NodePort","ports":[{"port":443,"targetPort":8443,"nodePort":30443}]}}'
```
3. 生成token
```
kubeclt apply -f dashboard_token.yml
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```
5. 访问
```
https://172.16.0.21:30443
选择token输入3步骤生成的token
```
# 主页
https://172.16.0.21:30443/#/overview?namespace=ai-dev
# 后记
1. 参考链接
https://blog.csdn.net/networken/article/details/85607593
2. 更改token时长以及创建集群外浏览器访问
