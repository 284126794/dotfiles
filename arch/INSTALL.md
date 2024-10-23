Archlinux+btrfs+grub(with Windows)+nvidia

# ISO download,update the date 
https://mirror.nju.edu.cn/archlinux/iso/2024.10.01/

# set the font
setfont ter-132b

# connect to the internet
1. ip link
2. wire/wireless(iwctl)
3. ping baidu.com  

# INSTALL_GUAIDE_OFIICIAL
https://wiki.archlinux.org/title/installation_guide

# (optional) check the efi version,this is based on the 64
    cat /sys/firmware/efi/fw_platform_size

# timedate control
    timedatectl
    timedatectl set-timezone Asia/Shanghai
# with Windows
    timedatectl status
    sudo timedatectl set-local-rtc 1 --adjust-system-clock
    sudo timedatectl set-ntp true
    timedatectl status

# Partition the disks
## check the block
    fdisk -l
### if the log is too long
    fdisk /dev/to_yours

## justify the partitions
    cfdisk /dev/to_yours
# if use the WINDOWS EFI,the EFI partition can just be ignored now
# swap needs to be 8GB or 16GB
# one can divide the / and /home, but with btrfs, it can be set use the subvolume,so just make it one.

## format the partitions
    mkswap /dev/swap_partition
### btrfs
    mkfs.btrfs /dev/root_partition -f
#### set the subvolume
    mount /dev/root_partition /mnt
    btrfs subvolume create /mnt/@
    btrfs subvolume create /mnt/@home
    umount /dev/root_partition

## mount the partitions
    mount /dev/root_partition /mnt -o subvol=@
    mount /dev/root_partition /mnt/home -o subvol=@home --mkdir
# check the efi partition
    mount /dev/efi_system_partition /mnt/boot/efi --mkdir
    swapon /dev/swap_partition


# pacman_config
## mirror set for pacman
    # if the pacman cannot install the first 
    sudo pacman-key --init
    sudo pacman-key --populate

## keyring-CERNET-CHINA
    sudo vim /etc/pacman.conf

[archlinuxcn]
Server = https://mirrors.cernet.edu.cn/archlinuxcn/$arch

    sudo pacman -Sy archlinux-keyring

### if the archlinux-keyring broke
sudo pacman-key --lsign-key "farseerfc@archlinux.org"

## mirrorlist
### if the relector can be normally run
    reflector --country China --age 24 --sort rate --protocol https --save /etc/pacman.d/mirrorlist
### else use the CERNET mirror
### or just add it to the top after.
    sudo vim /etc/pacman.d/mirrorlist
    Server = https://mirrors.cernet.edu.cn/archlinux/$repo/os/$arch
### uncomment the Color and ParallelDownloads lines
    pacman -Syyu

# INSTALL the Arch
    pacstrap -K /mnt base base-devel linux linux-firmware linux-headers git zsh grub efibootmgr os-prober openssl networkmanager dhcpcd neovim btrfs linux-headers

## mount
    genfstab -U /mnt >> /mnt/etc/fstab
    arch-chroot /mnt
## timedate
    ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
    hwclock --systohc
## language
    nvim /etc/locale.gen
### 取消en_US.UTF-8和zh_CN.UTF-8前的注释
    locale-gen
    nvim /etc/locale.conf
### 第一行写入LANG=en_US.UTF-8
    
## network
    nvim /etc/hostname
### hostname depends on you        
    systemctl enable dhcpcd
    systemctl enable NetworkManager
##  Initramfs配置
    nvim /etc/mkinitcpio.conf
### 在HOOKS中加入btrfs
    pacman -S btrfs-progs
### 上面这条命令会自动运行mkinitcpio -P
### 如果在pacstrap中就安装了btrfs-progs，那么改完/etc/mkinitcpio.conf后需要手动运行mkinitcpio -P
    
# User config
1. set the root passwd
    passwd
2. add the user
    useradd -m -G wheel <username>
    passwd <username>
3. add the sudo to the wheel
    sudo nvim /etc/sudoers
## uncomment the line before wheel:  # %wheel ALL=CALL:ALL) ALL    
4. su username

## terminal config
    whereis zsh / fish / bash
    chsh -s /usr/bin/fish(or other)
    Ctrl+D
    su <username>
    
# grub config
    sudo grub-install --recheck /dev/你的硬盘
    sudo nvim /etc/default/grub
## 将最后一行的注释去掉，启用os-prober检测双系统
## mount,and sudo os-prober看看能不能检测到windows
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    Ctrl+D 退出登陆
    umount -R /mnt 取消挂载
    reboot 重启



# Nvidia驱动
    lspci -k | grep -A 2 -E "(VGA|3D)"
    sudo pacman -S nvidia-dkms nvidia-utils nvidia-settings
    sudo pacman -S nvidia nvidia-utils nvidia-settings
    sudo nvim /etc/default/grub
## 在GRUB_CMDLINE_LINUX中添加nvidia_drm.modeset=1
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    sudo nvim /etc/mkinitcpio.conf
## 在MODULES中加入nvidia nvidia_modeset nvidia_uvm nvidia_drm
## 将kms从HOOKS中去掉
    sudo mkinitcpio -P
    reboot 
    nvidia-smi
在/etc/pacman.d/hooks/nvidia.hook中写入
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=linux

# Change the linux part above if a different kernel is used

[Action]
Description=Update NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux*) exit 0; esac; done; /usr/bin/mkinitcpio -P'


# Hyprland            
    sudo pacman -S hyprland kitty waybar
    sudo pacman -S sddm
    sudo pacman -S ttf-jetbrains-mono-nerd adobe-source-han-sans-cn-fonts adobe-source-code-pro-fonts
    sudo systemctl enable sddm
    reboot 重启
# Nvidia还要设置一下
# https://wiki.hyprland.org/Nvidia/
    CTRL+ALT+F3进如tty3并登陆用户
    nvim ~/.config/hypr/hyprland.conf
# 添加NVIDIA环境变量
    env = LIBVA_DRIVER_NAME,nvidia
    env = XDG_SESSION_TYPE,wayland
    env = GBM_BACKEND,nvidia-drm
    env = __GLX_VENDOR_LIBRARY_NAME,nvidia
    env = WLR_NO_HARDWARE_CURSORS,1
    reboot 重启
    Win+Q 开启终端
    Win+C 关闭窗口
    Win+R 呼出菜单
    Win+数字 切换桌面
    Win+Shift+数字 将当前窗口移动到对应工作区
    Win+鼠标左键 拖动窗口
    Win+鼠标右键 调整窗口大小
    Win+V 让窗口浮动出来
    （安装OBS中。。。。）

    安装输入法
        sudo pacman -S fcitx5 fcitx5-chinese-addons fcitx5-configtool
    配置输入法
        fcitx5-configtool
    安装paru
        git clone https://aur.archlinux.org/paru.git
        cd paru
        makepkg -si
    安装rofi
        sudo pacman -S rofi
    安装chrome和vscodfe
        paru -S google-chrome
        sudo pacman -S code
    配置
        获取我的配置文件
            git clone https://github.com/HeaoYe/config
            cd config
        配置neovim
            cp nvim ~/.config/ -r
        配置chrome和vscode
            cp *.conf ~/.config/
            code
            安装插件
            clangd CMake CmakeTools CodeLLDB GruvboxTheme
            cp settings.json ~/.config/Code\ -\ OSS/User/
        配置kitty
            cp kitty ~/.config/ -r
            在~/.config/kitty/kitty.conf 中设置字体大小
        配置hyprland
            sudo pacman -S xorg-xrdb
            cp hypr ~/.config/ -r
        配置waybar
            cp waybar ~/.config/ -r
            手动启动waybar查看效果
            waybar -c ~/.config/waybar/Waybar-3.0/config -s ~/.config/waybar/Waybar-3.0/style.css
            除了第一次配置，以后会自动启动waybar
        配置声音
            sudo pacman -S pavucontrol-qt
            pavucontrol-qt
        （可选）
        安装nvtop
            sudo pacman -S nvtop
        安装grim，截图软件
            sudo pacman -S grim
        安装QQ音乐
            （我装过了，不演示了）
            paru -S qqmusic-bin
        安装大鹅，我也不知道这个软件能干什么
            paru -S daed-git
        没啥要安装的了，arch配置成功
        hyprland和waybar和kitty配置文件是clone的别人的
        链接放简介里


