环境说明：

1. 本次服务器为国外服务器，可以直接访问google云的，因此格外顺利
服务器系统为centos 7.4

2. 本次安装全程使用的是root权限

3. 已经提前关闭selinux，firewalld以及iptable了，并且实现禁止开机启动了

4. 安装的k8s是1.14版本的

步骤如下：

配置docker，k8s以及kubeadm的yum源
``` shell
1. 配置docker的yum源，建议是用static的版本即可
cd /etc/yum.repos.d/
cat docker-ce.repo
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://download.docker.com/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

2. 配置k8s的yum源
cd /etc/yum.repos.d/
cat kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*

3. 运行yum makecache生成yum源的缓存即可
yum makecache
```

配置k8s的网络插件（本次用的calio）
``` shell
wget https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
x
# 在kubernetes官网可以查看，因为kubeadm是官方推荐的安装k8s的工具，网址如下https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network
```

开启net_bridge模块
``` shell
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
安装docker以及kubectl 等工具
``` shell
yum install -y docker-ce docker-ce-cli
systecmctl enable docker
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
指定cgroup为systemd（这一步不知道是不是必须的，但是我没做这一步会报错，这一步骤应该是可以跳过的）
cd /etc/docker/
cat daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
systemctl daemon-reload
systemctl restart kubelet
systemctl restart docker
```

查看安装k8s所需要的镜像
``` shell
kubeadm config images pull
# 如果是在国内可以kubeadm config images list 查看镜像，然后在国内找一下可以images，提前下载下来，然后重新打上tag即可，如下，所需的images如最后的图所示
```

使用kubeadm init安装k8s集群，（因为只有1台机器，且cpu只有1个，所以直接使用kubeadm init会报错cpu不够2个）
``` shell
kubeadm init --pod-network-cidr=10.1.0.0/16 --service-cidr=10.3.3.0/24 --ignore-preflight-errors=NumCPU
# kubeadm要求，主机的cpu至少2个，且mem不得小于2G，使用该参数可以忽略cpu个数的错误
# 使用pod-network-cidr和service-cidr指定k8s的pod的ip范围和service的ip的范围
```

将k8s的admin.config复制到家目录下，即可实现操作k8s
``` shell
 mkdir -p $HOME/.kube
 cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 chown $(id -u):$(id -g) $HOME/.kube/config
# 请注意，将admin.config复制到远程主机（比如1.2.2.3）上，并且命名为远程主机某个用户家目录下的.kube/config的话，只要该远程主机上的用户可以使用kubectl，那么该用户就具有操作该k8s的权限
```

安装网络插件，并且去除master污点
``` shell
kubectl apply -f *.yaml
# 刚才下载了calico的2个yaml文件，在这里安装网络插件，然后就可以看到node的状态冲not -ready变成了ready了，注意哈，如果没有安装网络插件，node的状态就会一直是not ready，而且coredns也一直都不能运行
kubectl taint node master node-role.kubernetes.io/master:NoSchedule-
# 使用kubeadm安装的时候，默认master节点会有污点，而由于我的集群只有一个node，所以只能吧这个污点去掉，让po的可以在master上运行
```

使用kubectl get pod --all-namespaces查看所有的k8s的pod
``` shell
kubectl get pod --all-namesapces
k8s安装成功后的images和启动后的pod如下
k8s版本信息如下
➜  ~ kubectl version
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.1", GitCommit:"b7394102d6ef778017f2ca4046abbaa23b88c290", GitTreeState:"clean", BuildDate:"2019-04-08T17:11:31Z", GoVersion:"go1.12.1", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.1", GitCommit:"b7394102d6ef778017f2ca4046abbaa23b88c290", GitTreeState:"clean", BuildDate:"2019-04-08T17:02:58Z", GoVersion:"go1.12.1", Compiler:"gc", Platform:"linux/amd64"}

k8s所需要的docker images如下
➜  ~ docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy                v1.14.1             20a2d7035165        5 days ago          82.1MB
k8s.gcr.io/kube-apiserver            v1.14.1             cfaa4ad74c37        5 days ago          210MB
k8s.gcr.io/kube-controller-manager   v1.14.1             efb3887b411d        5 days ago          158MB
k8s.gcr.io/kube-scheduler            v1.14.1             8931473d5bdb        5 days ago          81.6MB
calico/node                          v3.3.6              ce902e610f51        2 weeks ago         75.3MB
calico/cni                           v3.3.6              b8eeeae14aa4        2 weeks ago         75.4MB
k8s.gcr.io/coredns                   1.3.1               eb516548c180        3 months ago        40.3MB
hello-world                          latest              fce289e99eb9        3 months ago        1.84kB
k8s.gcr.io/etcd                      3.3.10              2c4adeb21b4f        4 months ago        258MB
k8s.gcr.io/pause                     3.1                 da86e6ba6ca1        15 months ago       742kB

k8s的pod如下
➜  ~ kubectl get pod --all-namespaces
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   calico-node-2m27k                2/2     Running   0          33m
kube-system   coredns-fb8b8dccf-2jzwg          1/1     Running   0          35m
kube-system   coredns-fb8b8dccf-bq5s7          1/1     Running   0          35m
kube-system   etcd-master                      1/1     Running   0          34m
kube-system   kube-apiserver-master            1/1     Running   0          34m
kube-system   kube-controller-manager-master   1/1     Running   0          34m
kube-system   kube-proxy-kssrg                 1/1     Running   0          35m
kube-system   kube-scheduler-master            1/1     Running   0          34m
```

国内下载images(kubeadm v1.14.1)
``` shell
docker pull mirrorgooglecontainers/coredns:1.3.1
docker pull mirrorgooglecontainers/kube-apiserver:v1.14.1
docker pull mirrorgooglecontainers/kube-controller-manager:v1.14.1
docker pull mirrorgooglecontainers/kube-scheduler:v1.14.1
docker pull mirrorgooglecontainers/kube-proxy:v1.14.1
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd:3.3.10
docker pull coredns/coredns:1.3.1
重新打上tag 即可，tag如下
k8s.gcr.io/kube-apiserver:v1.14.1
k8s.gcr.io/kube-controller-manager:v1.14.1
k8s.gcr.io/kube-scheduler:v1.14.1
k8s.gcr.io/kube-proxy:v1.14.1
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
```
