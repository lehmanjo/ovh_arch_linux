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
