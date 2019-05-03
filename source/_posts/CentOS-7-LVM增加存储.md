---
title: CentOS 7 LVM增加存储
date: 2019-05-03 21:39:50
tags:
---



# 介绍

本文介绍CentOS 7 LVM扩展的步骤



# 前置条件

硬盘已加到服务器上，可以用`fdisk -l` 看到

```bash
[root@localhost ~]# fdisk -l

Disk /dev/sdc: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000ed875

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048    15626239     7812096   83  Linux
/dev/sda2        15626240    19531775     1952768   82  Linux swap / Solaris
```



yum 需先安装lvm2，可以使用root账号执行 vgdisplay 测试是否命令有

```bash
[root@localhost ~]# vgdisplay
-bash: vgdisplay: command not found

[root@localhost ~]# yum install lvm2
[root@localhost ~]# vgdisplay
```



# 操作步骤

1. 对新硬盘进行分区，此处以sdb为例

   ```
   [root@localhost ~]# fdisk /dev/sdb
   Welcome to fdisk (util-linux 2.23.2).
   
   Changes will remain in memory only, until you decide to write them.
   Be careful before using the write command.
   
   Device does not contain a recognized partition table
   Building a new DOS disklabel with disk identifier 0x60d028f4.
   
   Command (m for help): p
   
   Disk /dev/sdb: 2147 MB, 2147483648 bytes, 4194304 sectors
   Units = sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   Disk label type: dos
   Disk identifier: 0x60d028f4
   
      Device Boot      Start         End      Blocks   Id  System
   
   Command (m for help): n
   Partition type:
      p   primary (0 primary, 0 extended, 4 free)
      e   extended
   Select (default p): p
   Partition number (1-4, default 1): 1
   First sector (2048-4194303, default 2048):
   Using default value 2048
   Last sector, +sectors or +size{K,M,G} (2048-4194303, default 4194303):
   Using default value 4194303
   Partition 1 of type Linux and of size 2 GiB is set
   
   Command (m for help): t
   Selected partition 1
   Hex code (type L to list all codes): 8e
   Changed type of partition 'Linux' to 'Linux LVM'
   
   Command (m for help): p
   
   Disk /dev/sdb: 2147 MB, 2147483648 bytes, 4194304 sectors
   Units = sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   Disk label type: dos
   Disk identifier: 0x60d028f4
   
      Device Boot      Start         End      Blocks   Id  System
   /dev/sdb1            2048     4194303     2096128   8e  Linux LVM
   
   Command (m for help): w
   The partition table has been altered!
   
   Calling ioctl() to re-read partition table.
   Syncing disks.
   ```

2. 重新读取分区表

   ```bash
   [root@localhost ~]# partprobe
   Warning: Unable to open /dev/sr0 read-write (Read-only file system).  /dev/sr0 has been opened read-only.
   ```

3. 创建物理卷

   ```bash
   [root@localhost ~]# pvcreate /dev/sdb1
     Physical volume "/dev/sdb1" successfully created.
   [root@localhost ~]# pvdisplay
     "/dev/sdb1" is a new physical volume of "<2.00 GiB"
     --- NEW Physical volume ---
     PV Name               /dev/sdb1
     VG Name
     PV Size               <2.00 GiB
     Allocatable           NO
     PE Size               0
     Total PE              0
     Free PE               0
     Allocated PE          0
     PV UUID               SyGUPc-ruAh-S111-wBPS-ZMPd-Kt3z-vhEqtc
   ```

4. 验证是否已有创建卷组Volume Group，没有才需执行里面的创建

   ```bash
   [root@localhost ~]# vgdisplay
   
   # 如有什么信息都没有，则需执行vgcreate
   [root@localhost ~]# vgcreate centos /dev/sdb1
     Volume group "centos" successfully created
   [root@localhost ~]# vgdisplay
     --- Volume group ---
     VG Name               centos
     System ID
     Format                lvm2
     Metadata Areas        1
     Metadata Sequence No  1
     VG Access             read/write
     VG Status             resizable
     MAX LV                0
     Cur LV                0
     Open LV               0
     Max PV                0
     Cur PV                1
     Act PV                1
     VG Size               <2.00 GiB
     PE Size               4.00 MiB
     Total PE              511
     Alloc PE / Size       0 / 0
     Free  PE / Size       511 / <2.00 GiB
     VG UUID               cbeUUT-hUwi-zx16-LWWa-xhpx-2FR0-RjKsSF
   ```

5. 如果执行了上面的vgcreate步骤将sdb1将入到了卷组，则跳过这一步。这一步是将新PV扩容到已有的VG了

   ```
   vgextend centos /dev/sdb1
   
   [root@localhost ~]# vgdisplay
   # 未验证此步骤，结果应该和第4步的一样
   ```

6. 创建逻辑卷LV

   ```bash
   [root@localhost ~]# lvcreate -L 1G -n data centos
     Logical volume "data" created.
   [root@localhost ~]# lvdisplay
     --- Logical volume ---
     LV Path                /dev/centos/data
     LV Name                data
     VG Name                centos
     LV UUID                FbGmDa-0Gmz-jiCA-YL3O-blGa-C0NL-eEQiMS
     LV Write Access        read/write
     LV Creation host, time localhost.localdomain, 2019-05-03 22:40:44 +0800
     LV Status              available
     # open                 0
     LV Size                1.00 GiB
     Current LE             256
     Segments               1
     Allocation             inherit
     Read ahead sectors     auto
     - currently set to     8192
     Block device           253:0
   ```

7. 将剩余的硬盘都分配给data 的LV， vgdisplay的Free PE可以看到容量

   ```bash
   [root@localhost ~]# vgdisplay
     --- Volume group ---
     VG Name               centos
     System ID
     Format                lvm2
     Metadata Areas        1
     Metadata Sequence No  2
     VG Access             read/write
     VG Status             resizable
     MAX LV                0
     Cur LV                1
     Open LV               0
     Max PV                0
     Cur PV                1
     Act PV                1
     VG Size               <2.00 GiB
     PE Size               4.00 MiB
     Total PE              511
     Alloc PE / Size       256 / 1.00 GiB
     Free  PE / Size       255 / 1020.00 MiB
     VG UUID               cbeUUT-hUwi-zx16-LWWa-xhpx-2FR0-RjKsSF
   
   
   [root@localhost ~]# lvextend -l +100%FREE /dev/centos/data
     Size of logical volume centos/data changed from 1.00 GiB (256 extents) to <2.00 GiB (511 extents).
     Logical volume centos/data successfully resized.
     
     
     
   [root@localhost ~]# vgdisplay
     --- Volume group ---
     VG Name               centos
     System ID
     Format                lvm2
     Metadata Areas        1
     Metadata Sequence No  3
     VG Access             read/write
     VG Status             resizable
     MAX LV                0
     Cur LV                1
     Open LV               0
     Max PV                0
     Cur PV                1
     Act PV                1
     VG Size               <2.00 GiB
     PE Size               4.00 MiB
     Total PE              511
     Alloc PE / Size       511 / <2.00 GiB
     Free  PE / Size       0 / 0
     VG UUID               cbeUUT-hUwi-zx16-LWWa-xhpx-2FR0-RjKsSF
   
   ```

8. 格式化逻辑卷LV

   ```bash
   [root@localhost ~]# mkfs.xfs /dev/c
   cdrom            centos/          char/            console          core             cpu/             cpu_dma_latency  crash
   [root@localhost ~]# mkfs.xfs /dev/centos/data
   meta-data=/dev/centos/data       isize=512    agcount=4, agsize=130816 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=0, sparse=0
   data     =                       bsize=4096   blocks=523264, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal log           bsize=4096   blocks=2560, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   
   ```

9. 挂载

   ```bash
   [root@localhost ~]# mkdir /data
   [root@localhost ~]# mount -t xfs /dev/centos/data /data
   [root@localhost ~]# df -Th
   Filesystem              Type      Size  Used Avail Use% Mounted on
   /dev/sda1               xfs       7.5G  1.2G  6.4G  16% /
   devtmpfs                devtmpfs  1.9G     0  1.9G   0% /dev
   tmpfs                   tmpfs     1.9G     0  1.9G   0% /dev/shm
   tmpfs                   tmpfs     1.9G  8.6M  1.9G   1% /run
   tmpfs                   tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
   tmpfs                   tmpfs     379M     0  379M   0% /run/user/0
   /dev/sr0                iso9660    11G   11G     0 100% /media/cdrom
   /dev/mapper/centos-data xfs       2.0G   33M  2.0G   2% /data
   ```

   开机自动挂载，使用vi或者其他文本编辑器修改/etc/fstab，按下面的例子加入最后一行

   ```bash
   [root@localhost ~]# vi /etc/fstab
   
   #
   # /etc/fstab
   # Created by anaconda on Fri May  3 21:15:56 2019
   #
   # Accessible filesystems, by reference, are maintained under '/dev/disk'
   # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
   #
   UUID=ecea6b8c-d12d-498f-b5d4-a52c83779a0d /                       xfs     defaults        0 0
   UUID=d7d65d97-2f52-4a01-9996-5e2caca3ace0 swap                    swap    defaults        0 0
   
   /dev/centos/data        /data   xfs     defaults        0       0
   ```

   测试下

   ```
   [root@localhost ~]# mount -fav
   /                        : ignored
   swap                     : ignored
   /data                    : already mounted
   ```



## 扩容加入其他硬盘

1. 格式化及创建分区

   ```bash
   [root@localhost ~]# fdisk /dev/sdc
   Welcome to fdisk (util-linux 2.23.2).
   
   Changes will remain in memory only, until you decide to write them.
   Be careful before using the write command.
   
   Device does not contain a recognized partition table
   Building a new DOS disklabel with disk identifier 0x723419e7.
   
   Command (m for help): p
   
   Disk /dev/sdc: 2147 MB, 2147483648 bytes, 4194304 sectors
   Units = sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   Disk label type: dos
   Disk identifier: 0x723419e7
   
      Device Boot      Start         End      Blocks   Id  System
   
   Command (m for help): n
   Partition type:
      p   primary (0 primary, 0 extended, 4 free)
      e   extended
   Select (default p): p
   Partition number (1-4, default 1): 1
   First sector (2048-4194303, default 2048):
   Using default value 2048
   Last sector, +sectors or +size{K,M,G} (2048-4194303, default 4194303):
   Using default value 4194303
   Partition 1 of type Linux and of size 2 GiB is set
   
   Command (m for help): t
   Selected partition 1
   Hex code (type L to list all codes): 8e
   Changed type of partition 'Linux' to 'Linux LVM'
   
   Command (m for help): p
   
   Disk /dev/sdc: 2147 MB, 2147483648 bytes, 4194304 sectors
   Units = sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   Disk label type: dos
   Disk identifier: 0x723419e7
   
      Device Boot      Start         End      Blocks   Id  System
   /dev/sdc1            2048     4194303     2096128   8e  Linux LVM
   
   Command (m for help): w
   The partition table has been altered!
   
   Calling ioctl() to re-read partition table.
   Syncing disks.
   
   
   
   [root@localhost ~]# partprobe
   Warning: Unable to open /dev/sr0 read-write (Read-only file system).  /dev/sr0 has been opened read-only.
   ```

2. 创建物理卷PV

   ```
   [root@localhost ~]# pvcreate /dev/sdc1
     Physical volume "/dev/sdc1" successfully created.
   [root@localhost ~]# pvdisplay
     --- Physical volume ---
     PV Name               /dev/sdb1
     VG Name               centos
     PV Size               <2.00 GiB / not usable 3.00 MiB
     Allocatable           yes (but full)
     PE Size               4.00 MiB
     Total PE              511
     Free PE               0
     Allocated PE          511
     PV UUID               SyGUPc-ruAh-S111-wBPS-ZMPd-Kt3z-vhEqtc
   
     "/dev/sdc1" is a new physical volume of "<2.00 GiB"
     --- NEW Physical volume ---
     PV Name               /dev/sdc1
     VG Name
     PV Size               <2.00 GiB
     Allocatable           NO
     PE Size               0
     Total PE              0
     Free PE               0
     Allocated PE          0
     PV UUID               vj4VhM-trI4-qQRc-Do6Y-F72T-dgtX-4c7VjS
   ```

3. 将新PV加入到VG

   ```
   [root@localhost ~]# vgdisplay
     --- Volume group ---
     VG Name               centos
     System ID
     Format                lvm2
     Metadata Areas        1
     Metadata Sequence No  3
     VG Access             read/write
     VG Status             resizable
     MAX LV                0
     Cur LV                1
     Open LV               1
     Max PV                0
     Cur PV                1
     Act PV                1
     VG Size               <2.00 GiB
     PE Size               4.00 MiB
     Total PE              511
     Alloc PE / Size       511 / <2.00 GiB
     Free  PE / Size       0 / 0
     VG UUID               cbeUUT-hUwi-zx16-LWWa-xhpx-2FR0-RjKsSF
     
     [root@localhost ~]# vgextend centos /dev/sdc1
     Volume group "centos" successfully extended
     
     [root@localhost ~]# vgdisplay
     --- Volume group ---
     VG Name               centos
     System ID
     Format                lvm2
     Metadata Areas        2
     Metadata Sequence No  4
     VG Access             read/write
     VG Status             resizable
     MAX LV                0
     Cur LV                1
     Open LV               1
     Max PV                0
     Cur PV                2
     Act PV                2
     VG Size               3.99 GiB
     PE Size               4.00 MiB
     Total PE              1022
     Alloc PE / Size       511 / <2.00 GiB
     Free  PE / Size       511 / <2.00 GiB
     VG UUID               cbeUUT-hUwi-zx16-LWWa-xhpx-2FR0-RjKsSF
   ```

4. 扩容LV

   ```bash
   [root@localhost ~]# lvextend -l +100%FREE /dev/centos/data
     Size of logical volume centos/data changed from <2.00 GiB (511 extents) to 3.99 GiB (1022 extents).
     Logical volume centos/data successfully resized.
   [root@localhost ~]# lvdisplay
     --- Logical volume ---
     LV Path                /dev/centos/data
     LV Name                data
     VG Name                centos
     LV UUID                FbGmDa-0Gmz-jiCA-YL3O-blGa-C0NL-eEQiMS
     LV Write Access        read/write
     LV Creation host, time localhost.localdomain, 2019-05-03 22:40:44 +0800
     LV Status              available
     # open                 1
     LV Size                3.99 GiB
     Current LE             1022
     Segments               2
     Allocation             inherit
     Read ahead sectors     auto
     - currently set to     8192
     Block device           253:0
   
   [root@localhost ~]# df -Th
   Filesystem              Type      Size  Used Avail Use% Mounted on
   /dev/sda1               xfs       7.5G  1.2G  6.4G  16% /
   devtmpfs                devtmpfs  1.9G     0  1.9G   0% /dev
   tmpfs                   tmpfs     1.9G     0  1.9G   0% /dev/shm
   tmpfs                   tmpfs     1.9G  8.6M  1.9G   1% /run
   tmpfs                   tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
   /dev/mapper/centos-data xfs       2.0G   33M  2.0G   2% /data
   tmpfs                   tmpfs     379M     0  379M   0% /run/user/0
   ```

5. 在线调整/data大小

   ```bash
   [root@localhost ~]# df -Th
   Filesystem              Type      Size  Used Avail Use% Mounted on
   /dev/sda1               xfs       7.5G  1.2G  6.4G  16% /
   devtmpfs                devtmpfs  1.9G     0  1.9G   0% /dev
   tmpfs                   tmpfs     1.9G     0  1.9G   0% /dev/shm
   tmpfs                   tmpfs     1.9G  8.6M  1.9G   1% /run
   tmpfs                   tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
   /dev/mapper/centos-data xfs       2.0G   33M  2.0G   2% /data
   tmpfs                   tmpfs     379M     0  379M   0% /run/user/0
   [root@localhost ~]# xfs_growfs /dev/centos/data
   meta-data=/dev/mapper/centos-data isize=512    agcount=4, agsize=130816 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0
   data     =                       bsize=4096   blocks=523264, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=2560, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 523264 to 1046528
   [root@localhost ~]# df -Th
   Filesystem              Type      Size  Used Avail Use% Mounted on
   /dev/sda1               xfs       7.5G  1.2G  6.4G  16% /
   devtmpfs                devtmpfs  1.9G     0  1.9G   0% /dev
   tmpfs                   tmpfs     1.9G     0  1.9G   0% /dev/shm
   tmpfs                   tmpfs     1.9G  8.6M  1.9G   1% /run
   tmpfs                   tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
   /dev/mapper/centos-data xfs       4.0G   33M  4.0G   1% /data
   tmpfs                   tmpfs     379M     0  379M   0% /run/user/0
   ```

   