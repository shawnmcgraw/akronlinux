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