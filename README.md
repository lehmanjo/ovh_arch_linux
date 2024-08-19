# OVH Arch Linux

- install Ubuntu 24 server with a /new 64GB partition (EXT4)
- https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux#From_a_host_running_another_Linux_distribution
- Download boostrap ISO from https://geo.mirror.pkgbuild.com/iso/
- Extract: cd /new; tar xf /path-to-bootstrap-image/archlinux-bootstrap-x86_64.tar.zst --numeric-owner; mv <extracted directory>/* .; rmdir <extracted directory>
- Chroot: /new/root.x86_64/bin/arch-chroot /new
- Initialize pacman
- pacman-key --init
- pacman-key --populate
- pacman -Syyu
- Install Software
- pacman -S base linux linux-firmware grub2 vim openssh
- Add user
- useradd -u 1028 -g 984 -h /home/you you
- passwd you
- passwd root
- 
  
# ovh_arch_linux
byolinux image for OVH


- https://wiki.archlinux.org/title/Arch_Linux_on_a_VPS
- https://help.ovhcloud.com/csm/en-dedicated-servers-bring-your-own-linux?id=kb_article_view&sysparm_article=KB0061610
- https://stackoverflow.com/questions/70351250/is-it-possible-to-copy-files-to-qemu-image-without-running-qemu
- https://github.com/ovh/bringyourownlinux
- https://cloudinit.readthedocs.io/en/22.1_a/topics/examples.html
  

- https://github.com/lehmanjo/ovh_arch_linux/raw/main/Arch-Linux-x86_64-cloudimg.qcow2
