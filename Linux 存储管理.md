# Linux 存储管理



## 硬盘的两种形态

机械硬盘（HDD）和 固态硬盘（SSD）

## 硬盘设备命名

 /dev/sd[a-z] → 第一块硬盘是 sda，第二块 sdb 

/dev/nvme0n1 → NVMe SSD（云服务器常见）

## 分区表

MBR（传统）

GPT（现代）

## 管理分区常用工具

lsblk→列出块设备

fdisk → MBR 分区（<2TB） 

gdisk → GPT 分区（>2TB、推荐） 

parted → 支持 MBR+GPT，适合脚本、非交互

partprobe→ 重新读取内核分区表（分区后常用）

## parted

- 操作命令：

```bash
cp [FROM-DEVICE] FROM-MINOR TO-MINOR           #将文件系统复制到另一个分区 
help [COMMAND]                                 #打印通用求助信息，或关于 COMMAND 的信息 
mklabel 标签类型                               #创建新的磁盘标签 (分区表) 
mkfs MINOR 文件系统类型                        #在 MINOR 创建类型为“文件系统类型”的文件系统 
mkpart 分区类型 [文件系统类型] 起始点 终止点   #创建一个分区 
mkpartfs 分区类型 文件系统类型 起始点 终止点   #创建一个带有文件系统的分区 
move MINOR 起始点 终止点                       #移动编号为 MINOR 的分区 
name MINOR 名称                                #将编号为 MINOR 的分区命名为“名称” 
print [MINOR]                                  #打印分区表，或者分区 
quit                                           #退出程序 
rescue 起始点 终止点                           #挽救临近“起始点”、“终止点”的遗失的分区 
resize MINOR 起始点 终止点                     #改变位于编号为 MINOR 的分区中文件系统的大小 
rm MINOR                                       #删除编号为 MINOR 的分区 
select 设备                                    #选择要编辑的设备 
set MINOR 标志 状态                            #改变编号为 MINOR 的分区的标志
```

### 常用的命令

- 查看分区情况

```bash
[root@localhost ~]# parted /dev/nvme0n1 print
```

- 设置磁盘的分区表

```bash
[root@localhost ~]# parted /dev/nvme0n2 mklabel gpt
```

- 对磁盘进行分区

```bash
[root@localhost ~]# parted /dev/nvme0n2 mkpart primary 1 1G
# 创建1个G大小的分区
```

- 删除分区

```bash
[root@localhost ~]# parted /dev/nvme0n2 rm 1
```

- 修改磁盘为mbr分区，注意会丢失所有数据

```bash
[root@localhost ~]# parted /dev/nvme0n2 mklabel msdos
警告: The existing disk label on /dev/nvme0n2 will be destroyed and all data on this disk will be lost. Do
you want to continue?
是/Yes/否/No? yes
```

## fdisk 

管理磁盘中MBR分区

```
fdisk [磁盘名称]
```

fdisk命令中的参数以及作用

| 参数 | 作用                   |
| ---- | ---------------------- |
| m    | 查看全部可用的参数     |
| n    | 添加新的分区           |
| d    | 删除某个分区信息       |
| l    | 列出所有可用的分区类型 |
| t    | 改变某个分区的类型     |
| p    | 查看分区信息           |
| w    | 保存并退出             |
| q    | 不保存直接退出         |

添加一个磁盘分区

```bash
[root@localhost ~]# fdisk /dev/nvme0n2

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x447f44d1.

Command (m for help): n                    #添加新的分区
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): +1G                     #输入新的分区大小

Created a new partition 1 of type 'Linux' and of size 1 GiB.

Command (m for help): p                    #查看分区信息
Disk /dev/nvme0n2: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x447f44d1

Device         Boot Start     End Sectors Size Id Type
/dev/nvme0n2p1       2048 2099199 2097152   1G 83 Linux

Command (m for help): w                    #保存
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@localhost ~]# lsblk                  #列出块设备
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  1.7G  0 rom
nvme0n1     259:0    0   20G  0 disk
├─nvme0n1p1 259:1    0    1G  0 part /boot
└─nvme0n1p2 259:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2     259:3    0    5G  0 disk
└─nvme0n2p1 259:8    0    1G  0 part
nvme0n3     259:4    0    5G  0 disk
nvme0n4     259:5    0    5G  0 disk
nvme0n5     259:6    0    5G  0 disk
nvme0n6     259:7    0    5G  0 disk
```

## gdisk

管理磁盘中的GPT分区

使用方式和选项与fdisk中几乎一致



## 存储结构与磁盘划分



- **/**（根目录） 整个文件系统的起点，所有目录都从这里开始。

- **/boot** 开机所需文件：内核文件（vmlinuz）、initramfs、GRUB 引导加载器、开机菜单和配置文件等。 （开机第一步加载的东西）

- **/dev** 以文件形式存放所有设备和接口（设备文件）。 如 /dev/sda（硬盘）、/dev/sda1（分区）、/dev/null（黑洞）、/dev/zero（零设备）等。

- **/etc** 系统配置文件存放地，几乎所有服务、系统设置都在这里。 如 /etc/passwd、/etc/fstab、/etc/sysconfig、/etc/yum.repos.d 等。

- **/home** 普通用户的家目录。 每个用户有自己的子目录（如 /home/user1），存放个人文件、配置、下载等。

- **/bin** 基本命令存放目录（单用户模式下仍可使用）。 如 ls、cat、cp、mv、rm、ps、kill 等常用命令。

- **/lib** 和 **/lib64** 系统启动和运行命令所需的共享库（函数库）。 /bin 和 /sbin 下的命令依赖这里的库文件。

- **/sbin** 系统管理员命令（开机过程和维护用）。 如 fdisk、ifconfig、reboot、fsck、mkfs 等。

- **/media** 用于临时挂载可移动设备（如 U 盘、光盘、移动硬盘）的目录。 系统自动挂载时常用这里。

- **/mnt** 手动挂载临时文件系统的目录（自定义挂载点）。 如 mount /dev/sdb1 /mnt/data

- **/opt** 第三方软件安装目录。 如自己编译安装的软件、Oracle、Google Chrome 等。

- **/root** root 用户的家目录（超级管理员的个人文件存放地）。

- **/srv** 服务相关数据目录。 如网站数据、FTP 文件、版本控制仓库等（较少使用）。

- **/tmp** 临时目录，所有用户可写。 系统重启后通常清空（存放临时文件、下载缓存等）。

- **/proc** 虚拟文件系统（内存中的伪文件系统）。 存放系统内核、进程、硬件、网络状态等实时信息（如 /proc/cpuinfo、/proc/meminfo、/proc/net/dev）。

- **/sys** 类似 /proc，也是虚拟文件系统。 主要存放内核设备、驱动、电源管理等信息（sysfs）。

- **/usr**

  用户程序和数据目录（用户相关的大部分内容）。 子目录包括：

  - /usr/bin：普通用户命令
  - /usr/sbin：系统管理命令
  - /usr/lib：库文件
  - /usr/share：共享数据（文档、图标、帮助文件）
  - /usr/local：用户自行编译安装的软件（优先级高于系统默认）

- **/var** 经常变化的文件目录（variable）。 如日志（/var/log）、邮件（/var/mail）、数据库（/var/lib/mysql）、缓存（/var/cache）、运行时数据（/var/run）等。

- **/lost+found** 文件系统错误修复时存放丢失文件片段的目录。 通常在 fsck 修复后出现，正常情况下为空。

### 快速记忆

- **启动相关**：boot（内核引导）
- **设备相关**：dev（所有硬件文件）
- **配置相关**：etc（所有配置文件）
- **用户相关**：home（个人家目录）、root（超级用户家目录）
- **命令相关**：bin（基本命令）、sbin（系统命令）
- **库相关**：lib（函数库）
- **临时相关**：tmp（临时文件）、var（变化文件，如日志）
- **共享相关**：usr（用户程序）、opt（第三方软件）
- **虚拟相关**：proc（进程/内核信息）、sys（设备驱动）



## 物理设备的命名规则

常见的硬件设备及其文件名称

| 硬件设备      | 文件名称           |
| ------------- | ------------------ |
| IDE设备       | /dev/hd[a-d]       |
| SCSI/SATA/U盘 | /dev/sd[a-p]       |
| 软驱          | /dev/fd[0-1]       |
| 打印机        | /dev/lp[0-15]      |
| 光驱          | /dev/cdrom         |
| 鼠标          | /dev/mouse         |
| 磁带机        | /dev/st0或/dev/ht0 |

## 文件系统与数据资料

**日志式文件系统 **和 **索引式文件系统**

日志式文件系统（ext4/xfs）通过“预写日志”保证断电后快速恢复和一致性，是现代 Linux 默认和推荐选择； 索引式文件系统（ext2）直接写数据，性能高但崩溃恢复慢，已基本被淘汰。

## 格式化

格式化（Formatting）在 Linux 存储管理中，指的是在分区完成后，对分区创建文件系统（File System）的过程。

### 格式化的核心作用

1. 创建文件系统的结构：
   - superblock（超级块）：记录整个文件系统的总体信息（总块数、inode 数、块大小等）
   - inode 表：存放文件元数据（权限、所有者、大小、时间戳、数据块位置等），一个 inode 记录多个 block 号（直接/间接指针），记录满了再新建 inode 扩展。
   - 数据块（block）：存放实际文件内容
   - 日志区（journal）：日志式文件系统有，用于记录操作日志
2. 初始化元数据：
   - 清空或初始化 inode 和 block 位图，表示整个分区空间可用。（日志式文件系统）
   - 先清空或重置 **第一个索引节点**（inode 0），表示空间可用。（索引式文件系统）
   - 创建根目录（/）的 inode 和基本结构
3. 擦除旧数据（可选）：
   - 格式化通常会清空分区内容（但不一定彻底擦除数据，数据恢复工具可能找回）

### mkfs工具

Linux mkfs（英文全拼：make file system）命令用于在特定的分区上建立 linux 文件系统。

**实例：使用parted分区，然后使用mkfs创建ext4文件系统**

```bash
[root@localhost ~]# parted /dev/nvme0n3 mklabel gpt
[root@localhost ~]# parted /dev/nvme0n3 mkpart primary 1 1G
Information: You may need to update /etc/fstab.
[root@localhost ~]# parted /dev/nvme0n3 print
Model: VMware Virtual NVMe Disk (nvme)
Disk /dev/nvme0n3: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size   File system  Name     Flags
 1      1049kB  1000MB  999MB               primary

[root@localhost ~]# mkfs.ext4 /dev/nvme0n3p1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 243968 4k blocks and 61056 inodes
Filesystem UUID: db15a3c8-ae5d-4e69-aa93-cc0ee4107547
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@localhost ~]# parted /dev/nvme0n3 print
Model: VMware Virtual NVMe Disk (nvme)
Disk /dev/nvme0n3: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size   File system  Name     Flags
 1      1049kB  1000MB  999MB  ext4         primary
```

## mount

挂载文件系统

```bash
[root@localhost ~]#mount 文件系统 挂载目录
```

此为临时挂载系统，系统在重启后会失效

如果想让硬件设备和目录永久地进行自动关联，就必须把挂载信息按照指定的填写格式“设备文件 挂载目录 格式类型 权限选项 是否备份 是否自检”,写入到**/etc/fstab**文件中。

用于挂载信息的指定填写格式中，各字段所表示的意义

| 字段     | 意义                                                         |
| -------- | ------------------------------------------------------------ |
| 设备文件 | 一般为设备的路径+设备名称，也可以写唯一识别码（UUID，Universally Unique Identifier） |
| 挂载目录 | 指定要挂载到的目录，需在挂载前创建好                         |
| 格式类型 | 指定文件系统的格式，比如Ext3、Ext4、XFS、SWAP、iso9660（此为光盘设备）等 |
| 权限选项 | 若设置为defaults，则默认权限为：rw, suid, dev, exec, auto, nouser, async |
| 是否备份 | 若为1则开机后使用dump进行磁盘备份，为0则不备份               |
| 是否自检 | 若为1则开机后自动进行磁盘自检，为0则不自检                   |

- 实例，挂载分区`/dev/sdb1`到`/mnt/volume1`下，并且设置为永久自动挂载

```bash
[root@localhost ~]# mkdir -p /mnt/volume1
[root@localhost ~]# mount /dev/sdb1 /mnt/volume1
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   47G  995M   46G    3% /
devtmpfs                 979M     0  979M    0% /dev
tmpfs                    991M     0  991M    0% /dev/shm
tmpfs                    991M  8.5M  982M    1% /run
tmpfs                    991M     0  991M    0% /sys/fs/cgroup
/dev/sda1               1014M  133M  882M   14% /boot
tmpfs                    199M     0  199M    0% /run/user/0
/dev/sdb1                9.1G   37M  8.6G    1% /mnt/volume1
# 先卸载sdb1
[root@localhost ~]# umount /dev/sdb1
[root@localhost ~]# vim /etc/fstab
# 最后一行加上
/dev/sdb1 /mnt/volume1 ext4 defaults 0 0

[root@localhost ~]# umount /dev/sdb1
# 卸载sdb1，用mount -a 挂载/etc/fstab
[root@localhost ~]# mount -a
# 测试是否正确配置，fstab上的设备挂一遍试试看，万一错了重启系统会失败。
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   47G  995M   46G    3% /
devtmpfs                 979M     0  979M    0% /dev
tmpfs                    991M     0  991M    0% /dev/shm
tmpfs                    991M  8.5M  982M    1% /run
tmpfs                    991M     0  991M    0% /sys/fs/cgroup
/dev/sda1               1014M  133M  882M   14% /boot
tmpfs                    199M     0  199M    0% /run/user/0
/dev/sdb1                9.1G   37M  8.6G    1% /mnt/volume1

[root@localhost ~]# reboot
[root@localhost ~]# df -h
#重启看看是否已挂载/dev/sdb1
```



| 参数 | 作用                                 |
| ---- | ------------------------------------ |
| -a   | 挂载所有在/etc/fstab中定义的文件系统 |
| -t   | 指定文件系统的类型                   |

## umount

卸载已经挂载的设备文件`umount [挂载点/设备文件]`

```bash
[root@node-1 ~]# umount /dev/sda2
```

挂载点和挂在设备的关系:



物理磁盘 (/dev/nvme0n2, /dev/nvme0n3, ...)
        ↓
RAID 阵列 (/dev/md0)  ←────┐
        ↓                   │
文件系统 (ext4/xfs)                      │
        ↓                   │
挂载点 (/mnt/RAID10) ──────┘      通过这个访问



## blkid

查看磁盘的UUID（设备唯一标识符）

### df

查看磁盘空间使用情况的最常用工具（ disk free）

| 选项    | 含义                       | 示例          | 说明                                  |
| ------- | -------------------------- | ------------- | ------------------------------------- |
| -h      | human-readable（人类可读） | df -h         | 以 K/M/G/T 单位显示（最常用）         |
| -T      | 显示文件系统类型           | df -hT        | 显示 ext4/xfs 等类型                  |
| -t      | 只显示指定类型             | df -t xfs     | 只看 xfs 文件系统                     |
| -i      | 显示 inode 使用情况        | df -i         | 检查 inode 是否用尽（小文件多时有用） |
| --total | 显示总计行                 | df -h --total | 最后一行显示所有磁盘总和              |

```shell
[root@node-1 ~]#df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/rl-root      47G  1.5G   45G    4% /
devtmpfs                979M     0  979M    0% /dev
tmpfs                   991M  8.5M  982M    1% /dev/shm
tmpfs                   991M     0  991M    0% /run
/dev/nvme0n1p1         1014M  133M  882M   14% /boot
tmpfs                   199M     0  199M    0% /run/user/0
/dev/nvme0n2p1          5.0G   37M  4.9G    1% /mnt/newdisk
```

## du

查看某个目录下文件数据的占用量

```bash
[root@localhost ~]# du -sh /etc
21M	/etc
```

## 卸载已挂载的设备文件并删除磁盘分区

```shell
[root@localhost ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                726M  9.0M  717M   2% /run
/dev/mapper/rl-root   17G  1.4G   16G   9% /
/dev/nvme0n1p1       960M  226M  735M  24% /boot
/dev/nvme0n3p1       974M   24K  907M   1% /mnt/volume1 # 已挂载
tmpfs                363M     0  363M   0% /run/user/0
[root@localhost ~]# umount /dev/nvme0n3p1               # 卸载
[root@localhost ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                726M  9.0M  717M   2% /run
/dev/mapper/rl-root   17G  1.4G   16G   9% /
/dev/nvme0n1p1       960M  226M  735M  24% /boot
tmpfs                363M     0  363M   0% /run/user/0
[root@localhost ~]# vim /etc/fstab                # 删除配置文件中的挂载信息
[root@localhost ~]# fdisk /dev/nvme0n3            # 管理磁盘分区

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help): p                           # 查看分区信息

Disk /dev/nvme0n3: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9869515C-AC15-480B-8CCB-6A376399B7CE

Device         Start     End Sectors Size Type
/dev/nvme0n3p1  2048 2099199 2097152   1G Linux filesystem # 分区存在

Command (m for help): d                           # 删除分区
Selected partition 1
Partition 1 has been deleted.

Command (m for help): p                           
Disk /dev/nvme0n3: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9869515C-AC15-480B-8CCB-6A376399B7CE 
                                                           # 分区被删除
Command (m for help): w                            # 保存
The partition table has been altered.
```

## free

查看系统内存

**-h** ：以人类易读的形式显示

**-m** ：以MB为单位

**-g** ：以GB为单位

## mkswap

添加交换分区

## swapon

开启swap交换分区

```shell
[root@localhost ~]# mkswap /dev/nvme0n2p1
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=9d5158b3-9a30-4a0a-8167-7ee1bc9ce4f8
[root@localhost ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       448Mi       1.2Gi       6.0Mi       201Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@localhost ~]# swapon /dev/nvme0n2p1        # 挂载swap交换分区
[root@localhost ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           3.5Gi       510Mi       3.0Gi       9.0Mi       270Mi       3.0Gi
Swap:          7.0Gi          0B       7.0Gi     # /dev/nvme0n3的5G加入到swap里变成7G

[root@localhost ~]# vim /etc/fstab                # 写入fstab文件中永久挂载
#
# /etc/fstab
# Created by anaconda on Sat Nov  9 02:51:16 2024
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=12fcea99-d1db-4f0a-ad86-f03129024fdb /boot                   xfs     defaults        0 0
/dev/mapper/rl-swap     none                    swap    defaults        0 0
/dev/nvme0n2p1          swap                    swap    defaults        0 0

[root@localhost ~]# reboot
[root@localhost ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       451Mi       1.2Gi       6.0Mi       205Mi       1.3Gi
Swap:          3.0Gi          0B       3.0Gi
```

1.买一个磁盘，插入到服务器

2.我们可以对磁盘进行分区或不分区

3.格式化文件系统

4.挂载（临时挂载或写入/etc/fstab）



# 磁盘容量配额

##  实例切入

- 创建5个用户user1,user2,user3,user4,user5，密码和用户名相同，初始组为usergrp组。
- 5个用户都可以取得300M的磁盘使用空间，文件数量不限。超过250M,给于提示。



###  1. 准备磁盘

- 创建分区

```bash
[root@localhost ~]# fdisk /dev/nvme0n2

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x27819a3d.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): p
Disk /dev/nvme0n2: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x27819a3d

Device         Boot Start     End Sectors Size Id Type
/dev/nvme0n2p1       2048 4196351 4194304   2G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

- lsblk 查看当前分区情况

```bash
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  1.7G  0 rom
nvme0n1     259:0    0   20G  0 disk
├─nvme0n1p1 259:1    0    1G  0 part /boot
└─nvme0n1p2 259:2    0   19G  0 part
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2     259:3    0    5G  0 disk
└─nvme0n2p1 259:8    0    2G  0 part
nvme0n3     259:4    0    5G  0 disk
nvme0n4     259:5    0    5G  0 disk
nvme0n5     259:6    0    5G  0 disk
nvme0n6     259:7    0    5G  0 disk
```

- 格式化分区的文件系统

```bash
[root@localhost ~]# mkfs.ext4 /dev/nvme0n2p1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: da4f8f5c-6867-4cf8-8486-333a16adbcdb
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

- 创建目录并挂载

```bash
[root@localhost ~]# mkdir /mnt/mountpoint
[root@localhost ~]# mount /dev/nvme0n2p1 /mnt/mountpoint
[root@localhost ~]# df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs               tmpfs     872M     0  872M   0% /dev/shm
tmpfs               tmpfs     349M  6.3M  343M   2% /run
/dev/mapper/rl-root xfs        17G  1.7G   16G  10% /
/dev/nvme0n1p1      xfs       960M  261M  700M  28% /boot
tmpfs               tmpfs     175M     0  175M   0% /run/user/0
/dev/nvme0n2p1      ext4      2.0G   24K  1.8G   1% /mnt/mountpoint
```

### 2. 准备用户

```bash
[root@atopos ~]# setenforce 0
# 临时关闭SELinux
[root@atopos ~]# getenforce 
Permissive
[root@atopos ~]# groupadd usergrp
[root@atopos ~]# for i in {1..5}; do useradd -g usergrp -b /mnt/mountpoint user$i && echo user$i | passwd --stdin user$i; done
```

| 步骤 | 命令部分                                       | 作用                       |
| :--- | :--------------------------------------------- | :------------------------- |
| 1    | `for i in {1..5}`                              | 循环5次，i=1,2,3,4,5       |
| 2    | `useradd -g usergrp -b /mnt/mountpoint user$i` | 创建用户，指定组和基础目录 |
| 3    | `&&`                                           | 只有前命令成功才执行后面的 |
| 4    | `echo user$i | passwd --stdin user$i`          | 设置密码为用户名           |

###  3. 确保文件系统支持

- 检查挂载点是否支持quota配置

```bash
[root@localhost ~]# mount | grep mountpoint
/dev/nvme0n2p1 on /mnt/mountpoint type ext4 (rw,relatime,seclabel)
```

- 重新挂载，让文件系统支持quota配置

```bash
[root@localhost ~]# mount -o remount,usrquota,grpquota /mnt/mountpoint/
# mount -o 是挂载文件系统时指定挂载选项的参数，-o 是 --options 的缩写
[root@localhost ~]# mount | grep mountpoint
/dev/nvme0n2p1 on /mnt/mountpoint type ext4 (rw,relatime,seclabel,quota,usrquota,grpquota)
```

###  4. 安装 quota

```bash
[root@atopos ~]# yum install -y quota
```

###  5. 开启 quota

quotacheck主要参数

- -a：扫描所有在/etc/mtab内含有quota参数的文件系统
- -u：针对用户扫描文件与目录的使用情况，会新建一个aquota.user文件
- -g：针对用户组扫描文件与目录的使用情况，会新增一个aquota.group文件
- -v：显示扫描过程的信息

```bash
[root@localhost ~]# quotacheck -avug
quotacheck: Your kernel probably supports ext4 quota feature but you are using external quota files. Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated.
quotacheck: Scanning /dev/nvme0n2p1 [/mnt/mountpoint] done
quotacheck: Cannot stat old user quota file /mnt/mountpoint/aquota.user: No such file or directory. Usage will not be subtracted.
quotacheck: Cannot stat old group quota file /mnt/mountpoint/aquota.group: No such file or directory. Usage will not be subtracted.
quotacheck: Cannot stat old user quota file /mnt/mountpoint/aquota.user: No such file or directory. Usage will not be subtracted.
quotacheck: Cannot stat old group quota file /mnt/mountpoint/aquota.group: No such file or directory. Usage will not be subtracted.
quotacheck: Checked 8 directories and 15 files
quotacheck: Old file not found.
quotacheck: Old file not found.
[root@localhost ~]# quotaon -avug
quotaon: Your kernel probably supports ext4 quota feature but you are using external quota files. Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated.
/dev/nvme0n2p1 [/mnt/mountpoint]: group quotas turned on
/dev/nvme0n2p1 [/mnt/mountpoint]: user quotas turned on
```

### 6.  编辑配额配置

**edquota**

| 选项 | 说明                 | 示例                         |
| :--- | :------------------- | :--------------------------- |
| `-u` | 编辑用户配额（默认） | `edquota -u user1`           |
| `-g` | 编辑组配额           | `edquota -g usergrp`         |
| `-p` | 复制配额设置         | `edquota -p 源用户 目标用户` |
| `-t` | 编辑宽限期           | `edquota -t`                 |
| `-T` | 编辑用户的宽限期     | `edquota -T user1`           |

```bash
[root@localhost ~]# edquota -u user1
```

![image-20260311204259141](C:\Users\w1664\AppData\Roaming\Typora\typora-user-images\image-20260311204259141.png)

| **文件系统**     | /dev/nvme0n2p1 | 所在分区                                                     | 💾    |
| ---------------- | -------------- | ------------------------------------------------------------ | ---- |
| **已用块数**     | 300,000        | 已使用的磁盘块数量                                           | ⚠️    |
| **软限制**       | 250,000        | 当达到软限制时会提示用户，但仍允许用户在限定的额度内继续使用 | 🟡    |
| **硬限制**       | 300,000        | 当达到硬限制时会提示用户，且强制终止用户的操作               | 🔴    |
| **已用inodes**   | 4              | 已使用的文件节点数                                           | ✅    |
| **inodes软限制** | 0              | 文件数警告阈值（无限制）                                     | ⚪    |
| **inodes硬限制** | 0              | 文件数最大允许值（无限制）                                   | ⚪    |

- 可以将针对user1的限制复制给user2

```sh
[root@localhost ~]# edquota -p user1 -u user2
```



**repquota**

`repquota` 是 **report quota** 的缩写，用于**生成磁盘配额报告**。

- 查看限制情况

```bash
[root@localhost ~]# repquota -as
*** Report for user quotas on device /dev/nvme0n2p1
Block grace time: 7days; Inode grace time: 7days
                        Space limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --     20K      0K      0K              2     0     0
user1     --     16K    245M    293M              4     0     0
user2     --     16K    245M    293M              4     0     0
user3     --     16K      0K      0K              4     0     0
user4     --     16K      0K      0K              4     0     0
user5     --     16K      0K      0K              4     0     0
```

### 7. 测试

```bash
# user1用户测试
[root@localhost ~]# su - user1
[user1@localhost ~]$ dd if=/dev/zero of=bigfile bs=10M count=50
nvme0n2p1: warning, user block quota exceeded.
nvme0n2p1: write failed, user block limit reached.
dd: error writing 'bigfile': Disk quota exceeded
30+0 records in
29+0 records out
307179520 bytes (307 MB, 293 MiB) copied, 1.1834 s, 260 MB/s
[user1@localhost ~]$ du -sh
293M    .

# user2用户测试
[root@localhost ~]# su - user2
[user2@localhost ~]$ dd if=/dev/zero of=bigfile bs=10M count=50
nvme0n2p1: warning, user block quota exceeded.
nvme0n2p1: write failed, user block limit reached.
dd: error writing 'bigfile': Disk quota exceeded
30+0 records in
29+0 records out
307183616 bytes (307 MB, 293 MiB) copied, 1.43269 s, 214 MB/s
[user2@localhost ~]$ du -sh
293M    .
```

**dd if=/dev/zero of=bigfile bs=10M count=50 **  的说明

| 参数           | 说明                                                         | 含义                       |
| :------------- | :----------------------------------------------------------- | :------------------------- |
| `dd`           | 命令名称                                                     | 数据复制和转换工具         |
| `if=/dev/zero` | Input File (输入文件) = /dev/zero 是一个特殊的虚拟设备：  读取它时，永远返回 0 ，永远不会读完（无限供应），就像"永远流不完的零的水龙头" | 提供无限个零字节的虚拟设备 |
| `of=bigfile`   | of = Output File (输出文件) bigfile = 要创建的文件名         | 要创建的文件名             |
| `bs=10M`       | 块大小                                                       | 每次读写 10MB 数据         |
| `count=50`     | 块数量                                                       | 执行 50 次读写操作         |

## quota 命令

Linux quota命令用于显示磁盘已使用的空间与限制。

执行quota指令，可查询磁盘空间的限制，并得知已使用多少空间

选项：

- -g 列出群组的磁盘空间限制。
- -q 简明列表，只列出超过限制的部分。
- -u 列出用户的磁盘空间限制。
- -v 显示该用户或群组，在所有挂入系统的存储设备的空间限制。
- -V 显示版本信息。

#  软硬方式链接

在Linux系统中存在硬链接和软连接两种文件。

- 硬链接（hard link）：
  - 可以将它理解为一个“指向原始文件inode的指针”，系统不为它分配独立的inode和文件。所以，硬链接文件与原始文件其实是同一个文件，只是名字不同。我们每添加一个硬链接，该文件的inode连接数就会增加1；而且只有当该文件的inode连接数为0时，才算彻底将它删除。换言之，由于硬链接实际上是指向原文件inode的指针，因此即便原始文件被删除，依然可以通过硬链接文件来访问。需要注意的是，由于技术的局限性，我们不能跨分区对目录文件进行链接。
- 软链接（也称为符号链接[symbolic link]）：
  - 仅仅包含所链接文件的路径名，因此能链接目录文件，也可以跨越文件系统进行链接。但是，当原始文件被删除后，链接文件也将失效，从这一点上来说与Windows系统中的“快捷方式”具有一样的性质。

**一句话总结**：

“硬链接是多个文件名共享同一个 inode，删除一个不影响其他；软链接是一个独立文件，只存路径，源文件没了链接就失效。”

## ln

用于创建链接文件

```
ln [选项] 目标
```

ln命令中可用的参数以及作用

| 参数 | 作用                                               |
| ---- | -------------------------------------------------- |
| -s   | 创建“符号链接”（如果不带-s参数，则默认创建硬链接） |
| -f   | 强制创建文件或目录的链接                           |
| -i   | 覆盖前先询问                                       |
| -v   | 显示创建链接的过程                                 |

##  软链接演示

```shell
[root@localhost ~]# echo 'hello workld' >> testfile
[root@localhost ~]# ln -s /root/testfile softlink  # 创建软连接（写绝对路径）
[root@localhost ~]# ll
total 12
-rw-------.  1 root root  956 Mar  5 17:17 anaconda-ks.cfg
drwxr-xr-x.  2 root root    6 Mar 10 10:35 backup
drwxrwxrwx.  2 root root    6 Mar  5 21:36 dir
-rw-rw-rw-.  1 root root    0 Mar  5 21:36 file
-rw-------.  1 root root    0 Mar  9 17:01 nohup.out
drwxrwxrwt. 11 root root 4096 Mar 12 08:37 softfile
lrwxrwxrwx.  1 root root   14 Mar 12 09:24 softlink -> /root/testfile
-rw-r--r--.  1 root root   12 Mar 12 09:23 testfile
[root@localhost ~]# echo 123 >> softlink
[root@localhost ~]# cat testfile
hello workld
123
[root@localhost ~]# echo 321 >> testfile
[root@localhost ~]# cat softlink         # 修改 软链接 原文件会被修改，修改原文件 软链接文件 也会被修改
hello workld
123
321
[root@localhost ~]# mv softlink /root/dir/
[root@localhost ~]# cd /root/dir/
[root@localhost dir]# cat softlink       # 链接写成绝对路径，移动链接到不同目录都能找到原始文件
hello world
[root@localhost dir]# cd
[root@localhost ~]# rm -f testfile       # 删除原始文件后，链接文件也将失效
[root@localhost ~]# cd /root/dir/
[root@localhost dir]# cat softlink
cat: softlink: No such file or directory
[root@localhost dir]# ll
total 0
lrwxrwxrwx. 1 root root 14 Mar 12 09:24 softlink -> /root/testfile
[root@localhost ~]# ln -s / softlink     # 软连接文件也可以链接到目录
[root@localhost ~]# ll
total 8
-rw-------.  1 root root  956 Mar  5 17:17 anaconda-ks.cfg
drwxr-xr-x.  2 root root    6 Mar 10 10:35 backup
drwxrwxrwx.  2 root root   22 Mar 12 09:28 dir
-rw-rw-rw-.  1 root root    0 Mar  5 21:36 file
-rw-------.  1 root root    0 Mar  9 17:01 nohup.out
drwxrwxrwt. 11 root root 4096 Mar 12 08:37 softfile
lrwxrwxrwx.  1 root root    1 Mar 12 09:47 softlink -> /
```

##  硬链接演示

```shell
[root@localhost ~]# echo 'hello linux' >> testfile
[root@localhost ~]# ln testfile hardlink
[root@localhost ~]# ll
total 16
-rw-------.  1 root root  956 Mar  5 17:17 anaconda-ks.cfg
drwxr-xr-x.  2 root root    6 Mar 10 10:35 backup
drwxrwxrwx.  2 root root   22 Mar 12 09:28 dir
-rw-rw-rw-.  1 root root    0 Mar  5 21:36 file
-rw-r--r--.  2 root root   12 Mar 12 09:53 hardlink
-rw-------.  1 root root    0 Mar  9 17:01 nohup.out
drwxrwxrwt. 11 root root 4096 Mar 12 08:37 softfile
-rw-r--r--.  2 root root   12 Mar 12 09:53 testfile
[root@localhost ~]# ln testfile hardlink2
[root@localhost ~]# ll
total 20
-rw-------.  1 root root  956 Mar  5 17:17 anaconda-ks.cfg
drwxr-xr-x.  2 root root    6 Mar 10 10:35 backup
drwxrwxrwx.  2 root root   22 Mar 12 09:28 dir
-rw-rw-rw-.  1 root root    0 Mar  5 21:36 file
-rw-r--r--.  3 root root   12 Mar 12 09:53 hardlink
-rw-r--r--.  3 root root   12 Mar 12 09:53 hardlink2
-rw-------.  1 root root    0 Mar  9 17:01 nohup.out
drwxrwxrwt. 11 root root 4096 Mar 12 08:37 softfile
-rw-r--r--.  3 root root   12 Mar 12 09:53 testfile
# 我们每添加一个硬链接，这几个文件的inode连接数就会增加1
[root@localhost ~]# ls -i                # 查看inode编号  hardlink和hardlink2和testfile的inode编号市一样的，说明他们是同一个文件
34275099 anaconda-ks.cfg  51095495 dir   33686682 hardlink   33686336 nohup.out  33686682 testfile
17196932 backup           33686353 file  33686682 hardlink2  16777345 softfile
[root@localhost ~]# stat testfile
  File: testfile
  Size: 12        	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 33686682    Links: 3
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2026-03-12 09:53:46.057037356 +0800
Modify: 2026-03-12 09:53:46.057037356 +0800
Change: 2026-03-12 09:54:31.784513384 +0800
 Birth: 2026-03-12 09:53:46.057037356 +0800
[root@localhost ~]# echo 123 >> hardlink
[root@localhost ~]# cat testfile
hello linux
123
[root@localhost ~]# echo 321 >> hardlink2
[root@localhost ~]# cat testfile
hello linux
123
321
# 修改hardlink和hardlink2 都会对testfile修改
[root@localhost ~]# rm -f testfile
[root@localhost ~]# cat hardlink
hello linux
123
321
# 删除原文件，硬链接文件不会失效，他们可以通过保存的inode信息找到文件
```

## 跨文件链接

```bash
[root@localhost ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                1.8G     0  1.8G   0% /dev/shm
tmpfs                726M  9.1M  717M   2% /run
/dev/mapper/rl-root   17G  1.4G   16G   9% /
/dev/nvme0n1p1       960M  226M  735M  24% /boot
tmpfs                363M     0  363M   0% /run/user/0
/dev/nvme0n2p1       974M  294M  614M  33% /mnt/mountpoint
/dev/nvme0n2p2       974M   24K  907M   1% /mnt/mountpoint2
[root@localhost ~]# touch /mnt/mountpoint testfile
[root@localhost ~]# ln -s /mnt/mountpoint/testfile /mnt/mountpoint/softlink
[root@localhost ~]# ln /mnt/mountpoint/testfile /mnt/mountpoint2/hardlink
ln: failed to access '/mnt/mountpoint/testfile': No such file or directory
# 软连接可以 硬链接不行
```

