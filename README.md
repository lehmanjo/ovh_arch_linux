# There is nothing useful here. 
Installed ArchLinux on OVH with software RAID but had to use remote console (KVM).



# Qemu prep

Create raw image files for QEMU
```
#!/bin/bash

DISKS=(ovhd1 64G ovhd2 64G)

for (( i=0; i < ${#DISKS[@]}; i+=2 ))
do
    disk=${DISKS[$i]}
    size=${DISKS[$i+1]}
    echo "Preparing disk: ${disk} of size ${size}"
    if [ ! -f ${disk} ]; then
        qemu-img create -f raw ${disk} ${size}
    else
        echo "Disk ${disk} already exists."
    fi
done
```

Partition template
```
$ cat disk.img.template
label: gpt
label-id: 929C7D56-D01B-D745-AC84-3C33C12E3738
device: ovhd
unit: sectors
first-lba: 2048
last-lba: 134217694
sector-size: 512

p1 : start=        2048, size=     1048576, type=A19D880F-05FC-4D3B-A006-743F0F84911E, uuid=E75EE1B3-FA45-8E45-AF3C-44641EF9EC04
p2 : start=     1050624, size=     2097152, type=0657FD6D-A4AB-43C4-84E5-0933C84B4F4F, uuid=9F906375-36CA-4D48-81E7-D1EE70006419
p3 : start=     3147776, size=   131069919, type=A19D880F-05FC-4D3B-A006-743F0F84911E, uuid=DC108A8A-F52B-664E-95B8-4679BED815AD
```

Apply partitioning
```
sfdisk ovhd1 < disk.img.template
sfdisk ovhd2 < disk.img.template
```

Check partition
```
$ fdisk -l ovhd1
Disk ovhd1: 64 GiB, 68719476736 bytes, 134217728 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 929C7D56-D01B-D745-AC84-3C33C12E3738

Device       Start       End   Sectors  Size Type
ovhd1p1       2048   1050623   1048576  512M Linux RAID
ovhd1p2    1050624   3147775   2097152    1G Linux swap
ovhd1p3    3147776 134217694 131069919 62.5G Linux RAID
```

Blah
```
modprobe nbd
qemu-nbd -f raw -c /dev/nbd0 ovhd1
qemu-nbd -f raw -c /dev/nbd1 ovhd2
qemu-nbd -c /dev/nbd2 Arch-Linux-x86_64-basic.qcow2
mkdir /mnt/ovhd1
mkdir /mnt/ovhd2
mkdir /mnt/ovhimg
mount /dev/nbd2p3 /mnt/ovhimg
```

Raid-1
```
mdadm --create --verbose /dev/md0 --level=1 --metadata=0.90 --raid-devices=2 /dev/nbd0p1 /dev/nbd1p1
mdadm --create --verbose /dev/md1 --level=1 --metadata=0.90 --raid-devices=2 /dev/nbd0p3 /dev/nbd1p3
cat /proc/mdstat
Personalities : [raid1]
md1 : active raid1 nbd1p3[1] nbd0p3[0]
      65534848 blocks [2/2] [UU]
      [=======>.............]  resync = 35.9% (23539008/65534848) finish=3.4min speed=201914K/sec

md0 : active raid1 nbd1p1[1] nbd0p1[0]
      524224 blocks [2/2] [UU]
```

Format and Mount
```
mkfs.ext4 /dev/md0
mkfs.ext4 /dev/md1
mkdir /mnt/ovhmd0
mkdir /mnt/ovhmd1
mount /dev/md0 /mnt/ovhmd0
mount /dev/md1 /mnt/ovhmd1
ls -l /mnt/ovhmd0
total 12
drwx------ 2 root root 12288 Aug 24 09:50 lost+found
ls -l /mnt/ovhmd1
total 16
drwx------ 2 root root 16384 Aug 24 09:50 lost+found
```

rsync options
```
rsync -avzHXShP

-a -> -rlptgoD (no -H,-A,-X)
-r recurse
-l copy symlinks
-p preseve perms
-t preserve times
-g preserve group
-o preserve owner
-D -> --devices --specials
--devices preserve device files
--specials preserve special files
-v verbose
-z compress
-H preserve hard links
-X preserve extended attributes
-S handle sparse files
-h human readable output
-P -> --partial --progress
--partial keep partially transferred files
--progress show progress during transfer
```

Copy
```
rsync -avzHXShP --exclude "/boot" /mnt/ovhimg/* /mnt/ovhmd1
rsync -avzHXShP /mnt/ovhimg/boot/* /mnt/ovhmd0
```

Unmount image
```
umount /mnt/ovhimg
qemu-nbd -d /dev/nbd2
```

Remount /boot
```
umount /mnt/ovhmd0
mkdir /mnt/ovhmd1/boot
mount /dev/md0 /mnt/ovhmd1/boot
ls -l /mnt/ovhmd1/boot/
total 13
drwxr-xr-x 3 root root  1024 Aug 15 20:04 boot
drwx------ 2 root root 12288 Aug 24 09:50 lost+found
```

Chroot 
```
arch-chroot /mnt/ovhmd1/

# Misc stuff
useradd -u 1028 -g 984 -m -s /bin/bash -d /home/me me
passwd -d root
yes mypass | passwd me
 New password: Retype new password: passwd: password updated successfully


# Network


# Grub



```



# OVH Arch Linux

- install Ubuntu 24 server with a /new 64GB partition (EXT4)
- https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux#From_a_host_running_another_Linux_distribution
- https://wiki.archlinux.org/title/Installation_guide#Install_essential_packages
- https://wiki.archlinux.org/title/GRUB
- https://bbs.archlinux.org/viewtopic.php?id=269955 (virt-customize)
- https://gitlab.archlinux.org/archlinux/arch-boxes/
- https://cloud-init.io/
- https://libguestfs.org/virt-customize.1.html
- https://wiki.archlinux.org/title/Dhcpcd


  
- Download boostrap ISO from https://geo.mirror.pkgbuild.com/iso/
- Extract: cd /new; tar xf /path-to-bootstrap-image/archlinux-bootstrap-x86_64.tar.zst --numeric-owner; mv <extracted directory>/* .; rmdir <extracted directory>
- Chroot: /new/bin/arch-chroot /new
- Initialize pacman
- pacman-key --init
- pacman-key --populate
> cat >> /etc/pacman.d/mirrorlist
> 
> Server = https://geo.mirror.pkgbuild.com/$repo/os/$arch
- pacman -Syyu
- Install Software
- pacman -S base linux linux-firmware grub vim openssh efibootmgr
- Add user
- useradd -u 1028 -g 984 -m -d /home/you you
- passwd you
- passwd root
- systemctl enable sshd
- su - you
- mkdir ~/.ssh
- chmod 0700 ~/.ssh
- cat > ~/.ssh/authorized_keys
- chmod 0600 ~/.ssh/authorized_keys
- exit (back to root)
- ln -sf /usr/share/zoneinfo/Hongkong /etc/localtime
- hwclock --systohc
- echo ovh01 > /etc/hostname
- locale-gen
> cat /etc/systemd/network/20-wired.network
> [Match]
> Name=eno1
> 
> [Network]
> Address=192.99.44.130/24
> Gateway=192.99.44.254
> DNS=1.1.1.1
- systemctl enable systemd-networkd.service
- pacman -S linux-firmware grub efibootmgr
- grub-mkconfig -o /boot/grub/grub.cfg
- exit to Ubuntu
- genfstab -U /new >> /new/etc/fstab
- cd /boot;rm -fr *
- cp -rp /new/boot/* /boot
- sync
- reboot
- 
  
# ovh_arch_linux
byolinux image for OVH


- https://wiki.archlinux.org/title/Arch_Linux_on_a_VPS
- https://help.ovhcloud.com/csm/en-dedicated-servers-bring-your-own-linux?id=kb_article_view&sysparm_article=KB0061610
- https://stackoverflow.com/questions/70351250/is-it-possible-to-copy-files-to-qemu-image-without-running-qemu
- https://github.com/ovh/bringyourownlinux
- https://cloudinit.readthedocs.io/en/22.1_a/topics/examples.html
  

- https://github.com/lehmanjo/ovh_arch_linux/raw/main/Arch-Linux-x86_64-cloudimg.qcow2
