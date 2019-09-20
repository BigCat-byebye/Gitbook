``` shell
给虚拟机添加磁盘
qemu-img create -f qcow2 /mnt/data/kvm/images/vm1-extra1.qcow2 50G
# 创建硬盘镜像，如果-f qcow2则创建qcow2格式的硬盘，此种格式为硬盘动态扩展，限制硬盘最大容量为50G，而如果-f raw，则创建raw格式的硬盘，此种格式为硬盘固定大小
# 可以分别创建然后使用ls -lh 查看2各种硬盘格式大小，即可知道区别

virsh attach-disk vm1 /mnt/data/kvm/images/vm1-extra1.qcow2 vdd
# 此处vm1-extra1.img必须填写绝对路径，否则报错
# 永久添加硬盘，可使用virsh attach-disk vm1 xxx xxx --persistent

virsh detach-disk vm1 --target vdd
# 卸载硬盘

修改虚拟机配置文件
virsh dumpxml vm1 >> 1.xml
virsh eidt vm1 

kvm磁盘迁移
virt-clone --original centos.img --name vm1 --file `pwd`/vm2.qcow2
# 将机器克隆，但是此方法不能跨机器迁移

virt-install  --name test --vcpus=4 --ram=4096 --disk path=/mnt/data/test.img,bus=virtio --network bridge=br0 --graphics none --console pty --features kvm_hidden=on --machine pc-q35-rhel7.4 --import
# 提前将虚拟机备份，将磁盘拷贝，主要参数为--import
