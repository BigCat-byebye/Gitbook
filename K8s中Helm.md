在k8s中搭建helm以及tiller
``` shell
1. 下载helm
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz

2. 将其解压，安装到PATH路径直接直接使用
tar xzvf htlm-v2.13.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/

3. 查看helm版本，下载对应的tiller镜像
helm version

4. 我查看的version是2.13.1，因此需要提前下载tiller镜像，镜像的最终标签如下
gcr.io/kubernetes-helm/tiller:v2.13.1

5. 安装tiller
helm init --tiller-image gcr.io/kubernetes-helm/tiller:v2.13.1 --skip-refresh
# 需要指定image，否则会去gce上下载，国内没办法翻墙，，，fuck GFW
# 需要使用参数--skip-refresh，否则会去gce上拉取chart的缓存到本地，，，同样，fuck GFW

6， 由于k8s有RBAC的权限控制，因此需要配置权限给tiller，使得tiller的这个pod可以通过k8s的api去控制集群
内容如下：
[root@node1 tmp]# cat tiller-serveraccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system

7. 创建serviceaccout即可
kubectl apply -f tiller-serveraccount.yaml

8. 为tiller设置账号
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

8. 验证
helm repo list即可查看到本地的repo信息
```

helm相关命令
``` shell
查询命令
helm --help

创建chart
helm create one_chart

删除chart
curl -X delete http；//192.168.2.11：Port/api/charts/one_chart/v1.0.0

修改chart，下载
创建一个目录
mkdir helm_dirx
cd helm_dir
helm fetch harbor/ocean 
# 将仓库chart包下载到本地
# 其中harbor为仓库名，ocean为chart名
# 完成的chart包名称为one_chart-1.0.0.tgz，其中one_chart为名，1.0.0为版本号，tgz为固定格式
重新打包chart
helm package one_chart
上传chart到仓库
helm push one_chart harbor
# helm push 为插件，默认的helm，没有push插件

安装helm push插件
1. 复制helm push 插件tar.gz包
cp helm_push.tar.gz /xxx/

2. 输入helm home ，找到helm的home目录
helm home
# 查询得到/root/.helm/目录

3. 进入/root/.helm/目录，在plugins中解压helm_push.tar.gz
cd /$(HELM_HOME)/.helm/plugins/
tar xzvf helm-push.tar.gz 

4. 创建一个目录(helmpush)，将helm-push的二进制和plugin.yaml移入（这里以helm-push为例）

5. 编辑plugin.yaml文件，将command:"$HELM_PLUGIN_DIR/XX/XX"改为
     command: "$HELM_PLUGIN_DIR/二进制名"
   vi plugin.yaml
   
   name: "push"
   version: "0.7.1"
   useage: "xxxxxxxxxxxxxxxxxxx  for usage"
   description: "xxxxxxxxxxxx" 
   ** command: "$HELM_PLUGIN_DIR/helmpush***"
6. 输入helm --help，显示push命令  


查询chart
helm search one_chart

更新本地chart仓库
helm repo update

安装chart到k8s集群
helm install harbor/one_chart --name second_chart
# 安装one_chart到k8s集群，并命名为second_chart

helm仓库常用命令
helm repo --help
看仓库信息
helm repo list

创建仓库
helm repo add repo_name repo_addr(http://Ip:Port)

删除仓库
helm repo remove repo_name
```