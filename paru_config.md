# install 
sudo pacman -S --needed base-devel git vim
git clone https://aur.archlinux.org/paru.git
cd paru

# config the makepkg 
sudo vim /etc/makepkg.conf
# find the FLAGS and set 
MAKEFLAGS="-j$(nproc)"
makepkg -si
