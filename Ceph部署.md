安装集群之前，需要注意以下事情
各个节点间的host都必须统一，即使用hostname可以相互ping通
如果使用普通用户安装，admin节点必须可以使用ssh免密登陆到其他所有节点，同时必须具备免密切换到root权限
ceph  ALL=(ALL)  NOPASSWD:ALL

具体安装如下
``` shell
初始化集群（admin节点）
ceph-deploy new ceph_admin
安装集群（admin节点）
ceph-deploy install ceph_admin
# 此命令运行时候，正常情况下会连接外网下载软件包
# 如果是离线环境安装ceph，需要将其用到的软件download下来做成源
# 然后在使用此命令安装的时候，需要加上--no-adjust-repo参数
安装ceph monitor（admin节点）
ceph-deploy mon create-initial
# 安装后使用ceph -s 可以看到如下信息
# mon: 1 daemons, quorun ceph_admin
安装ceph manager（admin节点）
ceph-deploy mgr create ceph_admin
安装ceph osd（使用真实硬盘）·
ceph-deploy osd create --data /dev/vdb ceph_admin
# 安装后，使用ceph -s 可以看到以下信息
# osd: 1 osds: 1 up, 1 in
```
安装ceph osd（使用虚拟硬盘)
![pic_1](https://uploader.shimo.im/f/k0abkrxmZyk0WU2g.png)
安装ceph rgw
ceph-deploy rgw create ceph_admin
安装后，可使用ceph -s看到如下信息
rgw: 1 daemon active

使用ceph -s看到的信息如下
图片: https://uploader.shimo.im/f/OOgwbqhDYcodfcu6.png
![pic_2](https://uploader.shimo.im/f/k0abkrxmZyk0WU2g.png)

创建ceph，磁盘其实就是lvm了，可以看到如下信息
![pic_3](https://uploader.shimo.im/f/ofBKy6IdANI9EoIS.png)
