# Logical Volume Manager - ALUG, March 2024

Intall LVM2 package: `dnf install lvm2`


### Warning: These steps are destructive! Make sure you're on a VM, not your PC.

-----------
## Working with Physical Volumes (PV)
Common commands when working with PVs:
- `pvscan`
- `pvs`
- `pvdisplay`
- `pvremove`

Create Physical Volume on a disk without first creating a partition: `pvcreate /dev/sdb`

Create first of two partitions of equal size on a disk: `fdisk /dev/sdc`

1. Create GPT partition table: `g` 
2. Create new partition: `n`
3. Partition number: leave blank for default value
4. First sector: leave blank for default value
5. Last sector: `+2.5G`
6. Write to the disk: `w`

Create second partition `fdisk /dev/sdc`
1. Create new partition: `n`
2. Partition number: leave blank
3. First sector: leave blank
4. Last sector: leave blank
5. Write to the disk: `w`

Create two physical volumes on /dev/sdc at once:
```
[root@aluglvm ~]# pvcreate /dev/sdc1 /dev/sdc2
  Physical volume "/dev/sdc1" successfully created.
  Physical volume "/dev/sdc2" successfully created.
```
Verify the new PVs: `pvs`
```
[root@aluglvm ~]# pvs
  PV         VG Fmt  Attr PSize  PFree 
  /dev/sdb      lvm2 ---   5.00g  5.00g
  /dev/sdc1     lvm2 ---   2.50g  2.50g
  /dev/sdc2     lvm2 ---  <2.50g <2.50g
```
Remove a PV:
```
[root@aluglvm ~]# pvremove /dev/sdc2
  Labels on physical volume "/dev/sdc2" successfully wiped.
```
Check it:
```
[root@aluglvm ~]# pvs
  PV         VG Fmt  Attr PSize PFree
  /dev/sdb      lvm2 ---  5.00g 5.00g
  /dev/sdc1     lvm2 ---  2.50g 2.50g
```
## Working with Volume Groups (VG)
Common commands when working with VGs:
- `vgs`
- `vgcreate`
- `vgrename`
- `vgscan`
- `vgdisplay`

Create new VG spanning two PVs:
```
[root@aluglvm ~]# vgcreate vg00 /dev/sdb /dev/sdc1
  Volume group "vg00" successfully created
```
Look at some basic details of the new VG:
```
[root@aluglvm ~]# vgs
  VG   #PV #LV #SN Attr   VSize VFree
  vg00   2   0   0 wz--n- 7.49g 7.49g
```
Moar details!
```
[root@aluglvm ~]# vgdisplay
  --- Volume group ---
  VG Name               vg00
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
  VG Size               7.49 GiB
  PE Size               4.00 MiB
  Total PE              1918
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1918 / 7.49 GiB
  VG UUID               WkgN6y-b6AK-qfgA-Afxb-S0MW-8wYB-RlNl0c
  ```
List PVs associated with a sepcific VG:
```
[root@aluglvm ~]# pvdisplay -S vgname=vg00 -C -o pv_name
  PV        
  /dev/sdb  
  /dev/sdc1
```
Extend a VG while adding a PV at the same time:
```
[root@aluglvm ~]# vgextend vg00 /dev/sdc2
  Physical volume "/dev/sdc2" successfully created.
  Volume group "vg00" successfully extended
```
Now `vg00` extends accross three PVs:
```
[root@aluglvm ~]# pvdisplay -S vgname=vg00 -C -o pv_name
  PV        
  /dev/sdb  
  /dev/sdc1 
  /dev/sdc2 
```
If a VG has no active logical volumes, you can reduce the VG:
```
[root@aluglvm ~]# vgreduce vg00 /dev/sdb
  Removed "/dev/sdb" from volume group "vg00"
```
Verify:
```
[root@aluglvm ~]# pvdisplay -S vgname=vg00 -C -o pv_name
  PV        
  /dev/sdc1 
  /dev/sdc2
```
Not going to do this yet, but it is possible to remove a VG: `vgremove vg00`
## Working with Logical Volumes (LV)
Logical Volumes, LVs, are what you will mostly work with. LVs are like partitions, but instead of existing on a raw disk, it exists within a volume group (VG).

LVs can be formatted with filesystems and can be mounted, just like a partition.

Common commands when working with LVs:
- `lvs`
- `lvcreate`
- `lvreduce`
- `lvrename`
- `lvdisplay`

Create a logical volume: `lvcreate -L <size> -n <lv_name> <vg_name>`
```
[root@aluglvm ~]# lvcreate -L 1G -n lv_alpha vg00
  Logical volume "lv_alpha" created.
```
Look at some details:
```
[root@aluglvm ~]# lvs
  LV       VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_alpha vg00 -wi-a----- 1.00g 
```
Moar details:
```
[root@aluglvm ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg00/lv_alpha
  LV Name                lv_alpha
  VG Name                vg00
  LV UUID                evPn7v-LWva-CRdC-LOJC-vJLO-xzz3-I4cSZk
  LV Write Access        read/write
  LV Creation host, time aluglvm, 2024-03-07 22:48:06 +0000
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```
Lets make a filesystem:
```
[root@aluglvm ~]# mkfs.ext4 /dev/vg00/lv_alpha
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: c094a167-dd49-43c2-af9b-bef0244984cf
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```
Mount the new filesystem: 
```
[root@aluglvm ~]# mount -t ext4 /dev/vg00/lv_alpha /mnt
```
Verify:
```
[root@aluglvm ~]# lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                 8:0    0 19.5G  0 disk 
├─sda1              8:1    0    1M  0 part 
├─sda2              8:2    0  200M  0 part /boot/efi
├─sda3              8:3    0  512M  0 part /boot
└─sda4              8:4    0 18.8G  0 part /
sdb                 8:16   0    5G  0 disk 
sdc                 8:32   0    5G  0 disk 
├─sdc1              8:33   0  2.5G  0 part 
│ └─vg00-lv_alpha 253:0    0    1G  0 lvm  /mnt
└─sdc2              8:34   0  2.5G  0 part 
sdd                 8:48   0    5G  0 disk 
```
Resize the VG:

First get free space. There are a couple options
1. Just `vgs`
```
[root@aluglvm ~]# vgs
  VG   #PV #LV #SN Attr   VSize VFree
  vg00   2   1   0 wz--n- 4.99g 3.99g
```
2. Just get the free space from `vgs`
```
[root@aluglvm ~]# vgs -S vgname=vg00 -o vg_free
  VFree
  3.99g
```
Extend the LV to the maximum available and resize the volume at the same time:
```
[root@aluglvm ~]# lvextend -l +100%FREE /dev/vg00/lv_alpha -r
  Size of logical volume vg00/lv_alpha changed from 1.00 GiB (256 extents) to 4.99 GiB (1278 extents).
  File system ext4 found on vg00/lv_alpha mounted at /mnt.
  Extending file system ext4 to 4.99 GiB (5360320512 bytes) on vg00/lv_alpha...
resize2fs /dev/vg00/lv_alpha
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/vg00/lv_alpha is mounted on /mnt; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg00/lv_alpha is now 1308672 (4k) blocks long.

resize2fs done
  Extended file system ext4 on vg00/lv_alpha.
  Logical volume vg00/lv_alpha successfully resized.
```
The `-r` in that command does the same as `resize2fs /dev/vg00/lv_alpha`

### Not all filesystems support hot resizing! EXT4 and XFS are ones I know of that do.
