## 离线安装rpm包
``` shell
yum isntall xxx --downloadonly --downloaddir=.
## 此方法，最好是在新装的机器上执行，这样的话，所依赖的依赖包才能够下得足够全
```

## 离线安装pip包
``` shell
pip download xxx
```

## pip只下载，不安装，可使用pip show xxx查看包的依赖关系
``` shell
pip install xxx --no-index --find-links=.
# 注意此命令要求在，所在包的当前目录执行
```

## Nmap扫描整个网段主机
``` shell
nmap -sP -PI -PT 192.168.1.0/24
```

## Docker中namespace的原理实验如下
原则上docker也是这个原理，docker创建1个容器，就会启动1个network namespace，但是使用ip netns list 是无法查看的，因为该命令只会查看/var/run/netns/下的namespace，将docker下的namespace软链接到该目录下即可查看到，docker的namespace可以在/proc/self/ns/下看到，多个容器中，此处符号[]中数字一致的，表示处于同一个namespace中

## 为network namespace配置网卡
``` shell
ip netns add ns-test
# 创建一个network namespace
ip netns list
# 查看创建的namespace
ip netns exec ns-test ip addr
# 可以执行exec 在ns-test的这个namespace中执行命令
ip netns exec ns-test bash
# 使用exec可以进入指定的namespace中，使用exit退出即可
ip netns exec ns-test ip link set dev lo up
# 在ns-test中将默认的lo网卡启动，默认的lo网卡默认不会启动，lo网卡必须默认启动，否则本机到本机的信号会ping不通
ip link add veth-a type veth peer name veth-b
# 在宿主机上创建2张虚拟网卡veth-a 和 veth-b
ip link set veth-b netns ns-test
# 将veth-b 添加到ns-test这个namespace中，这样的话veth-a就留在宿主机中了
# 以下需要注意的是，当veth-a和veth-b分别被分配到了宿主机和ns-test的网络中，所以对于veth-a和veth-b的操作需要在对应的网络中操作
ip netns exec ns-test ip addr
# 可以看到该namespace中有veth-b的网卡信息了
ip addr
# 可以看到宿主机中有veth-a的网卡信息
ip addr add 10.0.0.1/24 dev veth-a
# 给veth-a网卡分配IP
ip netns exec ns-test ip addr add 10.0.0.224 dev veth-b
# 给veth-b网卡分配IP
ip link set dev veth-a up
ip netns exec ns-test ip link set dev veth-b up
# 给 veth-a 和 veth-b 分配IP后，启动2个虚拟网卡
ip route
# 在宿主查看路由，查看 veth-a 的路由信息
ip netns exec ns-test ip route
# 在ns-test中查看 veth-b 的路由信息
ip 10.0.0.2
# 在宿主机中查看veth-b，是否可以ping通
ip netns exec ns-test ping 10.0.0.1
# 在ns-test中查看veth-a，是否可以ping通
连接2个namespace
思路：1. 创建2个namespce
     2. 创建2个虚拟veth-a和veth-b
     3. 将veth-a和veth-b分别放入ns1和ns2中
     4. 配置veth-a和veth-b启动
     5. 配置veth-a和veth-b的ip为同一网段
     6. 验证连通
     
ip netns add ns1
ip netns add ns2
# 创建2个namespace
ip link add veth-a type veth peer name veth-b
# 创建2个虚拟网卡
ip link set veth-a netns ns1
ip link set veth-b netns ns2
# 将2个网卡分别放入不同的namespace中
ip netns exec ns1 ip link set veth-a up
ip netns exec ns2 ip link set veth-b up
# 分别启动2个namespace中的2个网卡
ip netns exec ns1 ip addr add 10.0.0.1/24 dev veth-a
ip netns exec ns2 ip addr add 10.0.0.2/24 dev veth-b
# 分别给2个namespace中的2个虚拟网卡配置IP
ip netns exec ns1 ping 10.0.0.2
ip netns exec ns2 ping 10.0.0.1
```
验证效果
拓扑图如下，图4-1
![pic_1](https://uploader.shimo.im/f/9ITvES8UJMUGiCDY.png)


## docker的配置文件详解
``` shell
平时使用的话，关注部分参数即可
registory-mirrors，insecure-regitories，default-runtime
{
    "authorization-plugins": [],//访问授权插件
    "data-root": "",//docker数据持久化存储的根目录
    "dns": [],//DNS服务器
    "dns-opts": [],//DNS配置选项，如端口等
    "dns-search": [],//DNS搜索域名
    "exec-opts": [],//执行选项
    "exec-root": "",//执行状态的文件的根目录
    "experimental": false,//是否开启试验性特性
    "storage-driver": "",//存储驱动器
    "storage-opts": [],//存储选项
    "labels": [],//键值对式标记docker元数据
    "live-restore": true,//dockerd挂掉是否保活容器（避免了docker服务异常而造成容器退出）
    "log-driver": "",//容器日志的驱动器
    "log-opts": {},//容器日志的选项
    "mtu": 0,//设置容器网络MTU（最大传输单元）
    "pidfile": "",//daemon PID文件的位置
    "cluster-store": "",//集群存储系统的URL
    "cluster-store-opts": {},//配置集群存储
    "cluster-advertise": "",//对外的地址名称
    "max-concurrent-downloads": 3,//设置每个pull进程的最大并发
    "max-concurrent-uploads": 5,//设置每个push进程的最大并发
    "default-shm-size": "64M",//设置默认共享内存的大小
    "shutdown-timeout": 15,//设置关闭的超时时限(who?)
    "debug": true,//开启调试模式
    "hosts": [],//监听地址(?)
    "log-level": "",//日志级别
    "tls": true,//开启传输层安全协议TLS
    "tlsverify": true,//开启输层安全协议并验证远程地址
    "tlscacert": "",//CA签名文件路径
    "tlscert": "",//TLS证书文件路径
    "tlskey": "",//TLS密钥文件路径
    "swarm-default-advertise-addr": "",//swarm对外地址
    "api-cors-header": "",//设置CORS（跨域资源共享-Cross-origin resource sharing）头
    "selinux-enabled": false,//开启selinux(用户、进程、应用、文件的强制访问控制)
    "userns-remap": "",//给用户命名空间设置 用户/组
    "group": "",//docker所在组
    "cgroup-parent": "",//设置所有容器的cgroup的父类(?)
    "default-ulimits": {},//设置所有容器的ulimit
    "init": false,//容器执行初始化，来转发信号或控制(reap)进程
    "init-path": "/usr/libexec/docker-init",//docker-init文件的路径
    "ipv6": false,//开启IPV6网络
    "iptables": false,//开启防火墙规则
    "ip-forward": false,//开启net.ipv4.ip_forward
    "ip-masq": false,//开启ip掩蔽(IP封包通过路由器或防火墙时重写源IP地址或目的IP地址的技术)
    "userland-proxy": false,//用户空间代理
    "userland-proxy-path": "/usr/libexec/docker-proxy",//用户空间代理路径
    "ip": "0.0.0.0",//默认IP
    "bridge": "",//将容器依附(attach)到桥接网络上的桥标识
    "bip": "",//指定桥接ip
    "fixed-cidr": "",//(ipv4)子网划分，即限制ip地址分配范围，用以控制容器所属网段实现容器间(同一主机或不同主机间)的网络访问
    "fixed-cidr-v6": "",//（ipv6）子网划分
    "default-gateway": "",//默认网关
    "default-gateway-v6": "",//默认ipv6网关
    "icc": false,//容器间通信
    "raw-logs": false,//原始日志(无颜色、全时间戳)
    "allow-nondistributable-artifacts": [],//不对外分发的产品提交的registry仓库
    "registry-mirrors": [],//registry仓库镜像
    "seccomp-profile": "",//seccomp配置文件
    "insecure-registries": [],//非https的registry地址
    "no-new-privileges": false,//不需要赋予新的权限
    "default-runtime": "runc",//OCI联盟(The Open Container Initiative)默认运行时环境
    "oom-score-adjust": -500,//内存溢出被杀死的优先级(-1000~1000)
    "node-generic-resources": ["NVIDIA-GPU=UUID1", "NVIDIA-GPU=UUID2"],//对外公布的资源节点
    "runtimes": {//运行时
        "cc-runtime": {
            "path": "/usr/bin/cc-runtime"
        },
        "custom": {
            "path": "/usr/local/bin/my-runc-replacement",
            "runtimeArgs": [
                "--debug"
            ]
        }
    }
}
```

自建namespace的一些使用技巧
``` shell
1. 查看命名空间中的设备是否可以转移
yum install ethtool
ethtool -k br0
# 如果查看到netns-local: on 则说明该设备br0不可以转移，如果为off，则可以转移

2. 查看namespace中的veth的对端设备
ip netns exec ns1 ethtool -S veth1 
NIC statistics:
     peer_ifindex: 24 
# 对端设备序列号为24，然后去另一个namespace中使用同样的命令查找序列号为24的设备，则这2个设备为一队
3. 给设备增加网桥
yum install bridget-utils
brctl addbr test
# 创建网桥test
brctl addif test ethx
# 将物理网卡关联到网桥test
ifconfig ethx 0.0.0.0
# 设定ethx网卡为0.0.0.0，即相当于取消ethx的ip
ifconfig test 1.2.3.4
# 设定网桥test的ip为1.2.3.4
```
