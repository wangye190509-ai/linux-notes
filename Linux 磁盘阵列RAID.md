

# Linux 磁盘阵列RAID



## 磁盘阵列

磁盘阵列（RAID，**R**edundant **A**rray of **I**ndependent **D**isks）是一种将多个物理硬盘组合在一起形成一个逻辑硬盘组的技术。通过 RAID，可以实现数据冗余、性能提升或两者兼具。RAID 在存储系统中广泛使用，尤其是在服务器和大数据存储环境中，用于提升数据可靠性和性能。

### 常见 RAID 级别对比

RAID0 : 数据加速

RAID1 :数据备份

RAID5 :综合数据加速和数据备份，原理是通过奇偶校验来进行计算

RAID10 :先RAID1再 RAID0，综合RAID1和RAID0的优点，至少需要四块磁盘

RAID01 :先RAID0再RAID1，综合RAID1和RAID0的优点，至少需要四块磁盘

### 硬件 RAID 和软件 RAID

| **对比项**   | **硬件 RAID**                            | **软件 RAID**                      |
| ------------ | ---------------------------------------- | ---------------------------------- |
| **实现方式** | 专用 RAID 控制器                         | 操作系统或软件实现                 |
| **性能**     | 高，特别是在 RAID 5/6 等级别中           | 较低，占用主机 CPU 资源            |
| **磁盘管理** | 独立于操作系统，系统只看到一个逻辑磁盘   | 依赖于操作系统的磁盘管理           |
| **成本**     | 高，需购买 RAID 控制器                   | 低，无需额外硬件                   |
| **功能支持** | 支持热备盘、硬盘监控、电池缓存等高级功能 | 功能较少，依赖操作系统提供的功能   |
| **灵活性**   | 依赖特定硬件，不易迁移                   | 容易迁移，跨硬件平台使用           |
| **故障恢复** | 控制器损坏可能需要相同型号的控制器       | 可以在不同硬件上恢复               |
| **适用场景** | 企业级、大型存储系统，数据中心           | 个人用户、小型服务器，开发测试环境 |

**硬件 RAID**：性能更好、可靠性更高、运维更简单，但**成本高** → 适合**企业级、生产环境、对数据安全和 I/O 性能要求高**的场景。

**软件 RAID**：几乎零成本、灵活性强、支持所有常见 RAID 级别，但**占用主机资源、重建慢** → 适合**学习、个人服务器、低预算 NAS(Network Attached Storage网络附加存储)、中小规模应用**。

## 

## mdadm命令

通过软件的形式构建磁盘整列

**常用参数以及作用**

| 参数                                                      | 作用                                            |
| --------------------------------------------------------- | ----------------------------------------------- |
| -a yes                                                    | 自动（auto）创建所需的设备文件（如 `/dev/md0`） |
| -n                                                        | 指定设备数量                                    |
| -l                                                        | 指定RAID级别（level）                           |
| -C                                                        | 创建                                            |
| -v                                                        | 显示过程                                        |
| -f                                                        | 模拟设备损坏                                    |
| -r                                                        | 移除设备                                        |
| xxxxxxxxxx [root@localhost ~]# systemctl restart sshdbash | 查看摘要信息                                    |
| -D                                                        | 查看详细信息                                    |
| -S                                                        | 停止RAID磁盘阵列                                |
| -x                                                        | 备份盘数量                                      |

```bash
mdadm -Cv /dev/mdX -a yes -n 磁盘数 -l RAID级别 [磁盘列表]
```



## RAID0 阵列实验

### 部署

```shell
[root@localhost ~]# mdadm -Cv /dev/md0 -a yes -n 2 -l 0 /dev/nvme0n2 /dev/nvme0n3
mdadm: --auto is deprecated and will be removed in future releases.
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

# 用mdadm创建Raid0磁盘阵列
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sr0          11:0    1  1.7G  0 rom   
nvme0n1     259:0    0   20G  0 disk  
├─nvme0n1p1 259:1    0    1G  0 part  /boot
└─nvme0n1p2 259:2    0   19G  0 part  
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
nvme0n2     259:3    0    5G  0 disk  
└─md0         9:0    0   10G  0 raid0 
nvme0n3     259:4    0    5G  0 disk  
└─md0         9:0    0   10G  0 raid0 
nvme0n4     259:5    0    5G  0 disk  
nvme0n5     259:6    0    5G  0 disk  
nvme0n6     259:7    0    5G  0 disk  
[root@localhost ~]# mkfs.ext4 /dev/md0    
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2618880 4k blocks and 655360 inodes
Filesystem UUID: ea8254a3-e282-40cd-b66c-00ec3b71150f
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

# 格式化文件系统

[root@localhost ~]# mkdir /mnt/RAID0       
# 创建挂载点
[root@localhost ~]# mount /dev/md0 /mnt/RAID0
# 挂载
[root@localhost ~]# df -TH
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.2M     0  4.2M   0% /dev
tmpfs               tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs               tmpfs     761M  9.5M  752M   2% /run
/dev/mapper/rl-root xfs        19G  1.5G   17G   9% /
/dev/nvme0n1p1      xfs       1.1G  237M  771M  24% /boot
tmpfs               tmpfs     381M     0  381M   0% /run/user/0
/dev/md0            ext4       11G   25k   10G   1% /mnt/RAID0
[root@localhost ~]# echo "/dev/md0 /mnt/RAID0 ext4 defaults 0 0" >> /etc/fstab
# mount为临时挂载，也可以将挂载信息写入/etc/fstab文件中，进行永久挂载
[root@localhost ~]# mount -a 
[root@localhost ~]# df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs               tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs               tmpfs     726M  9.0M  717M   2% /run
/dev/mapper/rl-root xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1      xfs       960M  226M  735M  24% /boot
tmpfs               tmpfs     363M     0  363M   0% /run/user/0
/dev/md0            ext4      9.8G   24K  9.3G   1% /mnt/RAID0
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 15:38:20 2026
        Raid Level : raid0
        Array Size : 10475520 (9.99 GiB 10.73 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Mar 15 15:38:20 2026
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

            Layout : original
        Chunk Size : 512K

Consistency Policy : none

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : d0b01b81:013581ad:69061c00:a1fb4a03
            Events : 0

    Number   Major   Minor   RaidDevice State
       0     259        3        0      active sync   /dev/nvme0n2
       1     259        4        1      active sync   /dev/nvme0n3

# 查看/dev/md0磁盘阵列的详细信息

# Number: 这个字段表示每个磁盘在 RAID 阵列中的编号（从 0 开始）。
# 这是 RAID 阵列中每个设备的索引或设备号。

# Major: 这是设备文件的主设备号（major number），
# 它用于标识设备的类型。
# 不同的主设备号通常对应不同的驱动程序或设备类别。

# Minor: 这是设备文件的次设备号（minor number），
# 用于标识同一类型设备中的不同实例。
# 主设备号和次设备号共同唯一标识系统中的一个设备。

# RaidDevice: 这是设备在 RAID 阵列中的逻辑设备编号。
# 通常从 0 开始，表示该设备在 RAID 阵列中的顺序。

# State: 表示 RAID 阵列中该设备的当前状态。
# 常见状态如下：
#   - active sync: 表示该设备处于活动状态，并且与 RAID 阵列中的其他设备同步。
#   - faulty: 表示设备出现故障，不能正常工作。
#   - spare: 表示该设备是热备盘，当其他设备故障时可自动替换。
```

### 测试

通过fio磁盘测试工具来测试我们的RAID 0阵列的读写情况

#### fio

fio是 Linux 上最强大的磁盘性能测试工具，它可以精确模拟各种 I/O 负载模式。

| 参数                  | 含义             | 典型值 / 说明                                                |
| :-------------------- | :--------------- | :----------------------------------------------------------- |
| **`filename`**        | 测试目标         | 可以是磁盘设备（如 `/dev/sdb`）或文件路径（如 `/mnt/test/testfile`）。**注意**：直接测试设备会破坏文件系统！ |
| **`direct`**          | 是否使用直接 I/O | `1` 表示绕过文件系统缓存，直接读写磁盘，结果更真实 。        |
| **`ioengine`**        | I/O 引擎         | **`libaio`**：Linux 原生异步 I/O，最常用 。 **`sync`**：同步读写。 **`psync`**：使用 `pread`/`pwrite`。 |
| **`rw`**              | 读写模式         | **`read`/`write`**：顺序读/写 。 **`randread`/`randwrite`**：随机读/写 。 **`randrw`**：混合随机读写，用 `rwmixread` 调整比例 。 |
| **`bs`**              | 块大小           | 每次 I/O 操作的数据块大小。测 IOPS 常用 `4k`，测吞吐量常用 `1m` 。 |
| **`iodepth`**         | I/O 队列深度     | 在异步模式下，一次性提交多少个 I/O 请求。数值越大，并发越高 。 |
| **`numjobs`**         | 并发线程数       | 同时启动多少个工作线程/进程 。                               |
| **`size`**            | 测试数据量       | 每个线程读/写的数据总量 。与 `runtime` 冲突时，以先到者为准。 |
| **`runtime`**         | 测试时长         | 测试持续运行的时间（如 `60s`）。常与 `time_based` 配合使用。 |
| **`time_based`**      | 基于时间         | 即使 `size` 已写完，也要跑满 `runtime` 指定的时间 。         |
| **`group_reporting`** | 汇总报告         | 汇总所有 `numjobs` 线程的统计信息，而不是每个线程单独显示 。 |

**顺序写性能测试**

```bash
fio --name=write_test \           # 测试任务名称
     --filename=/mnt/test/testfile \ # 测试文件路径
     --size=1G \                   # 写入1GB数据
     --bs=1M \                     # 每次I/O块大小=1MB
     --rw=write \                  # 顺序写入模式
     --direct=1 \                   # 直接I/O，绕过缓存
     --numjobs=1                    # 单线程测试
```



#### 单块硬盘测试

```bash
[root@localhost ~]# mkdir /mnt/test
# 创建挂载点
[root@localhost ~]# mkfs.ext4 /dev/nvme0n4
# 格式化
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1310720 4k blocks and 327680 inodes
Filesystem UUID: 8ca69eee-d111-4f98-8811-d6b50e77f70d
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

[root@localhost ~]# mount /dev/nvme0n4 /mnt/test
# 挂载
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# fio --name=write_test --filename=/mnt/test/testfile --size=1G --bs=1M --rw=write --direct=1 --numjobs=1
# 进行磁盘写入测试
write_test: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=psync, iodepth=1
fio-3.35
Starting 1 process
write_test: Laying out IO file (1 file / 1024MiB)
Jobs: 1 (f=1)
write_test: (groupid=0, jobs=1): err= 0: pid=1841: Sun Mar 15 16:11:03 2026
  write: IOPS=929, BW=929MiB/s (974MB/s)(1024MiB/1102msec); 0 zone resets
    clat (usec): min=689, max=8807, avg=1057.58, stdev=406.18
     lat (usec): min=704, max=8822, avg=1073.74, stdev=406.34
    clat percentiles (usec):
     |  1.00th=[  734],  5.00th=[  799], 10.00th=[  857], 20.00th=[  898],
     | 30.00th=[  930], 40.00th=[  963], 50.00th=[  996], 60.00th=[ 1037],
     | 70.00th=[ 1074], 80.00th=[ 1139], 90.00th=[ 1270], 95.00th=[ 1450],
     | 99.00th=[ 2089], 99.50th=[ 2245], 99.90th=[ 8717], 99.95th=[ 8848],
     | 99.99th=[ 8848]
   bw (  KiB/s): min=951289, max=952605, per=100.00%, avg=951947.00, stdev=930.55, samples=2
   iops        : min=  928, max=  930, avg=929.00, stdev= 1.41, samples=2
  lat (usec)   : 750=1.66%, 1000=49.02%
  lat (msec)   : 2=47.95%, 4=1.17%, 10=0.20%
  cpu          : usr=0.18%, sys=4.72%, ctx=1025, majf=0, minf=9
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,1024,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=929MiB/s (974MB/s), 929MiB/s-929MiB/s (974MB/s-974MB/s), io=1024MiB (1074MB), run=1102-1102msec

Disk stats (read/write):
  nvme0n4: ios=0/1722, merge=0/0, ticks=0/1724, in_queue=1723, util=89.15%

```

#### RAID0测试

```bash
[root@localhost ~]# fio --name=write_test --filename=/mnt/RAID0/testfile --size=1G --bs=1M --rw=write --direct=1 --numjobs=1
write_test: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=psync, iodepth=1
fio-3.35
Starting 1 process
write_test: Laying out IO file (1 file / 1024MiB)
Jobs: 1 (f=1)
write_test: (groupid=0, jobs=1): err= 0: pid=1863: Sun Mar 15 16:20:05 2026
  write: IOPS=951, BW=952MiB/s (998MB/s)(1024MiB/1076msec); 0 zone resets
    clat (usec): min=689, max=8416, avg=1032.15, stdev=386.87
     lat (usec): min=702, max=8434, avg=1048.71, stdev=387.52
    clat percentiles (usec):
     |  1.00th=[  742],  5.00th=[  807], 10.00th=[  865], 20.00th=[  898],
     | 30.00th=[  922], 40.00th=[  947], 50.00th=[  971], 60.00th=[ 1004],
     | 70.00th=[ 1037], 80.00th=[ 1090], 90.00th=[ 1221], 95.00th=[ 1401],
     | 99.00th=[ 1975], 99.50th=[ 2540], 99.90th=[ 8225], 99.95th=[ 8455],
     | 99.99th=[ 8455]
   bw (  KiB/s): min=951146, max=993941, per=99.80%, avg=972543.50, stdev=30260.63, samples=2
   iops        : min=  928, max=  970, avg=949.00, stdev=29.70, samples=2
  lat (usec)   : 750=1.17%, 1000=58.01%
  lat (msec)   : 2=39.94%, 4=0.68%, 10=0.20%
  cpu          : usr=0.09%, sys=4.84%, ctx=1024, majf=0, minf=9
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,1024,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=952MiB/s (998MB/s), 952MiB/s-952MiB/s (998MB/s-998MB/s), io=1024MiB (1074MB), run=1076-1076msec

Disk stats (read/write):
    md0: ios=0/1954, merge=0/0, ticks=0/1951, in_queue=1951, util=89.30%, aggrios=0/1024, aggrmerge=0/120, aggrticks=0/988, aggrin_queue=988, aggrutil=89.14%
  nvme0n2: ios=0/1024, merge=0/120, ticks=0/984, in_queue=984, util=89.14%
  nvme0n3: ios=0/1024, merge=0/120, ticks=0/992, in_queue=992, util=88.46%

```

单块磁盘写入速率：

WRITE: bw=929MiB/s (974MB/s), 929MiB/s-929MiB/s (974MB/s-974MB/s), io=1024MiB (1074MB), run=1102-1102msec

RAID0写入速率：

WRITE: bw=952MiB/s (998MB/s), 952MiB/s-952MiB/s (998MB/s-998MB/s), io=1024MiB (1074MB), run=1076-1076msec



对比单块磁盘和RAID0 我们发现写入速率RAID0虽然快、但是没快多少。按道理来说，RAID 0是将数据分散在两块磁盘上，写入速度理应翻倍。但是由于这是虚拟机，使用我们自己电脑的物理磁盘，我们的物理硬盘只有一块，所以这里看不到RAID 0的双倍读写速度的效果。

### **彻底废弃**磁盘阵列

```bash
[root@localhost ~]# umount /dev/md0
# 卸载已挂载的磁盘(相当于公司关门歇业)
[root@localhost ~]# vim /etc/fstab
# 删除写入/etc/fstab下的挂载命令
[root@localhost ~]# mdadm -S /dev/md0
# 停止磁盘阵列(相当于拆除整栋大楼)
mdadm: stopped /dev/md0
[root@localhost ~]# mdadm --zero-superblock /dev/nvme0n2 /dev/nvme0n3
# 删除阵列中的数据块信息（彻底清除）
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  1.7G  0 rom  
nvme0n1     259:0    0   20G  0 disk 
├─nvme0n1p1 259:1    0    1G  0 part /boot
└─nvme0n1p2 259:2    0   19G  0 part 
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2     259:3    0    5G  0 disk 
nvme0n3     259:4    0    5G  0 disk 
nvme0n4     259:5    0    5G  0 disk /mnt/test
nvme0n5     259:6    0    5G  0 disk 
nvme0n6     259:7    0    5G  0 disk 

```

## RAID1 阵列实验

```bash
[root@localhost ~]# mdadm -Cv /dev/md0 -a yes -n 2 -l 1 /dev/nvme0n2 /dev/nvme0n3
mdadm: --auto is deprecated and will be removed in future releases.
To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 5237760K
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[root@localhost ~]# mkfs.ext4 /dev/md0
mke2fs 1.46.5 (30-Dec-2021)
/dev/md0 contains a ext4 file system
	last mounted on /mnt/RAID0 on Sun Mar 15 15:39:34 2026
Proceed anyway? (y,N) y
Creating filesystem with 1309440 4k blocks and 327680 inodes
Filesystem UUID: 1861d3dc-1f17-4cc6-a924-d68166ceeb4e
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

[root@localhost ~]# mkdir /mnt/RAID1
[root@localhost ~]# mount /dev/md0 /mnt/RAID1
[root@localhost ~]# df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs               tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs               tmpfs     726M  9.1M  717M   2% /run
/dev/mapper/rl-root xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1      xfs       960M  226M  735M  24% /boot
tmpfs               tmpfs     363M     0  363M   0% /run/user/0
/dev/md0            ext4      4.9G   24K  4.6G   1% /mnt/RAID1
[root@localhost ~]# fio --name=write_test --filename=/mnt/RAID1/testfile --size=1G --bs=1M --rw=write --direct=1 --numjobs=1
write_test: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=psync, iodepth=1
fio-3.35
Starting 1 process
write_test: Laying out IO file (1 file / 1024MiB)
Jobs: 1 (f=1)
write_test: (groupid=0, jobs=1): err= 0: pid=5046: Sun Mar 15 19:41:20 2026
  write: IOPS=572, BW=572MiB/s (600MB/s)(1024MiB/1790msec); 0 zone resets
    clat (usec): min=1144, max=9611, avg=1729.62, stdev=619.42
     lat (usec): min=1164, max=9622, avg=1745.56, stdev=619.84
    clat percentiles (usec):
     |  1.00th=[ 1188],  5.00th=[ 1237], 10.00th=[ 1287], 20.00th=[ 1369],
     | 30.00th=[ 1516], 40.00th=[ 1647], 50.00th=[ 1696], 60.00th=[ 1745],
     | 70.00th=[ 1811], 80.00th=[ 1893], 90.00th=[ 2073], 95.00th=[ 2245],
     | 99.00th=[ 3130], 99.50th=[ 6849], 99.90th=[ 9372], 99.95th=[ 9634],
     | 99.99th=[ 9634]
   bw (  KiB/s): min=521088, max=616818, per=99.55%, avg=583172.00, stdev=53829.34, samples=3
   iops        : min=  508, max=  602, avg=569.00, stdev=52.89, samples=3
  lat (msec)   : 2=86.52%, 4=12.89%, 10=0.59%
  cpu          : usr=0.00%, sys=3.80%, ctx=1045, majf=0, minf=9
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,1024,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=572MiB/s (600MB/s), 572MiB/s-572MiB/s (600MB/s-600MB/s), io=1024MiB (1074MB), run=1790-1790msec

Disk stats (read/write):
    md0: ios=0/975, merge=0/0, ticks=0/1670, in_queue=1670, util=93.82%, aggrios=0/1045, aggrmerge=0/0, aggrticks=0/1686, aggrin_queue=1685, aggrutil=92.50%
  nvme0n2: ios=0/1045, merge=0/0, ticks=0/1656, in_queue=1655, util=91.16%
  nvme0n3: ios=0/1045, merge=0/0, ticks=0/1717, in_queue=1716, util=92.50%

```

从测试结果中可以看见，RAID 1阵列的磁盘写入速度会比单块磁盘相对慢一点，因为RAID 1阵列是将一份数据写入两遍。

xxxxxxxxxx [root@localhost ~]# systemctl restart sshdbash

## RAID10 阵列实验

```bash
[root@localhost ~]# mdadm -Cv /dev/md0 -a yes -n 4 -l 10 /dev/nvme0n2 /dev/nvme0n3 /dev/nvme0n4 /dev/nvme0n5
mdadm: --auto is deprecated and will be removed in future releases.
To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: layout defaults to n2
mdadm: layout defaults to n2
mdadm: chunk size defaults to 512K
mdadm: /dev/nvme0n4 appears to contain an ext2fs file system
mdadm: size=5242880K  mtime=Sun Mar 15 16:08:20 2026
mdadm: size set to 5237760K
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[root@localhost ~]# mkfs.ext4 /dev/md0
mke2fs 1.46.5 (30-Dec-2021)
/dev/md0 contains a ext4 file system
	last mounted on /mnt/RAID1 on Sun Mar 15 19:39:12 2026
Proceed anyway? (y,N) y
Creating filesystem with 2618880 4k blocks and 655360 inodes
Filesystem UUID: f9ae8591-71d2-463f-a2bd-3ecc84301894
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

[root@localhost ~]# mkdir /mnt/RAID10
[root@localhost ~]# mount /dev/md0 /mnt/RAID10
[root@localhost ~]# df -TH
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.2M     0  4.2M   0% /dev
tmpfs               tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs               tmpfs     761M  9.5M  752M   2% /run
/dev/mapper/rl-root xfs        19G  1.5G   17G   9% /
/dev/nvme0n1p1      xfs       1.1G  237M  771M  24% /boot
tmpfs               tmpfs     381M     0  381M   0% /run/user/0
/dev/md0            ext4       11G   25k   10G   1% /mnt/RAID10
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 19:47:19 2026
        Raid Level : raid10
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Mar 15 19:50:10 2026
             State : clean 
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 74e0210e:5ccc9527:44a22a4d:ae4d12c2
            Events : 20

    Number   Major   Minor   RaidDevice State
       0     259        3        0      active sync set-A   /dev/nvme0n2
       1     259        4        1      active sync set-B   /dev/nvme0n3
       2     259        5        2      active sync set-A   /dev/nvme0n4
       3     259        6        3      active sync set-B   /dev/nvme0n5

[root@localhost ~]# fio --name=write_test --filename=/mnt/RAID10/testfile --size=1G --bs=1M --rw=write --direct=1 --numjobs=1
write_test: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=psync, iodepth=1
fio-3.35
Starting 1 process
write_test: Laying out IO file (1 file / 1024MiB)
Jobs: 1 (f=1): [W(1)][-.-%][w=510MiB/s][w=509 IOPS][eta 00m:00s]
write_test: (groupid=0, jobs=1): err= 0: pid=5111: Sun Mar 15 19:53:56 2026
  write: IOPS=489, BW=489MiB/s (513MB/s)(1024MiB/2094msec); 0 zone resets
    clat (usec): min=1300, max=61645, avg=2025.89, stdev=1905.18
     lat (usec): min=1324, max=61654, avg=2041.94, stdev=1905.07
    clat percentiles (usec):
     |  1.00th=[ 1516],  5.00th=[ 1696], 10.00th=[ 1745], 20.00th=[ 1778],
     | 30.00th=[ 1827], 40.00th=[ 1860], 50.00th=[ 1893], 60.00th=[ 1926],
     | 70.00th=[ 1975], 80.00th=[ 2057], 90.00th=[ 2245], 95.00th=[ 2540],
     | 99.00th=[ 3261], 99.50th=[ 3785], 99.90th=[ 9241], 99.95th=[61604],
     | 99.99th=[61604]
   bw (  KiB/s): min=427670, max=530889, per=99.83%, avg=499893.00, stdev=48688.92, samples=4
   iops        : min=  417, max=  518, avg=487.50, stdev=47.56, samples=4
  lat (msec)   : 2=74.51%, 4=25.00%, 10=0.39%, 100=0.10%
  cpu          : usr=0.10%, sys=4.20%, ctx=1047, majf=0, minf=11
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,1024,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=489MiB/s (513MB/s), 489MiB/s-489MiB/s (513MB/s-513MB/s), io=1024MiB (1074MB), run=2094-2094msec

Disk stats (read/write):
    md0: ios=0/2934, merge=0/0, ticks=0/5695, in_queue=5695, util=94.47%, aggrios=0/1616, aggrmerge=0/0, aggrticks=0/2985, aggrin_queue=2986, aggrutil=93.90%
  nvme0n4: ios=0/2068, merge=0/0, ticks=0/3846, in_queue=3846, util=93.72%
  nvme0n2: ios=0/1164, merge=0/0, ticks=0/2094, in_queue=2094, util=93.49%
  nvme0n5: ios=0/2068, merge=0/0, ticks=0/3890, in_queue=3890, util=93.90%
  nvme0n3: ios=0/1164, merge=0/0, ticks=0/2113, in_queue=2114, util=93.63%

```

### 损坏磁盘阵列及修复(RAID10为例)

```bash
[root@localhost ~]# mdadm /dev/md0 -f /dev/nvme0n2
# -f 模拟设备损坏
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 20:03:10 2026
        Raid Level : raid10
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Mar 15 20:05:25 2026
             State : active, degraded 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : cf703021:18553a15:0f539169:4ac6d863
            Events : 38

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1     259        4        1      active sync set-B   /dev/nvme0n3
       2     259        5        2      active sync set-A   /dev/nvme0n4
       3     259        6        3      active sync set-B   /dev/nvme0n5

       0     259        3        -      faulty   /dev/nvme0n2  # 此处可以看见/dev/nvme0n2 已损坏
[root@localhost ~]# umount /mnt/RAID10、
# 先卸载，防止还有重要文件正在写入
[root@localhost ~]# mdadm /dev/md0 -r /dev/nvme0n2
mdadm: hot removed /dev/nvme0n2 from /dev/md0
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 20:03:10 2026
        Raid Level : raid10
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 4
     Total Devices : 3
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Mar 15 20:12:45 2026
             State : clean, degraded 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : cf703021:18553a15:0f539169:4ac6d863
            Events : 88

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1     259        4        1      active sync set-B   /dev/nvme0n3
       2     259        5        2      active sync set-A   /dev/nvme0n4
       3     259        6        3      active sync set-B   /dev/nvme0n5
[root@localhost ~]# mdadm /dev/md0 -a /dev/nvme0n2
mdadm: re-added /dev/nvme0n2
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 20:03:10 2026
        Raid Level : raid10
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Mar 15 20:13:46 2026
             State : clean, degraded, recovering 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : bitmap

    Rebuild Status : 45% complete

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : cf703021:18553a15:0f539169:4ac6d863
            Events : 95

    Number   Major   Minor   RaidDevice State
       0     259        3        0      spare rebuilding   /dev/nvme0n2   # 刚添加上的磁盘处于rebuilding的状态，等待一会儿，就会恢复正常了
       1     259        4        1      active sync set-B   /dev/nvme0n3
       2     259        5        2      active sync set-A   /dev/nvme0n4
       3     259        6        3      active sync set-B   /dev/nvme0n5
       
[root@localhost ~]# mount /dev/md0 /mnt/RAID10
# 重新挂载就可以使用了
[root@localhost ~]# df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs               tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs               tmpfs     726M  9.1M  717M   2% /run
/dev/mapper/rl-root xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1      xfs       960M  226M  735M  24% /boot
tmpfs               tmpfs     363M     0  363M   0% /run/user/0
/dev/md0            ext4      9.8G   24K  9.3G   1% /mnt/RAID10
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 20:03:10 2026
        Raid Level : raid10
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Mar 15 20:14:32 2026
             State : clean 
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : cf703021:18553a15:0f539169:4ac6d863
            Events : 106

    Number   Major   Minor   RaidDevice State
       0     259        3        0      active sync set-A   /dev/nvme0n2
       1     259        4        1      active sync set-B   /dev/nvme0n3
       2     259        5        2      active sync set-A   /dev/nvme0n4
       3     259        6        3      active sync set-B   /dev/nvme0n5

```

## RAID5 阵列实验+备份盘

```bash
[root@localhost ~]# mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/nvme0n2 /dev/nvme0n3 /dev/nvme0n4 /dev/nvme0n5
# 三块盘加上一块备份盘（-x）
To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: /dev/nvme0n4 appears to contain an ext2fs file system
mdadm: size=5242880K  mtime=Sun Mar 15 16:08:20 2026
mdadm: size set to 5237760K
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[root@localhost ~]# mkfs.ext4 /dev/md0
mke2fs 1.46.5 (30-Dec-2021)
/dev/md0 contains a ext4 file system
	last mounted on Sun Mar 15 20:14:31 2026
Proceed anyway? (y,N) y
Creating filesystem with 2618880 4k blocks and 655360 inodes
Filesystem UUID: 4662308d-912b-4b49-998a-cd340fe8fc85
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

[root@localhost ~]# mkdir /mnt/RAID5
[root@localhost ~]# mount /dev/md0 /mnt/RAID5
[root@localhost ~]# df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs               tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs               tmpfs     726M  9.1M  717M   2% /run
/dev/mapper/rl-root xfs        17G  1.4G   16G   9% /
/dev/nvme0n1p1      xfs       960M  226M  735M  24% /boot
tmpfs               tmpfs     363M     0  363M   0% /run/user/0
/dev/md0            ext4      9.8G   24K  9.3G   1% /mnt/RAID5
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 20:39:11 2026
        Raid Level : raid5
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Mar 15 20:40:04 2026
             State : clean 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 2dd83710:1964221c:183becdd:be0eba1a
            Events : 23

    Number   Major   Minor   RaidDevice State
       0     259        3        0      active sync   /dev/nvme0n2
       1     259        4        1      active sync   /dev/nvme0n3
       4     259        5        2      active sync   /dev/nvme0n4

       3     259        6        -      spare   /dev/nvme0n5        # 备份盘
[root@localhost ~]# mdadm /dev/md0 -f /dev/nvme0n2  
# 模拟/dev/nvme0n2设备损坏
[root@localhost ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Mar 15 20:39:11 2026
        Raid Level : raid5
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sun Mar 15 20:43:13 2026
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 2dd83710:1964221c:183becdd:be0eba1a
            Events : 47

    Number   Major   Minor   RaidDevice State
       3     259        6        0      active sync   /dev/nvme0n5  # 备用盘顶上
       1     259        4        1      active sync   /dev/nvme0n3
       4     259        5        2      active sync   /dev/nvme0n4

       0     259        3        -      faulty   /dev/nvme0n2

```

