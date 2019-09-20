Ceph集群最少有1个Ceph Monitor和2个OSD守护进程，运行Ceph文件系统客户端时，必须要有元数据服务器（Metadata Server)

Ceph OSDs : Ceph OSD守护进程（Ceph OSD），负责数据存储复制等以及向Ceph Monitor提供监控信息

Monitors ： 监视器，提供监控信息以及可视化图标，使用

ceph-deploy只能1台主机上安装1个mon，否则会挂掉

MDSs ： Ceph元数据服务器（MDS）为Ceph存储元数据，

rbd操作
``` shell
rados lspools
# 显示当前系统中的pool

rados rmpool 
# 默认情况下，删除pool会报错mon allow pool delete is set to false
# 需要修改ceph配置文件
# ceph --show-config | grep mon_allow_poo_delete
# 可查看到权限，如何修改？touch一个文件，然后覆盖配置文件，然后重启mon和mgr进程
# 创建文件，vim ceph.conf
# mon_allow_pool_delete = true
# 覆盖配置文件，ceph-deploy --overwrite-conf admin ceph-object-01 ceph-object-02 ceph-object-03
# 重启进程，
# systemctl restart ceph-mgr@ceph-object-01.service
# systemctl restart ceph-mon@ceph-object-01.service
# systemctl restart ceph-mgr@ceph-object-02.service
# systemctl restart ceph-mon@ceph-object-02.service
# systemctl restart ceph-mgr@ceph-object-03.service
# systemctl restart ceph-mon@ceph-object-03.service

rados purge pool_name --yes-i-really-really-mean-it
# 删除pool中的rbd_image,而不删除pool本身

rbd ls pool_name
# 显示pool中的rbd_image

rbd info pool_name/rbd_image_name
# 查看pool中的rbd_image信息
```

配置块设备
``` shell
1. 在ceph-client节点上创建1个块设备image
rbd create foo --size 4096 [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring]

2. 将块设备image映射为块设备
rbd map foo --name client.admin [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring]

3. 格式化
mkfs.ext4 -m0 /dev/rbd/rbd/foo

4. 挂载
mkdir /mnt/ceph-block-device
mount /dev/rbd/rbd/foo /mnt/ceph-block-device
cd /mnt/ceph-block-device
# 创建文件系统
ceph osd pool create cephfs_data <pg_num>
ceph osd pool create cephfs_metadata <pg_num>
ceph fs new <fs_name> cephfs_metadata cephfs_data

# 创建pool
ceph osd pool create pool_one

# 上传对象到pool
rados put --pool=pool_one object_ceshi /root/1.txt 

# 查看pool中的对象
rados -p pool_one ls
# 等同于 radow --pool=pool_one ls 
# 可以查看rados -h看到帮助
# 可以看到pool_one中的刚才上传的对象object_ceshi 

# 下载pool中的对象
radow get --pool=pool_one object_ceshi /root/2.log

# 重新命名集群
ceph-deploy --cluster rbdcluster new ceph_admin

# 显示映射块设备的rbd
rbd showmapped

# 将ceph中的rbd硬设备块设备
rbd map volumes/data --id admin --keyring /etc/ceph/ceph.client.admin.keyring

# 查看pool中的image信息
ceph osd pool ls

rbd ls harbor
rbd info harbor/foo
# 查看harbor/foo的信息

radosgw-admin bucket stats --bucket xxxxx
# 查看bucket信息
radosgw-admin bucket quota set --bucket xxx --max_size=234234
# 设置bucket的max_size

s3cmd setacl s3://xxxxx --acl-public 
# 设置公开的acl权限（anyone）
```