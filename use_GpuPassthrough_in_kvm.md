``` shell
1. 安装kvm组件
yum install -y qemu-kvm libvirt virt-install bridge-utils 

2. 检查kvm模块开始
lsmod | grep kvm

3. 设置网桥配置文件
cd /etc/sysconfig/network-script/
cat ifcfg-br0
DEVICE="br0"
ONBOOT="yes"
TYPE="Bridge"
BOOTPROTO=static
IPADDR=192.168.2.32
NETMASK=255.255.255.0
GATEWAY=192.168.2.1
DEFROUTE=yes
修改桥接的网卡配置文件
TYPE="Ethernet"
NAME="eno1"
DEVICE="eno1"
BRIDGE="br0"
ONBOOT=yes
MTU=1500
# 如果虚拟机网桥和docker的网桥冲突，需要删除docker网桥，并禁止docker修改iptables规则
cat /etc/systemd/system/docker.service
ExeStart=/usr/bin/dockerd --iptables=falsse
systemctl daemon-reload
brctl show
ip link set dev docker0 down
brctl delbr docker0
# 如果此时无法访问192.168.2.1/24网段，请在/etc/rc.local中加入
cat /etc/rc.local
route add default gw 192.168.2.1

4. 屏蔽nouveau模块（一般机器都会安装的通用显卡）
cd /etc/modprobe.d/
echo "blacklist nouveau" > blacklist.conf
mv /boot/initramfs-$(uname -r).img  .boot/initramfs-$(uname -r).img.bak
dracut -v /boot/initramfs-$(uname -r).img $(uname -r)
reboot

5. 修改内核参数
cat /etc/default/grub
# 加入iommu相关配置，并禁止宿主机调用Gpu
GRUB_CMDLINE_LINUX="crashkernel=aut selinux=0 ipv6.disable=1 rhgb quiet inter_iommu=on iommu=ptrd.driver.blacklist=nonveau nouveau.modeset=0 pci-stub.ids=10de:1b06"
# 生成内核启动参数
grub2-mkconfig -o /etc/grub2.cfg
reboot
# 注意其中pci-stub.ids后的参数为
lspci -nnk | grep -i nvidia 看到的参数，如图9
![Gpu](https://uploader.shimo.im/f/9txgGhqVgZkQD15r.png)



6. 增加Gpu设备定义文件,准备后用
cat gpu0.xml
<hostdev mode=`subsystem` type='pci' managed='yes'>
  <driver name='vfio' />
  <source>
    <address domain='0x0000' bus='0x03' slot='0x00' function='0x0' /> 
  </source>
</hostdev>
# bus,slot,function的参数如上图9

7. 设置好以上网桥后，开始kvm安装虚拟机
virt-install \
--name vm1 \
--vcpu2=12 \
--ram=14336 \
--disk path=/mnt/datax/kvm/images/vm1.qcow2,size=64 \
# 此处参数size表示磁盘大小，单位G
--network bridge=br0 \
--graphics none \
--location=/nfs-public/share/Centos7-1708.iso \
--console pty \
--extra-args 'console=ttyS0' \
--features kvm_hidden=on \
# 此参数是为了后面配置Gpu穿透到虚拟机，因为当Gpu检测到处于虚拟机时，就会停止工作，因此添加此参数，表示隐藏kvm信息，防止Gpu识别
--force

8. 进入刚创建的虚拟机，屏蔽nouveau模块
virtsh list 
virsh console vm1
# 进入虚拟机
由于安装的虚拟机也是一个完整的系统，因此在虚拟机里面也需要屏蔽nouveau模块，请重复步骤7
 安装驱动
./NVIDIA.run
