---
layout: post
published: true
author: Alexandre Dumont
last_modified_at: '2018-11-08 21:40 +0100'
title: Migrate your Dropbox folder from ZFS to an ext4 ZVOL
---
This month, Dropbox has dropped support for non-ext4 filesystems on Linux. On my Linux computer, my Dropbox folder was on a ZFS filesystem... If you want to keep Dropbox syncing, they force you to move back to Ext4... non-sense, but I didn't find any other alternative. So what I did is create a ZVOL (volume), format it as Ext4 and move my Dropbox content to it.

## Create a Dropbox ZVOL

Here I create a 27GB ZVOL called `dropbox` on my `zpool` zpool, format it as Ext4 and mount it _temporarily_ as `/mnt`:

```nosynthax
# zfs create -s -V 27G zpool/dropbox
# mkfs.ext4 /dev/zvol/zpool/dropbox
# mount /dev/zvol/zpool/dropbox /mnt
# chown adumont.adumont /mnt
```
Notice the # prompt: I run these commands as **root**.

## Copy Dropbox content

Here I stop dropbox, and copy all the content of my Dropbox folder to /mnt:

```nosynthax
$ dropbox stop
$ rsync -avnx /home/adumont/Dropbox/ /mnt/
```
Notice the $ prompt: I run these commands as my non-root user.

## Remount the Dropbox ZVOL in place

Now I unmount /mnt, I delete my Dropbox folder (no problem, we copied it to the zvol), and I create the mountpoint:

```nosynthax
# umount /mnt
# rm -rf /home/adumont/Dropbox
# mkdir /home/adumont/Dropbox
```
Then I add the new filesystem (dropbox zvol) to my /etc/fstab and mount it:

```nosynthax
# echo /dev/zvol/zpool/dropbox /home/adumont/Dropbox ext4 rw,relatime,stripe=2,data=ordered 0 0 >> /etc/fstab
# mount /home/adumont/Dropbox
```

## Dropbox on ZVOL

Last step I start Dropbox, and it will pick the new filesystem as Ext4 (on a Zvol):

```nosynthax
$ dropbox stop
```

Enjoy!
