#  LVM（逻辑卷管理器）

LVM（Logical Volume Manager）逻辑卷管理器

作用：解决传统分区大小固定、扩缩容困难、易丢数据的问题 

核心思想：在物理硬盘和文件系统之间加一层“逻辑层”，让分区大小可以动态伸缩（不丢数据）

## 三层结构

**PV（Physical Volume，物理卷）** 

最底层：可以是整块硬盘、硬盘分区、RAID 阵列 

作用：把物理设备加入 LVM 系统，成为“原料” 

命令：pvcreate /dev/sdb1

**VG（Volume Group，卷组）** 

中间层：把多个 PV 合并成一个“大池子” 

作用：把所有 PV 的空间整合成一个统一资源池（容量 = 所有 PV 总和 - 元数据损耗） 

命令：vgcreate vgdata /dev/sdb1 /dev/sdc1 口诀：PV → 原料，VG → 大仓库

**LV（Logical Volume，逻辑卷）** 

最上层：从 VG 中切出一块“逻辑分区”给用户用 

作用：用户真正挂载使用的分区，可以随时扩容/缩容 

命令：lvcreate -L 10G -n lvdata vgdata 

口诀：LV → 用户看到的盘，可以随意切大切小

**PE（Physical Extent，物理扩展）**

- VG 的最小存储单位（默认 4MB，可自定义）
- VG 把所有 PV 的空间切成一个个 PE 小块
- LV 从 VG 里拿 PE 拼成自己的空间
- 扩容时：VG 有空闲 PE → LV 就能加 PE（不丢数据）



PV（原料）→ VG（大仓库）→ PE（最小砖块）→ LV（用户用的盘）

## LVM 创建/使用完整流程（PV → VG → LV → 格式化 → 挂载）

```bash
[root@localhost ~]# pvcreate /dev/nvme0n2 /dev/nvme0n3
  Physical volume "/dev/nvme0n2" successfully created.
  Physical volume "/dev/nvme0n3" successfully created.
# 准备物理卷
[root@localhost ~]# vgcreate storage /dev/nvme0n2 /dev/nvme0n3
  Volume group "storage" successfully created
# 创建卷组
[root@localhost ~]# vgdisplay storage
  --- Volume group ---
  VG Name               storage
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               9.99 GiB
  PE Size               4.00 MiB
  Total PE              2558
  Alloc PE / Size       0 / 0   
  Free  PE / Size       2558 / 9.99 GiB
  VG UUID               HOxf0E-qrOJ-VUaE-BZ48-VooE-yvSP-Rbk0Py
   
[root@localhost ~]# lvcreate -n vo -l 37 storage
  Logical volume "vo" created.
# 创建约为150MB的逻辑卷（使用-l 37可以生成一个大小为37×4MB=148MB的逻辑卷，也可以使用-L 150M生成一个大小为150MB的逻辑卷）
[root@localhost ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/storage/vo
  LV Name                vo
  VG Name                storage
  LV UUID                bBNh3E-cvWj-Qhzg-ucYt-gTVg-NvNz-lUfAYW
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-16 10:49:53 +0800
  LV Status              available
  # open                 0
  LV Size                148.00 MiB
  Current LE             37
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/rl/swap
  LV Name                swap
  VG Name                rl
  LV UUID                LutyLA-r0o4-eTla-gkC2-QVIp-08yl-ywFagt
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-05 17:15:25 +0800
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/rl/root
  LV Name                root
  VG Name                rl
  LV UUID                7QtRbR-pLGR-d7bB-MKH3-k4CU-NJPo-AM5m12
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-05 17:15:25 +0800
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
[root@localhost ~]# lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0           11:0    1  1.7G  0 rom  
nvme0n1      259:0    0   20G  0 disk 
├─nvme0n1p1  259:1    0    1G  0 part /boot
└─nvme0n1p2  259:2    0   19G  0 part 
  ├─rl-root  253:0    0   17G  0 lvm  /
  └─rl-swap  253:1    0    2G  0 lvm  [SWAP]
nvme0n2      259:3    0    5G  0 disk 
└─storage-vo 253:2    0  148M  0 lvm  
nvme0n3      259:4    0    5G  0 disk 
nvme0n4      259:5    0    5G  0 disk 
nvme0n5      259:6    0    5G  0 disk 
nvme0n6      259:7    0    5G  0 disk 
[root@localhost ~]# mkfs.ext4 /dev/storage/vo
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 151552 1k blocks and 37848 inodes
Filesystem UUID: d0f2fcdb-2d68-4751-b871-8443ddc28c60
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 
# 格式化
[root@localhost ~]# mkdir /mnt/vo
[root@localhost ~]# mount /dev/storage/vo /mnt/vo
# 挂载
```

## 扩容逻辑卷

```bash
[root@localhost ~]# umount /dev/storage/vo
# 先卸载防止文件损坏
[root@localhost ~]# lvextend -L 290MB /dev/storage/vo
  Rounding size to boundary between physical extents: 292.00 MiB.
  Size of logical volume storage/vo changed from 148.00 MiB (37 extents) to 292.00 MiB (73 extents).
  Logical volume storage/vo successfully resized.
# 扩容至290MB
[root@localhost ~]# e2fsck -f /dev/storage/vo
e2fsck 1.46.5 (30-Dec-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/storage/vo: 11/37848 files (0.0% non-contiguous), 15165/151552 blocks
# 检查硬盘完整性（-f：强制检查，即使文件系统看起来干净也检查）
[root@localhost ~]# resize2fs /dev/storage/vo
resize2fs 1.46.5 (30-Dec-2021)
Resizing the filesystem on /dev/storage/vo to 299008 (1k) blocks.
The filesystem on /dev/storage/vo is now 299008 (1k) blocks long.
# 重置硬盘容量
[root@localhost ~]# systemctl daemon-reload
# 重载系统systemd的配置文件
[root@localhost ~]# !mount
mount /dev/storage/vo /mnt/vo
# 重新挂载
[root@localhost ~]# df -Th
Filesystem             Type      Size  Used Avail Use% Mounted on
devtmpfs               devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                  tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs                  tmpfs     726M  9.0M  717M   2% /run
/dev/mapper/rl-root    xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1         xfs       960M  226M  735M  24% /boot
tmpfs                  tmpfs     363M     0  363M   0% /run/user/0
/dev/mapper/storage-vo ext4      268M   14K  250M   1% /mnt/vo

[root@localhost ~]# umount /dev/storage/vo
[root@localhost ~]# lvextend -L 6G /dev/storage/vo
  Size of logical volume storage/vo changed from 292.00 MiB (73 extents) to 6.00 GiB (1536 extents).
  Logical volume storage/vo successfully resized.
# 扩容至6G
[root@localhost ~]# e2fsck -f /dev/storage/vo
e2fsck 1.46.5 (30-Dec-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/storage/vo: 11/73704 files (0.0% non-contiguous), 24683/299008 blocks
[root@localhost ~]# resize2fs /dev/storage/vo
resize2fs 1.46.5 (30-Dec-2021)
Resizing the filesystem on /dev/storage/vo to 6291456 (1k) blocks.
The filesystem on /dev/storage/vo is now 6291456 (1k) blocks long.

[root@localhost ~]# !mount
mount /dev/storage/vo /mnt/vo
[root@localhost ~]# df -Th
Filesystem             Type      Size  Used Avail Use% Mounted on
devtmpfs               devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                  tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs                  tmpfs     726M  9.0M  717M   2% /run
/dev/mapper/rl-root    xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1         xfs       960M  226M  735M  24% /boot
tmpfs                  tmpfs     363M     0  363M   0% /run/user/0
/dev/mapper/storage-vo ext4      5.7G   14K  5.4G   1% /mnt/vo
[root@localhost ~]# lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0           11:0    1  1.7G  0 rom  
nvme0n1      259:0    0   20G  0 disk 
├─nvme0n1p1  259:1    0    1G  0 part /boot
└─nvme0n1p2  259:2    0   19G  0 part 
  ├─rl-root  253:0    0   17G  0 lvm  /
  └─rl-swap  253:1    0    2G  0 lvm  [SWAP]
nvme0n2      259:3    0    5G  0 disk 
└─storage-vo 253:2    0    6G  0 lvm  /mnt/vo
nvme0n3      259:4    0    5G  0 disk 
└─storage-vo 253:2    0    6G  0 lvm  /mnt/vo
nvme0n4      259:5    0    5G  0 disk 
nvme0n5      259:6    0    5G  0 disk 
nvme0n6      259:7    0    5G  0 disk 
# 如果扩容到6G，则会占用到nvme0n3的pv
```

## 缩小逻辑卷

```bash
[root@localhost ~]# umount /mnt/vo
# 先卸载
[root@localhost ~]# e2fsck -f /dev/storage/vo
# 检查硬盘完整性
e2fsck 1.46.5 (30-Dec-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/storage/vo: 11/393216 files (0.0% non-contiguous), 47214/1572864 blocks
[root@localhost ~]# resize2fs /dev/storage/vo 120M
# 重置硬盘容量为120M
resize2fs 1.46.5 (30-Dec-2021)
Resizing the filesystem on /dev/storage/vo to 30720 (4k) blocks.
The filesystem on /dev/storage/vo is now 30720 (4k) blocks long.

[root@localhost ~]# lvreduce -L 120M /dev/storage/vo
# 缩小卷为120M 因为已经resize2fs过了所以下面会显示skipping
  File system ext4 found on storage/vo.
  File system size (120.00 MiB) is equal to the requested size (120.00 MiB).
  File system reduce is not needed, skipping.
  Size of logical volume storage/vo changed from 6.00 GiB (1536 extents) to 120.00 MiB (30 extents).
  Logical volume storage/vo successfully resized.
[root@localhost ~]# !mount
# 重新挂载
mount /dev/storage/vo /mnt/vo
[root@localhost ~]# df -Th
Filesystem             Type      Size  Used Avail Use% Mounted on
devtmpfs               devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                  tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs                  tmpfs     726M   11M  716M   2% /run
/dev/mapper/rl-root    xfs        17G  1.4G   16G   8% /
/dev/nvme0n1p1         xfs       960M  226M  735M  24% /boot
tmpfs                  tmpfs     363M     0  363M   0% /run/user/0
/dev/mapper/storage-vo ext4       51M   24K   43M   1% /mnt/vo
# 显示的 51M 是 ext4 文件系统在 120M LV 上的实际可用空间（减去元数据、预留块等开销后）
```

## 逻辑卷快照

```bash
[root@localhost ~]# lvcreate -s -n SNAP -L 120M /dev/storage/vo
# 强烈建议快照大小和原来的LVM一样大，虽然指定大小是120M，并不是创建一个120M大小的快照。Copy-On-Write (CoW) 是 LVM 快照（snapshot）的核心机制，中文叫“写时复制”。快照创建时并不复制数据，而是让快照“指向”原始卷的所有块；只有当原始卷被修改时，才把被改的块复制一份到快照空间，原始卷继续改，快照保持原始状态。
  Logical volume "SNAP" created.
[root@localhost ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/storage/vo
  LV Name                vo
  VG Name                storage
  LV UUID                AHuS3y-Mc0f-zWAA-yKxa-qiHD-yyFG-x2KCCX
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-16 13:20:26 +0800
  LV snapshot status     source of
                         SNAP [active]
  LV Status              available
  # open                 1
  LV Size                120.00 MiB
  Current LE             30
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---                           #快照逻辑卷
  LV Path                /dev/storage/SNAP
  LV Name                SNAP
  VG Name                storage
  LV UUID                jMn5UO-0ZHZ-UBfI-dT0q-xuRf-cQh6-Yrfcvw
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-16 13:30:09 +0800
  LV snapshot status     active destination for vo
  LV Status              available
  # open                 0
  LV Size                120.00 MiB
  Current LE             30
  COW-table size         120.00 MiB
  COW-table LE           30
  Allocated to snapshot  0.01%                     # 此时cow几乎不占用
  Snapshot chunk size    4.00 KiB
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:5
   
  --- Logical volume ---
  LV Path                /dev/rl/swap
  LV Name                swap
  VG Name                rl
  LV UUID                LutyLA-r0o4-eTla-gkC2-QVIp-08yl-ywFagt
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-05 17:15:25 +0800
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/rl/root
  LV Name                root
  VG Name                rl
  LV UUID                7QtRbR-pLGR-d7bB-MKH3-k4CU-NJPo-AM5m12
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-05 17:15:25 +0800
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
[root@localhost ~]# dd if=/dev/zero of=/mnt/vo/file bs=30M count=1
# 输入30M垃圾文件
1+0 records in
1+0 records out
31457280 bytes (31 MB, 30 MiB) copied, 0.0541092 s, 581 MB/s
[root@localhost ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/storage/vo
  LV Name                vo
  VG Name                storage
  LV UUID                AHuS3y-Mc0f-zWAA-yKxa-qiHD-yyFG-x2KCCX
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-16 13:20:26 +0800
  LV snapshot status     source of
                         SNAP [active]
  LV Status              available
  # open                 1
  LV Size                120.00 MiB
  Current LE             30
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/storage/SNAP
  LV Name                SNAP
  VG Name                storage
  LV UUID                jMn5UO-0ZHZ-UBfI-dT0q-xuRf-cQh6-Yrfcvw
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-16 13:30:09 +0800
  LV snapshot status     active destination for vo
  LV Status              available
  # open                 0
  LV Size                120.00 MiB
  Current LE             30
  COW-table size         120.00 MiB
  COW-table LE           30
  Allocated to snapshot  40.74%              # 此时快照cow占用增加
  Snapshot chunk size    4.00 KiB
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:5
   
  --- Logical volume ---
  LV Path                /dev/rl/swap
  LV Name                swap
  VG Name                rl
  LV UUID                LutyLA-r0o4-eTla-gkC2-QVIp-08yl-ywFagt
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-05 17:15:25 +0800
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/rl/root
  LV Name                root
  VG Name                rl
  LV UUID                7QtRbR-pLGR-d7bB-MKH3-k4CU-NJPo-AM5m12
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-05 17:15:25 +0800
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
[root@localhost ~]# umount /mnt/vo
# 先卸载
[root@localhost ~]# lvconvert --merge /dev/storage/SNAP
# 恢复快照
# lvconvert：LVM 的转换工具，用于修改逻辑卷属性、类型、合并快照等
# --merge：指定操作类型为“合并快照”（merge snapshot）。
  Merging of volume storage/SNAP started.
  storage/vo: Merged: 100.00%
[root@localhost ~]# !mount
# 重新挂载
mount /dev/storage/vo /mnt/vo
[root@localhost ~]# lvdisplay
# vo恢复到快照创建前的状态，SNAP卷被自动删除
  --- Logical volume ---
  LV Path                /dev/storage/vo
  LV Name                vo
  VG Name                storage
  LV UUID                AHuS3y-Mc0f-zWAA-yKxa-qiHD-yyFG-x2KCCX
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-16 13:20:26 +0800
  LV Status              available
  # open                 1
  LV Size                120.00 MiB
  Current LE             30
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/rl/swap
  LV Name                swap
  VG Name                rl
  LV UUID                LutyLA-r0o4-eTla-gkC2-QVIp-08yl-ywFagt
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-05 17:15:25 +0800
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/rl/root
  LV Name                root
  VG Name                rl
  LV UUID                7QtRbR-pLGR-d7bB-MKH3-k4CU-NJPo-AM5m12
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2026-03-05 17:15:25 +0800
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

```

## 删除逻辑卷

```bash
[root@localhost ~]# umount /mnt/vo
# 先卸载vo（如果写入配置文件要删除配置文件中永久生效的设备参数 vim /etc/fstab ）
[root@localhost ~]# lvremove -y /dev/storage/vo
# 删除逻辑卷lv
  Logical volume "vo" successfully removed.
[root@localhost ~]# vgremove /dev/storage
# 删除卷组vg
  Volume group "storage" successfully removed
[root@localhost ~]# pvremove /dev/nvme0n2 /dev/nvme0n3
# 删除物理卷pv
  Labels on physical volume "/dev/nvme0n2" successfully wiped.
  Labels on physical volume "/dev/nvme0n3" successfully wiped.

```

