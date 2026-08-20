do as root obviously as easier + UUID to fstab requires
make lvm = 
```
sudo lvcreate -L 3G -n var ubuntu-vg (exchange var to whatever and size)
```

makefs with
mkfs.ext4 /dev/ubuntu-vg/X

mount to /mnt/x{1-3} with

check it fits with
```
sudo du -sh /{var,var/log,var/tmp}
```


then
```
sudo rsync -aHAXS --numeric-ids \
    --exclude='/log/***' \
    --exclude='/tmp/***' \
    /var/ /mnt/v1/
```

to copy to /var without /log and /tmp since mounted separately
-a preserve perms
-H preserve Hlinks
-A preserve ACLs
-X preserve extended attributes (i guess like sticky bit etc?)
-S handle sparce files seperately
numeric ids is for UID and GID numbers
and exclude is obvious

then do same with /var/log and /var/tmp
mount to test with mount /dev/ubuntu-vg/var /var and follow with others

grab uuids with
```
sudo blkid | grep var | awk '{print $2}' >> /etc/fstab
```

make /var/log and /var/tmp directories to mount to:
```
mkdir -vp /var/{log,tmp}
```

then add to fstab the ext4 defaults,nosuid,noexec,nodev 0 2 (where appropriate)



add to fstab:
```
tmpfs /tmp tmpfs defaults,nosuid,nodev,noexec,size=1G 0 0
```
do:
```
sudo mount -o remount /tmp
```

then with boot, cant remove since added to fstab on creation:

add - nodev,nosuid,noexec to end of boot, ansible? no idea

do home with

create dir /mnt/h
sudo lvcreate -L {x}G -n home ubuntu-vg
sudo mkfs.ext4 /dev/ubuntu-vg/home
sudo mount /dev/ubuntu-vg/home /mnt/h
rsync -aHAXS --numeric-ids /home /mnt/h
sudo su root
sudo blkid | grep h | awk '{print $2}' >> /etc/fstab
mount /dev/ubuntu-vg/home /home

then remove /mnt/v1/v2/v3 /mnt/{others}
with 
```
umount /mnt/*
rm -rfd /mnt
```

reboot + pray

run this do nuke the old stuff if reboot successful
```
mkdir -p /mnt/rootfs && mount --bind / /mnt/rootfs && rm -rf /mnt/rootfs/home/* /mnt/rootfs/home/.[!.]* /mnt/rootfs/home/..?* /mnt/rootfs/var/* /mnt/rootfs/var/.[!.]* /mnt/rootfs/var/..?* && umount /mnt/rootfs && rmdir /mnt/rootfs
```

