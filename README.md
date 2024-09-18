# Linux Kurulum (Ubuntu 23 - mantic)

---

## Kurucu System İçin Bağımlılıklar :

```bash
sudo apt update
sudo apt install grub-efi-amd64 -y
sudo apt install debootstrap -y
sudo apt install arch-install-scripts -y
sudo apt install pacman-package-manager -y

## ISO gereklilikleri
sudo apt install debootstrap xorriso squashfs-tools mtools grub-pc-bin grub-efi devscripts git nano -y
```

## Formatlama

476.94 GB Boyutundaki /dev/nvme0n1 Bölümü için :

|Bölüm|Türü|Tür Kodu|Yol|Boyut|İlk Sektör|Son Sektör|
|---|---|---|---|---|---|---|
|/dev/nvme0n1p1|UEFI|1|/boot/efi/|512 MB|Boş Bırak|+512M|
|/dev/nvme0n1p2|BOOT|20|/boot/|1 GB|Boş Bırak|+1G|
|/dev/nvme0n1p3|SWAP|19|-|16 GB (RAM Kadar)|Boş Bırak|+16G|
|/dev/nvme0n1p4|ROOT|23|/|100 GB|Boş Bırak|+100G|
|/dev/nvme0n1p5|HOME|42|/home|200 GB|Boş Bırak|+200G|
|/dev/nvme0n1p6|DEPO|20|-|159 GB|Boş Bırak|Boş Bırak|

``` bash

sudo fdisk /dev/nvme0n1

# g: GPT,  n: NEW t: TYPE
# t 1 : UEFI
# t 20 : BOOT, Linux FS
# t 19 : SWAP
# t 23 : ROOT
# t 42 : HOME

```

```bash
sudo mkswap /dev/nvme0n1p3
sudo swapon /dev/nvme0n1p3


sudo mkfs.vfat /dev/nvme0n1p1
sudo mkfs.ext4 /dev/nvme0n1p2
sudo mkfs.btrfs /dev/nvme0n1p4
sudo mkfs.ext4 /dev/nvme0n1p5
sudo mkfs.vfat /dev/nvme0n1p6

```

## Mount İşlemleri


```bash

sudo mount /dev/nvme0n1p4 /mnt
sudo btrfs su cr /mnt/@
#sudo btrfs su cr /mnt/@home
sudo umount /mnt


sudo mount -t btrfs -o noatime,compress=lzo,space_cache=v2,subvol=@ /dev/nvme0n1p4 /mnt



sudo chmod 777 /mnt



sudo mkdir -p /mnt/home
sudo mount /dev/nvme0n1p5 /mnt/home

sudo mkdir -p /mnt/boot
sudo mount /dev/nvme0n1p2 /mnt/boot

sudo mkdir -p /mnt/boot/efi
sudo mount /dev/nvme0n1p1 /mnt/boot/efi


```

## Temel System Kurulumu : Debootstrap


```bash

sudo debootstrap mantic /mnt

for dir in sys dev proc ; do mount --rbind /$dir /mnt/$dir && mount --make-rslave /mnt/$dir ; done

sudo chroot /mnt apt update
sudo chroot /mnt apt upgrade -y

```

## Uygulama Repoları : sources.list


```bash
sudo nano /mnt/etc/apt/sources.list
```

```kl
deb http://archive.ubuntu.com/ubuntu/ mantic main restricted universe
deb http://security.ubuntu.com/ubuntu/ mantic-security main restricted universe
deb http://archive.ubuntu.com/ubuntu/ mantic-updates main restricted universe
```

## Yerel Ağ Tanımlamaları : resolv.conf

```bash
sudo nano /mnt/etc/resolv.conf
```

```kl
nameserver 127.0.0.53
options edns0 trust-ad
search home
```

## Yerelleştirme (Türkçe, Türkiye)

```bash
sudo rm /mnt/etc/localtime
sudo ln -s /mnt/usr/share/zoneinfo/Europe/Istanbul /mnt/etc/localtime


sudo nano /mnt/etc/locale.gen
```

```kl
en_US.UTF-8 UTF-8
tr_TR.UTF-8 UTF-8
```

```bash
sudo nano /mnt/etc/locale.conf
```

```kl
LANG=tr_TR.UTF-8
```

```bash
sudo chroot /mnt apt install locales -y
sudo chroot /mnt dpkg-reconfigure tzdata
sudo chroot /mnt dpkg-reconfigure locales
sudo chroot /mnt locale-gen
```

## Kernel Çekirdeği

[XanMod Kernel](https://xanmod.org/)
,


```bash
sudo chroot /mnt apt update
sudo chroot /mnt apt install linux-image-generic -y


sudo chroot /mnt dpkg -i /linux-image-zenmod.deb

sudo chroot /mnt apt install sudo dhcpcd5 -y

sudo chroot /mnt apt install grub-efi-amd64 -y
sudo chroot /mnt apt install btrfs-progs -y
sudo chroot /mnt apt install exfatprogs -y
sudo chroot /mnt apt install exfat-utils -y

# sudo chroot /mnt apt install zfsutils-linux -y
# sudo chroot /mnt apt install zfs-initramfs -y
# sudo chroot /mnt apt install zfs-dkms -y

sudo chroot /mnt apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget -y
sudo chroot /mnt apt install git dosfstools amd64-microcode nano -y
sudo chroot /mnt apt install dosfstools amd64-microcode network-manager git cryptsetup sudo -y
sudo chroot /mnt apt install lsb-release ca-certificates apt-transport-https software-properties-common -y
sudo chroot /mnt apt install cups printer-driver-all system-config-printer simple-scan xsane -y
sudo chroot /mnt apt install tzdata curl ca-certificates openssh-server curl -y
sudo chroot /mnt apt install sensors-applet -y
sudo chroot /mnt apt install psensor fancontrol -y
sudo chroot /mnt apt install tlp ubuntu-restricted-extras -y
sudo chroot /mnt apt install ufw -y



sudo chroot /mnt ubuntu-drivers autoinstall

sudo chroot /mnt apt --fix-missing update
sudo chroot /mnt apt --fix-broken install -y
sudo chroot /mnt apt autoremove
sudo chroot /mnt apt clean
sudo chroot /mnt update-initramfs -u


```

## Diskleri Sisteme Tanıtılması


```bash
sudo genfstab -U /mnt >> /mnt/etc/fstab

sudo nano /mnt/etc/fstab

```

Eklenecek :

```kl
tmpfs /tmp tmpfs rw,nosuid,nodev,inode64 0 0
```

## Bootloader : GRUB


```bash
sudo chroot /mnt apt install grub-efi-amd64 -y
sudo chroot /mnt update-initramfs -u
sudo chroot /mnt update-grub
sudo grub-install /dev/nvme0n1
sudo chroot /mnt grub-install
```

## Kullanıcılar


```bash

sudo chroot /mnt useradd -mG sudo mustafa

sudo chroot /mnt passwd

sudo chroot /mnt passwd mustafa


sudo nano /etc/sudoers
```

```kl
mustafa ALL=(ALL:ALL) NOPASSWD: ALL
```

```bash
sudo mkdir -p /mnt/home/mustafa/Templates
sudo mkdir -p /mnt/home/mustafa/Public
sudo mkdir -p /mnt/home/mustafa/Videos
sudo mkdir -p /mnt/home/mustafa/Music
sudo mkdir -p /mnt/home/mustafa/Pictures
sudo mkdir -p /mnt/home/mustafa/Documents
sudo mkdir -p /mnt/home/mustafa/Desktop
sudo mkdir -p /mnt/home/mustafa/Downloads
```

## Desktop Environment : KDE Plasma

```bash
sudo chroot /mnt apt install kde-full -y
sudo chroot /mnt apt install plasma-workspace-wayland -y
sudo chroot /mnt apt install kubuntu-desktop -y
sudo chroot /mnt apt install plasma-desktop -y

sudo chroot /mnt apt install qt5-style-kvantum qt5-style-kvantum-themes -y

sudo chroot /mnt apt install kwin-bismuth -y
sudo chroot /mnt apt install ultracopier -y

sudo chroot /mnt dpkg-reconfigure sddm


sudo chroot /mnt apt install samba samba-client samba-common -y

```

## Gerekli Uygulamalar

```bash
sudo chroot /mnt apt install synaptic -y
sudo chroot /mnt apt install flatpak -y
sudo chroot /mnt apt install nala -y
sudo chroot /mnt apt install libreoffice -y
sudo chroot /mnt apt install wget gpd nano -y

sudo chroot /mnt apt install htop -y
sudo chroot /mnt apt install git -y
sudo chroot /mnt apt install wine -y
sudo chroot /mnt apt install wine32 -y
sudo chroot /mnt apt install libwine -y
sudo chroot /mnt apt install fonts-wine -y
sudo chroot /mnt apt install neofetch -y
sudo chroot /mnt apt install zsh -y
sudo chroot /mnt apt install guake -y

sudo chroot /mnt apt install veracrypt -y
sudo chroot /mnt apt install thunderbird -y
sudo chroot /mnt apt install libreoffice -y
sudo chroot /mnt apt install distrobox -y
sudo chroot /mnt apt install podman -y

sudo chroot /mnt nala install firefox
sudo chroot /mnt apt install chromium-browser -y
sudo chroot /mnt apt install opera-stable -y

sudo chroot /mnt apt update
sudo chroot /mnt apt --fix-missing update
sudo chroot /mnt apt --fix-broken install
sudo chroot /mnt apt autoremove
sudo chroot /mnt apt clean




sudo chroot /mnt apt install curl apt-transport-https -y
sudo chroot /mnt apt install thunderbird -y
sudo chroot /mnt apt install timeshift -y
sudo chroot /mnt apt install syncthing -y
sudo chroot /mnt apt install openssh-server openssh-client -y
sudo chroot /mnt apt install stacer -y
sudo chroot /mnt apt install flatpak -y


sudo chroot /mnt apt install nodejs npm -y
sudo chroot /mnt apt install postgresql postgresql-contrib -y
sudo chroot /mnt apt install sqlite3 libsqlite3-dev -y

sudo chroot /mnt apt install redis -y

sudo chroot /mnt apt install rabbitmq-server -y
sudo chroot /mnt apt install elasticsearch -y






sudo mkdir -p /mnt/__tmp

wget -O /mnt/__tmp/google-chrome-stable.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

wget -O bitwarden.deb https://vault.bitwarden.com/download/?app=desktop&platform=linux&variant=deb


sudo chroot /mnt dpkg -i /__tmp/*

```

## Temizlik

```bash

sudo chroot /mnt update-initramfs -u

sudo chroot /mnt systemctl enable dhcpcd

sudo chroot /mnt update-grub

sudo umount -lf -R /mnt/* 2>/dev/null

sudo chroot /mnt apt autoremove
sudo rm -f /mnt/root/.bash_history
sudo rm -rf /mnt/var/lib/apt/lists/*
find /mnt/var/log/ -type f | xargs rm -f

sudo swapoff --all


```

## Açılışta

```bash

sudo add-apt-repository ppa:alessandro-strada/ppa
sudo apt-get install google-drive-ocamlfuse
mkdir ~/migoogledrive
google-drive-ocamlfuse ~/migoogledrive



curl -s https://syncthing.net/release-key.txt | sudo apt-key add -

echo "deb https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list

sudo apt install syncthing -y

sudo apt install wireshark -y




sudo snap install obsidian
sudo snap install muezzin
sudo snap install firefox
sudo snap install bitwarden
sudo snap install sqlitebrowser

sudo systemctl enable syncthing@username.service
sudo systemctl enable ssh

git config --global user.name "Mustafaozver"
git config --global user.email "mustafa.ozver@hotmail.com"
git config --global credential.credentialStore plaintext





sudo add-apt-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) universe"
sudo apt update

sudo apt install fonts-firacode -y



sudo reboot
```

---

# EKLER :

.

```bash

sudo mount -t btrfs -o noatime,compress=lzo,space_cache=v2,subvol=@ /dev/nvme0n1p4 /mnt
sudo mount /dev/nvme0n1p5 /mnt/home
sudo mount /dev/nvme0n1p2 /mnt/boot
sudo mount /dev/nvme0n1p1 /mnt/boot/efi



```
