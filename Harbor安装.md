Harbor的安装可以使用ssl也可以不使用ssl，但是docker由于本身的原因，默认启用https，所以harbor和docker需要做一些设置。

目前，只成功做http方式的，即不使用ssl认证


环境：Centos7
yum源：来源于阿里巴巴开源镜像站，百度设置即可
```
1. 首先安装docker-compose，因为harbor安装需要docker-compose
[root@localhost src]# cat docker-compose-install.sh 
#!/bin/bash
#install zhe docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
#grant privilege
chmod +x /usr/local/bin/docker-compose
#cat version
docker-compose --version
​
# 执行以上docker-compose-install.sh即可，脚本是在github上搜索的官方的
./docker-compose-install.sh
​
2. 安装harbor仓库
# 此harbor安装包为离线安装包，因为harbor在安装的时候，会从网上下载一些image，使用离线包，方便, harbor-offline-installer-v1.5.0.tgz，也是在githua上下载的
​
# 解压并进入目录
tar -xzvf harbor-offline-installer-v1.5.0.tgz 
cd harbor/
# 修改配置文件中的hostname即可，只需要修改这里即可，改为本机IP，注意，一定要替换掉该语句，不要注释后重起一行
# 由于本次试验中，不适用ssl认证，所有无需使用CA证书
vi harbor.cfg 
# 如下文，即可
#[root@localhost harbor]# grep hostname harbor.cfg 
#The IP address or hostname to access admin UI and registry service.
#hostname = 192.168.95.132 
./install.sh 
​
3. 修改docker的配置文件，由于docker默认使用https，所以，如果直接使用docker login则会报错
[root@localhost docker]# cat daemon.json 
{
# 这一句insecure-registries加上即可，后面接harbor仓库IP
  "insecure-registries":["192.168.95.132"],
  "registry-mirrors": ["https://registry.docker-cn.com", "https://docker.mirrors.ustc.edu.cn"], 
  "max-concurrent-downloads": 10,
  "log-driver": "json-file",
  "log-level": "warn",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
    },
  "graph": "/var/lib/docker"
}
systemctl reload docker
​
3. 登陆harbor仓库，并上传镜像
# Harbor仓库默认账号密码为admin和Harbor12345，可在harbor.cfg中查看
docker login http://192.168.95.132
# 输入username和password即可
docker pull busybox
docker images
# 将pull下来的image打上tag，并推送到仓库
docker tag busybox 192.168.95.132/library/busybox:v1
docker push 192.168.95.132/library/busybox:v1


4. 登陆harbor仓库的web端
http://192.168.95.132即可

5.在k8s中使用harbor仓库
# 创建secret，使node可以使用harbor仓库
kubectl create secret docker-registry registry-secret --namespace=default --docker-server=192.168.179.128 --docker-username=admin --docker-password=Harbor12345 --docker-email=admin@admin.com
# 查看secret情况
kubectl get secret

# 编写yaml文件，用来创建httpd
# yaml文件对于格式的要求非常严格，具体语法请百度
[root@localhost ~]# cat httpd.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
  - image: 192.168.179.128/library/httpd:v1
    name: httpd-pod
  imagePullSecrets:
  - name: my-secret
# 根据httpd.yaml文件创建httpd的pod
kubectl create -f httpd.yaml 
# 查看pod的情况，如果正常，应该可以看到是Running的状态
kubectl get pod






