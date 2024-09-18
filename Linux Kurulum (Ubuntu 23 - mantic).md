# Linux Kurulum (Ubuntu 23 - mantic)

---

## Kurucu System İçin Bağımlılıklar :

```bash
sudo apt update
sudo apt install grub-efi-amd64 -y
sudo apt install debootstrap -y
sudo apt install arch-install-scripts -y
sudo apt install pacman-package-manager -y
sudo apt install xorriso squashfs-tools mtools grub-pc-bin grub-efi devscripts -y

sudo loadkeys trq.map
```

## Dizin İnşaası

```bash

sudo mkdir -p ./tmp/
sudo chown root:root ./tmp/

sudo mkdir -p ./tmp/debroot/
sudo mkdir -p ./tmp/iso/

sudo mkdir -p ./tmp/iso/live
sudo mkdir -p ./tmp/iso/boot
sudo mkdir -p ./tmp/iso/boot/grub

sudo chmod 777 ./tmp/debroot/

```

## Temel System Kurulumu : Debootstrap

```bash

sudo debootstrap mantic ./tmp/debroot/

sudo su

for dir in sys dev proc ; do mount --rbind /$dir ./tmp/debroot/$dir && mount --make-rslave ./tmp/debroot/$dir ; done

sudo chroot ./tmp/debroot/ apt update
sudo chroot ./tmp/debroot/ apt upgrade -y
sudo chroot ./tmp/debroot/ apt dist-upgrade -y

```

## Uygulama Repoları : sources.list

```bash
sudo nano ./tmp/debroot/etc/apt/sources.list
```

```read
deb http://archive.ubuntu.com/ubuntu/ mantic main restricted universe
deb http://security.ubuntu.com/ubuntu/ mantic-security main restricted universe
deb http://archive.ubuntu.com/ubuntu/ mantic-updates main restricted universe
```

```bash
sudo chroot ./tmp/debroot/ apt update
sudo chroot ./tmp/debroot/ apt upgrade -y
```

## Yerel Ağ Tanımlamaları : resolv.conf

```bash
sudo nano ./tmp/debroot/etc/resolv.conf
```

```read
nameserver 127.0.0.53
options edns0 trust-ad
search home
```

## Yerelleştirme (Türkçe, Türkiye)

```bash
sudo nano ./tmp/debroot/etc/locale.gen
```

```read
en_US.UTF-8 UTF-8
tr_TR.UTF-8 UTF-8
```

```bash
sudo nano ./tmp/debroot/etc/locale.conf
```

```read
LANG=tr_TR.UTF-8
```

```bash
sudo chroot ./tmp/debroot/ apt install locales -y
sudo chroot ./tmp/debroot/ dpkg-reconfigure tzdata
sudo chroot ./tmp/debroot/ dpkg-reconfigure locales
sudo chroot ./tmp/debroot/ locale-gen
```

```bash
sudo nano /mnt/etc/vconsole.conf
```

```read
KEYMAP=trq
```

## Bellek Yapılanması

```bash
sudo nano ./tmp/debroot/etc/fstab
```

```read
overlay / overlay rw 0 0
tmpfs /tmp tmpfs rw,nosuid,nodev,inode64 0 0
```

## Kernel Çekirdeği

```bash
sudo chroot ./tmp/debroot/ apt update
sudo chroot ./tmp/debroot/ apt install linux-image-generic -y

## Diğer library ler

sudo chroot ./tmp/debroot/ apt install sudo dhcpcd5 -y

sudo chroot ./tmp/debroot/ apt install linux-image-amd64 -y
sudo chroot ./tmp/debroot/ apt install linux-headers-amd64 -y
sudo chroot ./tmp/debroot/ apt install console-setup ntp -y


sudo apt install bookworm-backports -y

#sudo chroot ./tmp/debroot/ apt install grub-efi-amd64 -y
sudo chroot ./tmp/debroot/ apt install btrfs-progs -y
sudo chroot ./tmp/debroot/ apt install exfatprogs -y
sudo chroot ./tmp/debroot/ apt install exfat-utils -y
sudo chroot ./tmp/debroot/ apt install exfat-fuse -y



# sudo chroot ./tmp/debroot/ apt install zfsutils-linux -y
# sudo chroot ./tmp/debroot/ apt install zfs-initramfs -y
# sudo chroot ./tmp/debroot/ apt install zfs-dkms -y

sudo chroot ./tmp/debroot/ apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget -y

sudo chroot ./tmp/debroot/ apt install git dosfstools amd64-microcode nano -y


sudo chroot ./tmp/debroot/ apt install dosfstools amd64-microcode network-manager git cryptsetup sudo -y


sudo chroot ./tmp/debroot/ apt install tzdata curl -y

#sudo chroot ./tmp/debroot/ apt install sensors-applet -y

#sudo chroot ./tmp/debroot/ apt install psensor fancontrol -y
sudo chroot ./tmp/debroot/ apt install tlp ubuntu-restricted-extras -y

sudo chroot ./tmp/debroot/ apt install live-boot -y

sudo chroot ./tmp/debroot/ apt install live-boot-initramfs-tools -y

sudo chroot ./tmp/debroot/ apt install extlinux -y

sudo chroot ./tmp/debroot/ apt --fix-missing update
sudo chroot ./tmp/debroot/ apt --fix-broken install -y
sudo chroot ./tmp/debroot/ apt autoremove
sudo chroot ./tmp/debroot/ apt clean
sudo chroot ./tmp/debroot/ update-initramfs -u

```

## Kullanıcılar

```bash
sudo chroot ./tmp/debroot/ useradd -mG sudo admin
sudo chroot ./tmp/debroot/ useradd user

sudo chroot ./tmp/debroot/ passwd

sudo chroot ./tmp/debroot/ passwd admin

####

sudo nano ./tmp/debroot/etc/sudoers
```

```read

admin ALL=(ALL:ALL) NOPASSWD: ALL

user ALL=(ALL)ALL, !SHELLS, !SU

```

```bash
sudo mkdir -p ./tmp/debroot/home/admin/

sudo mkdir -p ./tmp/debroot/home/admin/Templates
sudo mkdir -p ./tmp/debroot/home/admin/Public
sudo mkdir -p ./tmp/debroot/home/admin/Videos
sudo mkdir -p ./tmp/debroot/home/admin/Music
sudo mkdir -p ./tmp/debroot/home/admin/Pictures
sudo mkdir -p ./tmp/debroot/home/admin/Documents
sudo mkdir -p ./tmp/debroot/home/admin/Desktop
sudo mkdir -p ./tmp/debroot/home/admin/Downloads

sudo mkdir -p ./tmp/debroot/home/admin/.config
sudo mkdir -p ./tmp/debroot/home/admin/.icons
sudo mkdir -p ./tmp/debroot/home/admin/.local

sudo mkdir -p ./tmp/debroot/home/admin/.config/bspwm
sudo mkdir -p ./tmp/debroot/home/admin/.config/sxhkd

sudo chmod 777 -R ./tmp/debroot/home/admin/

sudo mkdir -p ./tmp/debroot/0/
sudo chmod 777 -R ./tmp/debroot/0/

git clone https://github.com/Mustafaozver/erp.git ./tmp/debroot/0/


```

## Desktop Environment

### KDE Plasma

```bash
sudo chroot ./tmp/debroot/ apt install kde-full -y
sudo chroot ./tmp/debroot/ dpkg-reconfigure sddm
```

### Minimal Flying Desktop

```bash
sudo chroot ./tmp/debroot/ apt install lightdm -y
sudo chroot ./tmp/debroot/ apt install openbox -y

sudo chroot ./tmp/debroot/ dpkg-reconfigure lightdm

```

### Minimal Tiling Desktop

```bash
sudo chroot ./tmp/debroot/ apt install lightdm -y
sudo chroot ./tmp/debroot/ apt install bspwm sxhkd -y

sudo chroot ./tmp/debroot/ dpkg-reconfigure lightdm
```

### Kiosk

```bash
sudo chroot ./tmp/debroot/ apt install lightdm -y
sudo chroot ./tmp/debroot/ apt install xorg -y
sudo chroot ./tmp/debroot/ apt install xinit -y
sudo chroot ./tmp/debroot/ apt install slick-greeter -y

sudo chroot ./tmp/debroot/ apt install bspwm sxhkd -y
```

### Ayarlar :

#### SDDM Auto Login

```bash
sudo mkdir -p ./tmp/debroot/etc/sddm.conf.d/
sudo nano ./tmp/debroot/etc/sddm.conf.d/autologin.conf
```

```read
[Autologin]
User=admin
Session=plasma

[Theme]  
Current=maya  
CursorTheme=Esinti_Dark_Neon  
Font=Noto Sans,10,-1,0,50,0,0,0,0,0  
  
[Users]  
MaximumUid=60000  
MinimumUid=1000

[General]
Numlock=on

```

#### LightDM Auto Login

```bash
sudo nano ./tmp/debroot/etc/lightdm/lightdm.conf
```

```read
[SeatDefaults]
autologin-user=admin
autologin-user-timeout=0
user-session=bspwm
```

##### Kiosk Modu

```bash
sudo nano ./tmp/debroot/usr/share/xsessions/kiosk.desktop
```

```read
[Desktop Entry]
Name=Kiosk
Exec=konsole
TryExec=konsole
Type=Application
```

```bash
sudo nano ./tmp/debroot/home/admin/.config/bspwm/bspwmrc
```

```bash
bspc monitor -d I

bspc config border_width            0
bspc config window_gap              0

bspc config borderless_monocle      true
bspc config gapless_monocle         true

konsole
```

```bash
sudo nano ./tmp/debroot/home/admin/.config/sxhkd/sxhkdrc
```

```read
ctrl + alt + t
    konsole
```

## Gerekli Uygulamalar

```bash
sudo chroot ./tmp/debroot/ apt install nano -y

sudo chroot ./tmp/debroot/ apt install chromium-browser -y

sudo chroot ./tmp/debroot/ apt update
sudo chroot ./tmp/debroot/ apt --fix-missing update
sudo chroot ./tmp/debroot/ apt --fix-broken install
sudo chroot ./tmp/debroot/ apt autoremove
sudo chroot ./tmp/debroot/ apt clean

#sudo chroot ./tmp/debroot/ apt install thunderbird -y
sudo chroot ./tmp/debroot/ apt install timeshift -y


sudo chroot ./tmp/debroot/ apt install nodejs npm -y
sudo chroot ./tmp/debroot/ apt install sqlite3 -y

sudo chroot ./tmp/debroot/ apt install grub-efi-amd64 -y
sudo chroot ./tmp/debroot/ apt install debootstrap -y
sudo chroot ./tmp/debroot/ apt install arch-install-scripts -y
sudo chroot ./tmp/debroot/ apt install pacman-package-manager -y

sudo chroot ./tmp/debroot/ loadkeys trq.map

sudo mkdir -p ./tmp/debroot/__tmp

wget -O ./tmp/debroot/__tmp/google-chrome-stable.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

wget -O bitwarden.deb https://vault.bitwarden.com/download/?app=desktop&platform=linux&variant=deb


sudo chroot ./tmp/debroot/ dpkg -i /__tmp/*

sudo rm -rf ./tmp/debroot/__tmp/*

## # # # # #

sudo mkdir -p /mnt/0/grub-btrfs/
git clone https://github.com/Antynea/grub-btrfs.git /mnt/0/

cd grub-btrfs
sudo make install

## # # # # #

```

## Temizlik

```bash
sudo chroot ./tmp/debroot/ apt update
##sudo chroot ./tmp/debroot/ apt upgrade -y
sudo chroot ./tmp/debroot/ apt --fix-missing update
sudo chroot ./tmp/debroot/ apt --fix-broken install -y
sudo chroot ./tmp/debroot/ apt autoremove
sudo chroot ./tmp/debroot/ apt clean

sudo chroot ./tmp/debroot/ update-initramfs -u
sudo chroot ./tmp/debroot/ update-grub

sudo umount -lf -R ./tmp/debroot/* 2>/dev/null

sudo chroot ./tmp/debroot/ apt autoremove
sudo rm -f ./tmp/debroot/root/.bash_history
sudo rm -rf ./tmp/debroot/var/lib/apt/lists/*
find ./tmp/debroot/var/log/ -type f | xargs rm -f


sudo rm -f ./tmp/debroot/etc/apt/sources.list

## tam silme

sudo rm -rf ./tmp/*

```

## Grub Girdisi

```bash
sudo nano ./tmp/iso/boot/grub/grub.cfg
```

```read

insmod all_video

menuentry "Start" --class debian {
 linux /live/vmlinuz boot=live live-config live-media-path=/live quiet splash --
 initrd /live/initrd.img
}

menuentry "UEFI Firmware Settings" {  
 fwsetup  
}

set timeout=60

```

## İmaj Al : Squashfs

```bash
sudo rm ./tmp/iso/live/filesystem.squashfs

sudo mksquashfs ./tmp/debroot/ ./tmp/iso/live/filesystem.squashfs -comp gzip -wildcards

```

## İmaj Al : ISO

```bash

sudo rm ./tmp/ISO.iso

sudo cp -pf ./tmp/debroot/boot/initrd.img* ./tmp/iso/live
sudo cp -pf ./tmp/debroot/boot/vmlinuz* ./tmp/iso/live

sudo grub-mkrescue ./tmp/iso/ -o ./tmp/ISO.iso

```

## EKLER :

```bash
sudo unsquashfs -f -d ./tmp/debroot/ ./tmp/iso/live/filesystem.squashfs

```

---

## Etiketler :

#linux #sudo #debootstrap #terminal #format #mount #uefi #mkdir #chown #chroot #chmod #os #disk #memory #dependency #grub #config #boot #kernel #grub #kde #plasma #squashfs #image #iso #usb

## Bağlantılar :

[[CLİNUX/Linux Kurulumu|Linux Kurulumu]]

[Debian 11.4 ZFS Bootstrap | Debian ZFS Command Line Installation - YouTube](https://www.youtube.com/watch?v=7F7Ch-ZkiQU)
