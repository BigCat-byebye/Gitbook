# Ingress
即将Ingress是从Kubernetes集群外部访问集群内部服务的入口
下面以Nginx Ingress 为例子

Ingress分为2部分，1部分是Ingress Controller，另1部分是Ingress，可以理解为Ingress Controller就是一个nginx，而Ingress就是Nginx的配置文件，每次创建Ingress的时候，Ingress Controller就会自动载入Ingress，从而实现将请求转发到不同的Svc

安装Ingress Controller
``` 
1.添加helm仓库
helm add stable	http://mirror.azure.cn/kubernetes/charts/

2.搜索ingress
helm search ingress

3.安装
helm install stable/ingressmonitorcontroller
# 注意，最好将该chart先pull下载，看看chart内容，是否契合主机
```

为应用创建ingress
```
helm create nginx
# 可以查看到该chart中有ingress部分的定义，修改下即可


