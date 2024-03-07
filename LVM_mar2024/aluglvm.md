%title: Logical Volume Manager (LVM) - ALUG
%author: Shawn McGraw
%date: 2024-03-07

-> # LVM <-

--------------------------------------------

-> # What is LVM? <-

From Wikipedia:

Logical Volume Manager (LVM) is a device mapper framework that provides logical volume management for the Linux kernel.

--------------------------------------------

-> # Why use LVM? <-

The main advantage of LVM is how easy it is to resize a logical volume or volume group. It abstracts away all the ugly parts (partitions, raw disks) and leaves us with a central storage pool to work with.

If you've ever experienced the horror of partition resizing, you'd wanna use LVM.

Pros:

- LVM gives you more flexibility than just using normal hard drive partitions
- Have logical volumes stretched over several disks (RAID, mirroring, striping which offer advantages such as additional resilliance and performance)
- Create small logical volumes and resize them "dynamically" as they get filled up
- Resize logical volumes regardless of their order on disk. It does not depend on the position of the LV within VG, there is no need to ensure surrounding available space
- Resize/create/delete logical and physical volumes online. File systems on them still need to be resized, but some (such as Ext4 and Btrfs) support online resizing
- Online/live migration of LV (or segments) being used by services to different disks without having to restart services
- Snapshots allow you to backup a frozen copy of the file system, while keeping service downtime to a minimum and easily merge the snapshot into the original volume later
- Support for unlocking separate volumes without having to enter a key multiple times on boot (LUKS)
- Built-in support for caching of frequently used data (lvmcache)

Cons:

- Additional steps in setting up the system (may require changes to mkinitcpio configuration), more complicated. Requires (multiple) daemons to constantly run.

--------------------------------------------

-> # Basic components of LVM <-

- Physical Volumes (PV)
- Volume Groups (VG)
- Logical Volumes (LV)


