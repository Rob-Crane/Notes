# VirtualBox Notes

## Resizing a Dynamic VDI

* Resize the VDI using VirtualBox media manager
* Follow [these instructions](http://derekmolloy.ie/resize-a-virtualbox-disk/) to resize the partition using GParted
* After making the new SWAP partiion, need to update two places in OS.  Get UUID of new partition with `sudo blkid`
  * `/etc/fstab`: update with the new SWAP partition
  * `/etc/initramfs-tools/conf.d/resume/`: update `resume` variable with UUID of new partition
