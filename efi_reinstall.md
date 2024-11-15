# mkfs.fat -F32 /dev/nvme0n1p1              #格式化efi分区
mount /dev/nvme0n1p2 /mnt -o subvol=@                #挂载系统分区
mount /dev/nvme0n1p2 /mnt/home -o subvol=@home --mkdir
mount /dev/nvme0n1p6 /mnt/efi            #挂载efi分区
swapon /dev/nvme0n1p1
arch-chroot /mnt
pacman -S grub efibootmgr
pacman -Rs os-prober                      #卸载冲突包
grub-install --recheck /dev/nvme0n1 --efi-directory=/efi/
ls /efi/                                  #检查文件
initramfs-linux.img,intel-ucode.img,vmlinuz-linux
pacman -Sy linux                          #有就略过
grub-mkconfig -o /efi/grub/grub.cfg      #生成配置
exit
reboot
