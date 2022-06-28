+++
title = "Fedora'yı Bootstrap Olarak Kurmak"
author = ["Taha"]
description = "Fedora'yı Bootstrap Olarak Kurmak"
date = 2022-06-26
tags = ["fedora", "bootstrap", "minimal", "fedora-minimal"]
draft = false
+++

# Fedora'yı Bootstrap Olarak Kurmak
Herkese selam. Bugün makalemizde Fedora'yı nasıl bootstrap olarak kuracağımızı anlatacağım. Bildiğiniz üzere bazı linux dağıtımları varsayılan hali ile epey paket barındırıyor. Bu paketlerin bir çoğunu kullanmayan var. Haliyle dağıtımı kurduktan sonra paketleri elle silmek epey yorucu olabiliyor. "ArchLinux, ilk kurulumda istediğimiz paketleri kurmamıza imkan sağlıyor ne güzel" dediğinizi duyar gibiyim. Aynen bu makalemizde ArchLinux kurar gibi Fedora kuracağız. Ama birtakım malzemelere ihtiyacımız var. Eğer linux sistemleri hakkında çok bilginiz yoksa bunu denememenizi şiddetle tavsiye ederim. Aksi durumda sistemi kullanılamaz hale getirebilirsiniz.
- 1) Fedora ISO dosyası (Xfce olanı [şuradaki](https://spins.fedoraproject.org) adresten indirebilirsiniz. Hangi ortam olduğunun çok bir önemi yok. Sadece kurulum esnasında kolaylık sağlaması bizim için daha iyidir. İsterseniz başka bir linux distrosunun ISO dosyasını kullanabilirsiniz)
- 2) Bir adet USB Bellek

<br>

# 1) Gerekli Ortamın Hazırlanması
Öncelikle herhangi bir program yardımı ile iso dosyasını flaş belleğe yazdıralım. Eğer UEFI sistem kullanıyorsanız flaş bellekteki sistemin GPT olmasına dikkat edelim.

<br>

# 2) Bilgisayarı USB Bellekten Başlatmak
Eğer buraya kadar sorunsuz geldiyseniz bilgisayarı USB bellek üzerinden başlatın. Ve masaüstüne gelin. 
- İlk adımda internet bağlantısını sağlayalım. Ardından disk düzenini kendimize göre ayarlayalım. (Gparted aracılığıyla)
  - Benim tavsiyem şöyle bir disk yapılandırması olacak. Eğer sıfırdan windows ile kurulum yapacaksanız (256GB Disk için)
    - 256MB EFI - 16MB MSR - 1GB Recovery - 100GB Windows - 140GB Linux (Btrfs)
  - Eğer sadece linux olacaksa (128GB Disk için)
    - 256MB EFI - 125GB Linux Root (Btrfs)
  - (256GB Disk için)
    - 256MB EFI - 240GB Linux Root (Btrfs)

<br>

# 3) Temel Repoların ve GPG Keylerin İçeri Alınması
Gerekli tüm her şeyi sağladıysanız öncelikle `dnf` kurmak ile başlayalım. Eğer Fedora Xfce isosunu açtıysanız zaten kurmanıza gerek yoktur. Eğer ArchLinux veya Debian tabanlı bir distro açtıysanız şu komutları uygulayın.
- ArchLinux : `sudo pacman -S dnf arch-install-scripts`
- Debian : `sudo apt install dnf arch-install-scripts`
- Ubuntu (öncelikle universe reposunu aktifleştirmeniz gerekiyor) : `sudo apt install dnf arch-install-scripts`

<br>

# 4) Dnf'yi kurduktan sonra temel Fedora repolarını kuralım. ("$ ve > işaretleri shell temsil eder. Siz yazmayın!!!")
```bash
~$ for a in fedora-cisco-openh264.repo fedora-modular.repo fedora-updates-modular.repo fedora-updates-testing-modular.repo fedora-updates-testing.repo fedora-updates.repo fedora.repo; do
>    wget https://raw.githubusercontent.com/discover02/mycstrepo/master/fedora/$a
> done
~$
~$ sudo mkdir -p /etc/yum.repos.d
~$ sudo mv *.repo /etc/yum.repos.d
```

<br>

# 5) GPG anahtarlarını içeri alalım.
```bash
~$ wget https://download-ib01.fedoraproject.org/pub/fedora/linux/releases/36/Everything/x86_64/os/Packages/f/fedora-gpg-keys-36-1.noarch.rpm
~$ rpm2cpio fedora-gpg-keys-36-1.noarch.rpm | cpio -idmv
~$ sudo mv etc/pki/rpm-gpg /etc/pki/
~$ sudo dnf --releasever=36 install fedora-gpg-keys
```

<br>

# 6) Dnf artık kullanıma hazır. Öncelikle bölümlerimizi bağlayalım. Root bölümünü BTRFS olarak biçimlendirdiğinizi varsayarsak
```bash
~$ sudo mount /dev/diskadıvenumarası /mnt
~$ cd /mnt
~$ sudo btrfs su cr @                           ## Root bölümü
~$ sudo btrfs su cr @home                       ## Home bölümü
~$ sudo btrfs su cr @snapshots                  ## Anlık görüntülerin kaydedileceği bölüm
                                               # (yakında snapper hakkında bir kılavuz hazırlayacağım.)
~$ cd
~$ sudo umount /mnt
```

<br>

# 7) Bölümleri oluşturduktan sonra artık root bölümünü bağlayalım. Ve temel sistemi kuralım.
- Eğer Legacy BIOS kullanıyorsanız EFI geçen komutları yazmanıza gerek yoktur.
```bash
~$ sudo mount -o subvol=@ /dev/diskadıvenumarası /mnt
~$ sudo mkdir -p /mnt/home 
~$ sudo mkdir -p /mnt/boot/efi && sudo mount /dev/efidiskvenumara /mnt/boot/efi
~$ sudo mount -o subvol=@home /dev/diskadıvenumarası /mnt/home
~$ sudo dnf --installroot=/mnt --releasever=36 --forcearch=x86_64 groupinstall "Core" "Minimal Install"
```

<br>

# 8) Sistemimizin fstab dosyasını çıkartıp kaydedelim.
```bash
~$ sudo genfstab -U /mnt >> /mnt/etc/fstab
```

# 9) Temel sistemi kurduktan sonra chroot ile sisteme girelim.
```bash
~$ cd /mnt
~$ sudo rm etc/resolv.conf
~$ sudo touch etc/resolv.conf
~$ for a in dev etc/resolv.conf proc sys sys/firmware/efi/efivars; do
>   sudo mount --bind /$a ./$a
> done
~$ sudo chroot . /bin/bash
```

<br>

# 10) Sistemimizde dil ve bölge ayarlarını yapalım. (ingilizce = en_US.UTF-8 | türkçe = tr_TR.UTF-8)
```bash
~# ln -sf /usr/share/zoneinfo/Europe/Istanbul /etc/localtime
~# echo -e 'LANG=en_US.UTF=8' | tee /etc/locale.conf
~# echo 'KEYMAP="tr"\nFONT="eurlatgr"' | tee /etc/vconsole.conf
~# dnf --releasever=36 install langpacks-{en,tr}\* glibc-all-langpacks
```

<br>

# 11) Sistemimizde hostname ayarlayalım.
```bash
~# echo "istediğinizad" | tee /etc/hostname
```

<br>

# 12) Sistemimizde yeni bir kullanıcı oluşturalım
```bash
~# useradd -mG wheel -c Takmaisim kullanıcıadı
~# passwd kullanıcıadı
~# passwd root
~# echo "kullanıcıadı ALL = (ALL) ALL" | tee /etc/sudoers.d/10-kullanıcıadı
```

<br>

# 13) Grub önyükleyicisini ayarlayalım. (Fedora, grub'u bizim yerimize ayarlayacaktır.)
```bash
~# dnf reinstall grub2-efi-* grub2-common
```

<br>

# 14) Artık istediğimiz masaüstü ortamını şu komutlar ile kurabiliriz.
```bash
~# dnf --releasever=36 install @base-x
```
- Gnome : `dnf --releasever=36 install gdm gnome-shell nautilus gnome-terminal gnome-system-monitor xdg-user-dirs-{gtk,gnome} fedora-workstation-backgrounds`
  - `sudo systemctl enable gdm`
    - `sudo systemctl set-default graphical.target`
    - Ekstra kurmak isteyebileceğiniz paketler : `firefox gnome-software file-roller gedit gnome-terminal-nautilus gvfs-mtp`

- KDE   : `dnf --releasever=36 install sddm plasma-desktop plasma-nm konsole kcm_colors kcm-fcitx young kosco kscreen spectacle ksysguard plasma-user-manager dolphin`
  - `sudo systemctl enable sddm`
    - `sudo systemctl set-default graphical.target`
    - Ekstra kurmak isteyebileceğiniz paketler : `firefox kate plasma-discover yakuake kcm_colors kcm_systemd okular gwenview ImageMagick sddm-kcm sddm-sddm themes-breeze kgamma colord-kde kdegraphics-thumbnailers kffmpegthumbnailer NetworkManager-config-connectivity-fedora kdeplasma-addons kinfocenter ksysguard kde-partitionmanager gvfs-mtp`

- Xfce  : `dnf --releasever=36 install network-manager-applet xfwm4 xfce4-power-manager xfce4-session xfce4-settings xfce4-whiskermenu-plugin xfdesktop lightdm-gtk\* lightdm xfce4-terminal`
  - `sudo systemctl enable lightdm`
    - `sudo systemctl set-default graphical.target`
    - Ekstra kurmak isteyebileceğiniz paketler : `firefox`

<br>

# 15) Reboot atarak işlemi sonlandırıyoruz.
Eğer bir eksik görürseniz pr atmaktan çekinmeyin. Dosyanın düzenlenebilir haline [buradan](https://github.com/discover02/discover02.github.io/blob/main/content/posts/FedoraBootstrap.md) ulaşabilirsiniz.
