---
layout: post
title:  "Git commands I forget"
date:   2023-04-10 13:00:00 +0000
---
Noting the commands used to set up a LUKS encrypted drive. Following https://docs.fedoraproject.org/en-US/quick-docs/encrypting-drives-using-LUKS/#_what_is_block_device_encryption

``` sh
lsblk
```
To work out the device name.

``` sh
cryptsetup luksFormat <device>
```
Run `cryptsetup` as `root`.
``` sh
cryptsetup luksUUID <device>
```
Get the UUID.
``` sh
cryptsetup luksOpen <device> <name>
```
Open the LUKS device and map it to `/dev/mapper/<name>`
``` sh
mkfs.btrfs -L "<label>" /dev/mapper/<name>
```
Make a filesystem.
``` sh
vi /etc/crypttab
```
Make an entry in `crypttab` to map the `/dev/mapper/<name>` to the LUKS UUID at boot.
``` sh
vi /etc/fstab
```
Take care getting this entry right, a mistake will prevent the system from booting. Use either `/dev/mapper/<name>` for the device or UUID where the UUID comes from `blkid`.
