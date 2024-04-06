# My Arch Install
Following the [arch wiki](https://wiki.archlinux.org/title/Installation_guide) guide.
## Live Environment Setup
```bash 
loadkeys sv-latin1 # keyboard
iwctl station wlan0 connect # if on wifi
ping archlinux.org -c 3 # test internet
timedatectl set-ntp true # fix systemclock if needed
Pacman -Sy # Refresh pacman repository
```
Optional: [install over ssh](https://wiki.archlinux.org/title/Install_Arch_Linux_via_SSH).

## Disk Setup
UEFI and encrypted BTRFS
### Partitioning
```bash 
gdisk /dev/sda # or whichever drive you will use
> n
> [default]
> [default]
> +350M
> ef00
> n # root partition use everything or give some space for SSD MAGIC
> [default]
> [default]
> [default] # OR -50G
> w
> Y
lsblk # verify partitioning
```
### Filesystems
```bash 
# EFI partition is fat32
mkfs.fat -F 32 /dev/sda1
# encrypted root volume
cryptsetup luksFormat /dev/sda2
> YES
> <password>
> <password>
cryptsetup luksOpen /dev/sda2 hemlig
> <password>
lsblk # verify again
ls /dev/mapper
mkfs.btrfs /dev/mapper/hemlig
# mount root partition as "root"
mount /dev/mapper/hemlig /mnt
# make subvolumes
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@var_log # or /mnt/@var
btrfs subvolume create /mnt/@swap
# need to unmount /mnt to mount subvolumes
umount /mnt
```

### Mounting
- noatime - dont write access times, but can break applications, use relatime
- compress - zstd is the middle one as for speed/compress ratio
- discard=async - optimized trimming
- space_cache=v2 https://wiki.tnonline.net/w/Btrfs/Space_Cache
```bash 
mount -o relatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@ /dev/mapper/hemlig /mnt

mkdir -p /mnt/{boot,home,var,swap,snapshots} # or just var depending on coice on line 45

mount -o relatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@home /dev/mapper/hemlig /mnt/home
mount -o relatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@snapshots /dev/mapper/hemlig /mnt/snapshots
mount -o relatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@var /dev/mapper/hemlig /mnt/var
mount -o relatime,ssd,space_cache=v2,subvol=@swap /dev/mapper/hemlig /mnt/swap
mount /dev/sda1 /mnt/boot
```
## Installation 
install base
```bash 
pacstrap /mnt base base-devel linux linux-firmware amd-ucode git helix fish btrfs-progs
```

Generate an fstab file (use -U or -L to define by UUID or labels, respectively):
```bash 
genfstab -U /mnt >> /mnt/etc/fstab
```
Change root into the new system:
```bash 
arch-chroot /mnt
```

Set the time zone:
```bash 
ln -sf /usr/share/zoneinfo/Europe/Stockholm /etc/localtime
```
Run hwclock(8) to generate /etc/adjtime:
```bash 
hwclock --systohc
```

Setup locale and keyboard
```bash 
# I uncomment sv_SE and en_GB
helix /etc/locale.gen
locale-gen
echo "LANG=en_GB.UTF-8" > /etc/locale.conf
echo "KEYMAP=sv-latin1" > /etc/vconsole.conf
echo "myhostname" > /etc/hostname
```


```bash 
helix /etc/hosts  
```
Copy and paste all that's below
```bash 
127.0.0.1       localhost
::1     localhost
127.0.1.1   myhostname.localhost myhostname
```
Create initial ramdisk environment
```bash 
mkinitcpio -p linux
# pacman -S dracut
# pacman -R mkinitcpio
# dracut --hostonly --no-hostonly-cmdline /boot/initramfs-linux.img
```
Enable systemd services
```bash 
systemctl enable fstrim.timer
systemctl enable NetworkManager
# if on virtualbox
# sudo pacman -S virtualbox-guest-utils
# systemctl enable vboxservice.service
```
Add a user
```bash 
useradd -m <username>
passwd <username>
usermod -aG wheel <username>
EDITOR=helix visudo
> uncomment wheel group
```

```bash 
exit
umount -R /mnt
reboot
```

Packages
```bash 
sudo pacman -S 
man-db man-pages \ # manpages
fish \ # shell
alacritty \ # terminal
plymouth \ # graphical boot, nice for decrypting hard drive
greed greetd-regreet \ # login manager (see setup if used)
xdg-desktop-portal-wlr xdg-desktop-portal-gtk xdg-user-dirs \ # needed by applications on wlroots
sway swaylock swayidle \ # window manager
j4-menu-desktop bemenu-wayland \ # launcher
bluez \ # bluetooth
xorg-wayland firefox gimp \ # will use eventually packages
imv \ # image viewer

```

AUR
```bash 
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si

## Configuration
- [Configure Plymouth](https://wiki.archlinux.org/title/Plymouth)
- [Change Default Shell](https://wiki.archlinux.org/title/Command-line_shell#Changing_your_default_shell)
```bash 
chsh -l
chsh -s /bin/fish
```

Configure login manager
- [ReGreet wiki](https://github.com/rharish101/ReGreet?tab=readme-ov-file#usage)

Configure Sway
- [Arch wiki](https://wiki.archlinux.org/title/Sway)
- [Sway wiki](https://github.com/swaywm/sway/wiki/Useful-add-ons-for-sway)
```bash 
# minimal setup is (but we clone dotiles from repo)
cp /etc/sway/config ~/.config/sway
helix ~/.config/sway/config
> change term to your terminal
```

Get dotfiles. This is a private repo, copy ssh keys from a usb stick first.
I currently just have a global ignore and force add files.
I don't like this way, but it is what I'm using at the moment
```bash 
cd ~
git init
git remote add <branch> <git repo>
git pull
```





----------

## TODO
- [General_recommendations](https://wiki.archlinux.org/title/General_recommendations)
- [Backup](https://wiki.archlinux.org/title/Snapper)

[Sway Inspiration](https://github.com/Madic-/Sway-DE)

### Alacritty
- [A good guide](https://clubmate.fi/alacritty)
- [find themes here](https://github.com/rajasegar/alacritty-themes)
