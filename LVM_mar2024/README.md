# ALUG - Logical Volume Manager (LVM) - March 2024
This presentation is scheduled for March 7th, 2024.

### To Follow Along:
1. [Install Vagrant](https://developer.hashicorp.com/vagrant/install)
2. [Install Virtualbox](https://www.virtualbox.org/wiki/Linux_Downloads)
3. Download or copy the [Vagrantfile](https://github.com/shawnmcgraw/akronlinux/blob/main/LVM_mar2024/Vagrantfile)
4. Set environment variable: `export VAGRANT_EXPERIMENTAL=disks`
5. To start the VM: `vagrant up`
6. Make sure the VM is running `vagrant status` should show `running (virtualbox)`
7. Access the VM: `vagrant ssh`

### Cleaning up:
1. `vagrant halt` to shutdonw the VM
2. `vagrant destroy` to delete the VM and created drives

### Links to more on LVM:
- [Arch Wiki](https://wiki.archlinux.org/title/LVM)
- [Red Hat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_logical_volumes/index)
- [Linux Handbook](https://linuxhandbook.com/lvm-guide/)
- [Wikipedia](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))