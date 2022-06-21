+++
title = "Github Pages üzerine Hugo Kurulumu"
author = ["Taha"]
description = "Github Pages üzerine Hugo Kurulumu"
date = 2022-06-21
tags = ["hugo", "github-pages"]
draft = false
+++

# Github Pages üzerine Hugo Kurulumu
Herkese selam. Bugün buradaki ilk postum ile karşınızdayım. Bu postta sizlerle birlikte github'un sağlamış olduğu github-pages üzerine hugo nasıl kurulur onu anlatacağım.
Bildiğiniz üzere kişisel blog açmak, domain almak gibi işlemler biraz ücret istiyor. Benim gibi öğrenci olanlar veya herhangi bir para geliri olmayanlar ise genelde github'un sağladığı ayrıcalıklardan faydalanabiliyor. Tabii github bu ayrıcalığı sağlıyor ama sadece sağlıyor. Hazır kurulu bir arayüz sağlamıyor. 
Nedir bu ayrıcalık derseniz; Github, kullanıcılarının kişisel bir domain kullanmasına imkan sağlıyor. Biraz düzenleme ile bu domain üstünde daha düzgün bir site oluşturabilirsiniz. Neyse lafı uzatmadan kuruluma geçelim.

<br>

# 1) Repo kurulumu
- Öncelikle github hesabımızda bu ayrıcalıktan faydalanmak için bir repo açmamız gerekiyor. Bunun için [Github](https://github.com)'u açıyoruz, giriş yapıyoruz ve soldaki menüden Yeni seçeneğini seçiyoruz.

- Ardından `kullanıcıadı.github.io` şeklinde yeni bir repo oluşturuyoruz. Dilerseniz açıklama kısmına `https://kullanıcıadı.github.io` linkini ekleyebilirsiniz

<br><br>

# 2) Hugo kurulumu
- Eğer bir linux makine kullanıyorsanız çoğu dağıtım repolarında hugo mevcut. 
  - Arch : `sudo pacman -S hugo`
  - Debian / Ubuntu : `sudo apt install hugo`
  - Fedora : `sudo dnf install hugo`
  - Gentoo : `emerge --ask www-apps/hugo`
  
<br><br>
  
# 3) Git kurulumu
- Eğer bir linux makine kullanıyorsanız çoğu dağıtım repolarında hugo mevcut. 
 - Arch : `sudo pacman -S git`
 - Debian / Ubuntu : `sudo apt install git`
 - Fedora : `sudo dnf install git`
 - Gentoo : `emerge --ask dev-vcs/git`

<br><br>
  
# 4) Hugo'yu repo için kurmak
- Gereken araçları halihazırda sağladığımıza göre gerekli dosyaları oluşturmaya başlayalım. Öncelikle oluşturduğumuz repoyu bilgisayara çekmemiz gerekiyor.
```bash
$ cd ~/Documents
$ git clone https://github.com/kullanıcıadı/kullanıcıadı.github.io
 ```
 
 
- Sonrasında hugo'yu repo içine kuralım
```bash
$ hugo new site kullanıcıadı.github.io --force
$ cd kullanıcıadı.github.io
```

<br>

- Hugo'yu yapılandıralım. Bunun için `nano config.toml` yazıyoruz. Ve içindekileri temizleyerek alttaki satırları ekliyoruz.

```
baseURL = 'https://kullanıcıadı.github.io'
languageCode = 'en-us'
title = "Benim güzel blogum"
theme = "hugo-coder"

[author]
  name = "Benim adım"
  email = "deneme@ornek.com"

[markup.goldmark.renderer]
  unsafe = true

[params]
  author = "Benim adım"
  info = "Ben bir geliştiriciyim"
  description = "Burası benim mekanım"
  keywords = "blog"
  colorScheme = "dark"

[[params.social]]
  name = "Github"
  icon = "fa fa-github fa-2x"
  weight = 1
  url = "https://github.com/kullanıcıadı"
[[params.social]]
  name = "RSS"
  icon = "fa fa-2x fa-rss"
  weight = 6
  url = "index.xml"
  rel = "alternate"
  type = "application/rss+xml"

[[menu.main]]
  name = "Blog"
  weight = 1
  url = "posts/"
```

<br> 

- Dosyayı kaydedip çıktıktan sonra ise [hugo-coder](https://github.com/luizdepra/hugo-coder) adlı temayı kuruyoruz. Dilerseniz kendinize başka bir hugo teması seçebilirsiniz.
```bash
$ git submodule add https://github.com/luizdepra/hugo-coder.git themes/hugo-coder
```

<br>

- Bundan sonra ise her gönderi attığımızda siteyi yenileyecek olan o kodu oluşturuyoruz.
```bash
$ mkdir .github/workflows -p
$ nano .github/workflows/gh-pages.yml          #içine alttakileri yazın

name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

<br>

- Kaydedip çıktıktan sonra ise bu yaptığımız bütün değişiklikleri kendi repomuza aktaralım (bunun için öncelikle github'tan access token oluşturmanız gerekiyor. [Buradan](https://github.com/settings/tokens) yeni bir token oluşturun ve oluşturduktan sonra ghp ile başlayan tokeni bir yerde saklayın.
```bash
$ hugo                                   # Son durumu kaydetmek için önemlidir
$ git add .
$ git commit -m "HugoSetup"
$ git push -u origin main -f             ## Sizden kullanıcı adı ve şifre isteyecek. Kullanıcı adınızı yazın ve 
                                         #  şifre kısmına ghp ile başlayan tokeni girin. Dosyalar karşıya yüklenecektir.
```

<br>

- Dosyalar karşıya yüklendikten sonra 1-2 dakika bekleyin ve ardından şu ayarı uygulayın.

![image](https://user-images.githubusercontent.com/62564400/174791940-e8b7dabc-a047-4bd9-84b5-8e7234a8d672.png)

<br><br>

## 5) Artık küçük çaplı websitemiz hazır. Eğer bir domain adresine sahipseniz bunu uygulayabilirsiniz. Sitenizde yeni bir post oluşturmak için şurada yeni dosya oluşturabilirsiniz.
`https://github.com/kullanıcıadı/kullanıcıadı.github.io/content/posts/benimyenipostum.md`
