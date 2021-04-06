# 汉化
## 汉化映射表
|ui|pod|contains|path|type|remote-image|
|---|---|---|---|---|---|
|主-home|centraldashboard|4656fe0aabdd|/app/dist/public/app.bundle.js|汉化|registry.cn-hangzhou.aliyuncs.com/nhj\_images/kubeflow-images-public:v20190823-v0.6.0-rc.0-69-gcb7dab59|
|notebook服务-jupyter|jupyter-web-app-deployment|18e94dda5adf|/app/kubeflow\_jupyter/default/static|汉化|registry.cn-hangzhou.aliyuncs.com/nhj\_images/jupyter-web-app1:9419d4d|
|超参数训练系统-katib|katib-ui|cfa77004b6fc|/app/build/static/js/main.70203756.chunk.js|汉化|docker push registry.cn-hangzhou.aliyuncs.com/nhj\_images/katib-ui1:v0.6.0-rc.0|
|pipeline|ml-pipeline-ui|k8s\_ml-pipeline-ui\_ml-pipeline-ui|/client/static/js/|汉化|registry.cn-hangzhou.aliyuncs.com/nhj\_images/frontend1:0.1.23|
|查看模型-arifacts|metadata-ui|k8s\_metadata-ui\_metadata-ui|/client/static/js/|汉化|registry.cn-hangzhou.aliyuncs.com/nhj\_images/metadata-ui:v0.1.8|
|home-ui-title|centraldashboard|k8s\_centraldashboard\_centraldashboard|/app/dist/public/index.html/|个性化|修改title标签|
|home-ui-title-logo|centraldashboard|k8s\_centraldashboard\_centraldashboard|/app/dist/public/assets/|个性化|覆盖favicon.ico|
|home-ui-logo|centraldashboard|k8s\_centraldashboard\_centraldashboard|/app/dist/public/app.bundle.js/|个性化|Kebeflow Logo，替换js l 变量内容|
## 操作说明
1. 根据pod或者contains找到汉化所在的位置。kubeflow具有自动迁移功能，也就是节点宕机的时候，调度服务会在集群找找到正常工作的节点，将其资源迁移到正常节点中去。根据pod名字，找到容器所在的节点中，然后到达容器几点，进入容器中替换汉化文件，达到汉化目的。
2. 根据path找到文件所在目录，直接替换或者运行bash文件。
## 使用命令
```
kubectl get pod -n kubeflow|grep pod_name
kubectl describe pod nj2008-0  -n zykj
docker ps |gerp container_name
docker exec -it container_id /bin/sh
docker cp file container_id:/path
```
