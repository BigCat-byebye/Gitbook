下载kubernets-dashboard.yaml文件
``` shell
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
kubectl apply -f kubernetes-dashboard.yaml

#注意哈，由于该kubernetes-dashboard.yaml默认是使用的clster和port，因此只有在集群内才能访问到，而我的机器是阿里云的，，，，所有需要将该dashboard在外网访问，因此我修改了这个kubernetes-dashboard.yaml文件，在其中添加了nodeport，修改部分如下
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort     #增加type为nodeport
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30003   #增加nodeport端口，其余均未修改
  selector:
    k8s-app: kubernetes-dashboard
十分不建议将dashboard暴露在公网，非常危险，一旦被黑，整个集群就都没了，慎重
```

为dashboard创建一个user，（本例中的user拥有所有权限，很危险）
``` shell
1. 创建1个service Account
cat /tmp/adminuser-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system 

2. 创建1个ClusterRoleBinding
cat /tmp/adminuser-clusterrolebingding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
# NOTE: apiVersion of ClusterRoleBinding resource may differ between Kubernetes versions. Prior to Kubernetes v1.8 the apiVersion was rbac.authorization.k8s.io/v1beta1.  
# 翻译哈：可能因为版本的问题，apiVersion需要修改哈
  
3. 创建Service Account和ClusterRoleBinding
kubectl apply -f /tmp/adminuser-account.yaml
kubectl apply -f /tmp/adminuser-clusterrolebingding.yaml

5. 查看刚刚创建的service Account的token
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

6. 登陆dashboard的ip+port，选择token登陆，输入的来的token就ok了
首先声明哈：参考文档来资源github，如下
https://github.com/kubernetes/dashboard/wiki/Creating-sample-user