# Set up a second HDD like the default one in Proxmox.

This tutorial shows how to set up an extra harddisk in Promox, and split it in storage and a thinlvm pool.

## Format the disk, create a physical volume and volume group.
Find the new drive with fdisk:
```bash
fdisk -l
```

Create a partition on the new drive:
```bash
fdisk /dev/sdb
```

Validate the partition is there:
```bash
fdisk -l
```

Create a physical volume on the new partition:
```bash
pvcreate /dev/sdb1
```

Create a new volume group:
```bash
vgcreate hdd2 /dev/sdb1
```

Check if the volume group has been created:
```bash
vgdisplay
```

## Create a storage directory (backups, ISO images).

Create a new logical volume for the backup in the new group:
```bash
lvcreate -n hdd2_data -L 300G hdd2 
```

Create a file system on the data lv:
```bash
mkfs.ext4 /dev/mapper/hdd2_data
```

Mount the data directory:
```
mount /dev/mapper/hdd2_data /mnt/hdd2_data
```

To automount the drive, find the UUID:
```bash
blkid /dev/mapper/hdd2_data
```

Append the following in /etc/fstab:
```
UUID=<UUID> /mnt/hdd2_data ext4 defaults 0 2
```

You can now add the drive via the UI: Data Center->Storage->Add->Directory. 

As the directory fill in the mount point.

## Create a lvm-thin pool (running VM's)
Create a new logical volume, for example with the remaining disk space
```
lvcreate -l100%FREE -n hdd2_thin-lvm hdd2
```
*sometimes 100% does not work, in that case 99% probably will.

Convert the logical volume into a thin-lvm:
```bash
lvconvert --type thin-pool hdd2_lvmthin
```

You can add more metadata storage if required, the default might be only a few megabytes:
```
lvextend --poolmetadatasize +1G hdd2_lvmthin
```

You can now add the drive via the UI: Data Center->Storage->Add->LVM-Thin. 

## More reading:
- [How to Add New Disks Using LVM to an Existing Linux System](https://www.tecmint.com/add-new-disks-using-lvm-to-linux/)
- [LFCS: How to Manage and Create LVM Using vgcreate, lvcreate and lvextend Commands](https://www.tecmint.com/manage-and-create-lvm-parition-using-vgcreate-lvcreate-and-lvextend/)
- [Adding disk as LVM to proxmox](https://forum.proxmox.com/threads/adding-a-disk-and-set-it-as-lvm-thin-help-needed-please.111724/)
