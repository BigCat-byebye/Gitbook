# LVM 

` LVM ` 全名` locical volume manager `，俗称“逻辑卷”，最重要的特点就是可以整合多个物理磁盘，弹性调整文件系统容量

1. 逻辑概念

   `pv`，物理卷，实际上就是1块物理磁盘

   `pe`：物理扩展，是整个`lvm`中的最小存储单位，默认`4M`

   `vg`：卷组，是由`pe`组成的集合，大小取决于`pe`，1个`vg`最多有65534个`pe`

   `lv`：逻辑卷，在`vg`的基础上，重新划分的单元，`lvm`的可扩展性也是体现在`lv`这一概念上

2. 基本操作

   ```
   pvcreate /dev/sdd # 将硬盘/dev/sdd创建为pv，注意需要保证磁盘/dev/sdd的格式为lvm格式 
   pvdisplay /dev/sdd # 显示/dev/sdd这个pv的信息，可简写为pvs
   vgcreate vg3 /dev/sdd /dev/sde # 使用pv（/dev/sdd和/dev/sde）创建1个vg3
   lvcreate -L 2G -n lv1 vg3 # 在vg3上创建1个lv1，大小为2G
   mkfs.ext4 /dev/vg3/lv1 # 格式化lv1后，即可使用
   ```

3. 扩容文件系统

   ```
   1. 扩容逻辑卷
   lvresize -L 20G /dev/vg3/lv1
   2. 检查lv
   e2fsck -f /dev/vg3/lv1
   3. 扩容文件系统
   resize2fs /dev/vg3/lv1
   ```

4. 缩容文件系统

   ```
   1. 卸载lv
   umount /dev/vg3/lv1
   2. 检查lv
   e2fsck -f /dev/vg3/lv1
   3. 缩容文件系统
   resize2fs /dev/vg3/lv1 900M
   4. 缩容lv
   lvresize -L 900M /dev/vg3/lv1
   ```

# Raid

`Raid`全名`redundant array of independent disks`，俗称磁盘阵列，即将多个磁盘组合起来达到更好的存储性能和数据冗余的技术

1. 逻辑概念（常见)

   **条带化**（`Raid 0`）：将数据分为多块，同时写入多个磁盘，每个磁盘承载一部分数据，可有效提高数据的`I/O`速度，优点：写入读取的速度非常快，缺点：阵列中的任何一块磁盘出现问题，该阵列中存储的所有数据都不可用

   **镜像**（`Raid 1`）：将数据复制为多份，分别写入不同的磁盘，每个磁盘的数据内容完全一致，优点：完整的数据冗余，缺点：磁盘的有效使用率降低

   **数据校验**：在数据写入多个盘时，对完整数据进行校验计算，将校验结算结果写入磁盘，当一部分数据出错，可凭借校验码和其他磁盘的数据找回出错的那一部分数据，优点：条带化和镜像的折中办法，缺点：数据读写时，要进行大量计算，需使用硬件`Raid`控制器

   **校验码**（`Raid 2或3或4或5或6`）：常用为海明校验码（`Raid 2`）和异或校验码，其中`Raid 5`最常用

   `Raid 01`：先将数据镜像，再条带化存储

   `Raid 10`：先将数据条带化，再镜像化存储

中心`Raid卡`操作配置：[LSI SAS 3008配置操作](https://blog.csdn.net/dufufd/article/details/79850868)

