+++
title = "Linux üzerinde Minecraft Server + GeyserMC Kurulumu"
author = ["Taha"]
description = "Linux üzerinde Minecraft Server + GeyserMC Kurulumu"
date = 2022-06-25
tags = ["minecraft", "minecraft-server", "crossplay", "bedrock", "java", "bedrock-java"]
draft = false
+++


# Linux Bilgisayar Üzerinde Crossplay Minecraft Server Kurulumu
Herkese selam. Bugün yazımızda herhangi bir linux kurulu bilgisayara Minecraft Server ve GeyserMC nasıl kurulur onu anlatacağım.

Bir süredir bu konu hakkında araştırma yapıyordum. Elimde bir bilgisayar ve bir telefon var. Kardeşim veya bir arkadaşım ile yakından olsun uzaktan olsun minecraft oynamanın yolunu ararken en sonunda bunu kendi bilgisayarımda çözmeyi başardım. Bugün bu makalemizde bunu nasıl yaptığımı adım adım anlatacağım. Diyebilirsiniz Aternos, Ploudos gibi ücretsiz alternatifler varken neden kendi bilgisayarında çalıştırmak istiyorsun diye. Evet bu tarz ücretsiz alternatifler var ama kısıtlı oluyor genelde. Sonuçta kimse hayrına server vermek istemez. Ek olarak bu sunucularda bakım veya kesinti olursa oyununuz yarım kalabilir. Neyse lafı uzatmadan konuya dalalım.

Not : "$ işaretleri shell'i temsilidir"

<br>

### 1) Öncelikle bilgisayarımıza Java'yı indiriyoruz. Ben en son sürüm Java üzerinde test ettim. Daha eski sürümlerde sorun çıkartacağını pek düşünmüyorum ama dilerseniz sizde şu komutlar ile son sürüm Java'yı bilgisayarınıza kurabilirsiniz.
- ArchLinux : ```sudo pacman -S jdk-openjdk```
- Debian/Ubuntu : ```sudo apt install openjdk-17-jre openjdk-17-jdk ```
- Fedora/CentOS : ```sudo yum install java-latest-openjdk```
- OpenSUSE : ```sudo zypper in java-17-openjdk```
- Gentoo : ```sudo emerge --ask dev-java/openjdk-bin && sudo emerge --ask --oneshot virtual/jdk virtual/jre```

<br>

### 2) Sonrasında ise gerekli dosyaları indirelim.
Bu dosyaları ~/Downloads altında Minecraft klasörü içine indirelim.
- PaperMC : https://papermc.io/downloads
- GeyserMC (Spigot) : https://ci.opencollab.dev//job/GeyserMC/job/Geyser/job/master/
- Floodgate (Spigot) : https://ci.opencollab.dev/job/GeyserMC/job/Floodgate/job/master/

<br>

### 3) Dosyaları indirdikten sonra gerekli kurulumu yapalım.
Bunun için terminali açalım ve sırasıyla şu komutları yazalım:
```bash
$ cd ~/Downloads/Minecraft
$ java -jar paper-*.jar
```

<br>

### 4) İşlem bittikten sonra EULA'yı kabul edelim.
Bunun için şu komutu yazıp `false` olan değeri `true` olarak değiştiriyoruz.
```bash
$ nano eula.txt
```

<br>

### 5) EULA'yı kabul ettikten sonra şu komut ile GeyserMC ve Floodgate pluginlerini kuralım.
```bash
$ mkdir plugins
$ mv Geyser-Spigot.jar floodgate-spigot.jar plugins/
```

<br>

### 6) Tek seferlik sunucumuzu çalıştıralım ve kapatalım.
Bunun için şu komutları yazalım:
```bash
$ java -Xmx1024M -Xms1024M -jar paper-*.jar nogui
[{ işlemin bitmesini bekleyin ve ardından }]
> stop
```

<br>

### 7) Yerel sunucumuzu yapılandıralım. Bunun için sırasıyla şu işlemleri yapabilirsiniz.
```bash
$ nano server.properties
-> difficulty=easy                   # Oyun zorluğunu seçebilirsiniz. (easy, medium, hard)
-> gamemode=survival                 # Oyun modunu seçebilirsiniz. (survival, spectator, creative)
-> level-name=ISTEDIGINIZISIM        # Sunucunuza isim verebilirsiniz.
-> max-players=NUMARA                # Sunucunuzun kaç kişilik olacağını seçebilirsiniz.
-> motd=ACIKLAMA                     # Sunucunuza açıklama girebilirsiniz.
-> server-port=25565                 # Sunucunuzun portu (Varsayılan kalması daha iyidir)
```

```bash
$ nano plugins/Geyser-Spigot/config.yml
bedrock:                             # Altındaki özelliklerden şunları ayarlayabilirsiniz
->   port: 19132                     # Bedrock ile bağlanacak kişilere özel port (Varsayılan kalması daha iyidir)
->   motd1: "MyWorld"                # Sunucu açıklaması
->   motd2: "MyWorldLocalServer"     # Sunucu açıklaması
->   server-name: "MyWorld"          # Sunucu ismi
```

<br>

### 8) Eğer sistemde varsa firewall ile bedrock için açılmış port'a izin verelim
Eğer ufw varsa
```bash
sudo ufw allow 19132:19132/tcp
sudo ufw allow 19132:19132/udp
```

Eğer firewalld varsa
```bash
$ bash

$ INTERFACES="$(firewall-cmd --get-active-zones | grep -v interfaces)"
$ for a in $INTERFACES; do
>   sudo firewall-cmd --zone=$a --permanent --add-port=19132/tcp
>   sudo firewall-cmd --zone=$a --permanent --add-port=19132/udp
>   sudo firewall-cmd --reload
>   sudo systemctl restart firewalld.service
> done
```

<br>


### 9) Bilgisayarımızı yeniden başlatalım ve sunucumuz hazır. İsterseniz şöyle bir alias kullanarak tek seferde başlatabilirsiniz. 
Bunun için kullandığınız shell yapılandırmasına şunu yazın
```bash
alias mcserver="pushd ~/Downloads/Minecraft && java -Xmx1024M -Xms1024M -jar paper-*.jar nogui && popd"
```
Veya bilgisayar açılışında otomatik açılmasını sağlayabilirsiniz
```bash
$ bash
$ tee <<-'EOF' ~/.config/autostart/mcserver.desktop 1>/dev/null
[Desktop Entry]
Exec=bash -c "pushd ~/Downloads/Minecraft && java -Xmx1024M -Xms1024M -jar paper-*.jar nogui && popd"
Name=Minecraft Server
StartupNotify=false
Terminal=false
Type=Application
EOF
```



