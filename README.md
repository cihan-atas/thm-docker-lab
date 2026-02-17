# 🐳 **DOCKER PRIVILEGE ESCALATION LAB**
## TryHackMe Room - Docker Grubu Yetki Yükseltme

---

## 📋 **Lab Bilgileri**

| Özellik | Detay |
|---------|-------|
| **Zorluk** | Orta |
| **Süre** | 45-60 dakika |
| **Konular** | LFI, Docker güvenliği, Privilege Escalation, Container Escape |
| **Flag'ler** | `user1.txt` (host - webuser), `user2.txt` (container), `root.txt` (host - root) |
| **Ortam** | Ubuntu 20.04, Docker yüklü |

---

## 🎯 **Hedefler**

1. **Initial Access** - LFI zafiyetinden sisteme gir
2. **Flag 1** - Host'ta `user1.txt` al
3. **Keşif** - Docker grubundaki kullanıcıyı bul
4. **Yatay Geçiş** - `docker_user` hesabına geç
5. **Container'a gir** - Docker ile container başlat
6. **Flag 2** - Container'da `user2.txt` al
7. **Root Ol** - Host'ta yetki yükselt
8. **Flag 3** - Host'ta `root.txt` al

---

# 📖 **LAB SENARYOSU**

## 🔰 **AŞAMA 1: Initial Access (LFI ile)**

Sana bir web uygulaması veriliyor. Bu uygulamada **LFI (Local File Inclusion)** zafiyeti var.

**Web uygulaması:**
```
http://10.10.10.100/index.php?page=welcome.txt
```

`page` parametresi dosya okumak için kullanılıyor ama **hiçbir güvenlik kontrolü yok**!

### **Ne yapacaksın?**
1. LFI zafiyetini kullanarak `/etc/passwd` dosyasını oku
2. Sistemdeki kullanıcıları keşfet
3. Bir kullanıcının **SSH private key'ini** bul
4. SSH ile sisteme bağlan

### **Bulacağın kullanıcılar:**
- `webuser` (normal kullanıcı)
- `docker_user` (docker grubunda özel kullanıcı)

---

## 🏁 **AŞAMA 2: Flag 1 - user1.txt**

Sisteme `webuser` olarak giriş yaptıktan sonra, home dizininde **ilk flag** seni bekliyor.

**Konum:** `/home/webuser/user1.txt`

Bu flag'i alarak ilk aşamayı tamamlıyorsun.

---

## 🔍 **AŞAMA 3: Keşif ve Yatay Geçiş**

Artık sistemde `webuser` olarak gezinmeye başlıyorsun. Yapman gerekenler:

1. **Grupları kontrol et** - Hangi kullanıcılar özel yetkilere sahip?
2. **Docker yüklü mü?** - `which docker`
3. **Docker grubunda kimler var?** - `cat /etc/group | grep docker`

Burada `docker_user` adlı bir kullanıcının **docker grubunda** olduğunu keşfedeceksin. Ama sen `webuser` olarak docker'ı çalıştıramazsın.

### **docker_user'a nasıl geçeceksin?**
- `docker_user`'ın home dizininde SSH key var mı?
- LFI zafiyetini kullanarak `docker_user`'ın SSH private key'ini oku
- SSH ile `docker_user` olarak bağlan

Artık **docker grubunda bir kullanıcısın!** 🎯

---

## 📦 **AŞAMA 4: Container'a Gir ve Flag 2'yi Al**

Docker yetkilerin var. Şimdi:

1. **Docker imajlarını listele** - `docker images`
2. **Bir container başlat** - `docker run -d --name lab ubuntu:lab sleep 3600`
3. **Container'a gir** - `docker exec -it lab /bin/bash`

Container içinde **root yetkisine** sahipsin! Container'ın root dizininde **ikinci flag** seni bekliyor.

**Konum:** `/root/user2.txt`

Bu flag'i alarak container'a başarıyla girdiğini kanıtlıyorsun.

---

## 🏃 **AŞAMA 5: Host'a Kaçış ve Flag 3**

Container içinde root'sun ama asıl hedef **host makinede root olmak**.

### **Ne yapacaksın?**
1. **Host diskini container'a mount et**
   ```bash
   mkdir /mnt/host
   mount /dev/sda1 /mnt/host
   ```

2. **chroot ile host'a geç**
   ```bash
   chroot /mnt/host /bin/bash
   ```

3. **Host'ta root'sun!** Root dizininde **üçüncü flag** seni bekliyor.

**Konum:** `/root/root.txt`

---

### **Alternatif Kısa Yol**
```bash
# docker_user olarak direkt root ol (container'a girmeden)
docker run -v /:/hostroot -it ubuntu bash
chroot /hostroot
cat /root/root.txt
```

Ama bu yöntemle **container flag'ini** kaçırırsın. Lab'ın amacı tüm flag'leri toplamak! 🏴

---

## 🎯 **ÖZET**

| Aşama | Kullanıcı | Konum | Flag | Yapılacak |
|-------|-----------|-------|------|-----------|
| 1 | - | Web | - | LFI ile `/etc/passwd` oku |
| 2 | webuser | Host | `user1.txt` | SSH key bul, bağlan |
| 3 | webuser | Host | - | `docker_user`'ı keşfet |
| 4 | docker_user | Host | - | SSH ile `docker_user`'a geç |
| 5 | docker_user | Host | - | Container başlat |
| 6 | root | Container | `user2.txt` | Container'da flag al |
| 7 | root | Host | `root.txt` | Host disk mount, chroot |

---

## 🧠 **ÖĞRENİLECEK KONULAR**

✅ LFI zafiyeti ile initial access  
✅ Sistem keşfi (enumeration)  
✅ Yatay geçiş (horizontal movement)  
✅ Docker grup yetkileri  
✅ Container başlatma ve yönetme  
✅ Volume mount ile host diskine erişim  
✅ chroot ile kök dizin değiştirme  
✅ Docker'dan yetki yükseltme  

---

# 🏗️ **VAGRANT İLE LAB KURULUMU**

Bu lab'ı kendi bilgisayarında kurmak için **Vagrant** kullanabilirsin.

## **Gereksinimler**
- [VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)

---

## **1. Lab Dizini Oluştur**

```bash
mkdir docker-privesc-lab
cd docker-privesc-lab
```

---

## **2. Vagrantfile Oluştur**

`Vagrantfile` adında bir dosya oluştur ve içine şunu yapıştır:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "docker-lab"
  
  # Network - Host-only adapter for access
  config.vm.network "private_network", ip: "192.168.56.10"
  
  # Forwarded ports - Web server
  config.vm.network "forwarded_port", guest: 80, host: 8080
  
  # Resources
  config.vm.provider "virtualbox" do |vb|
    vb.name = "docker-privesc-lab"
    vb.memory = "2048"
    vb.cpus = 2
  end
  
  # Provision script
  config.vm.provision "shell", path: "provision.sh"
end
```

---

## **3. Provision Script'i Oluştur**

`provision.sh` adında bir dosya oluştur ve içine şunu yapıştır:

```bash
#!/bin/bash

set -e

echo "########################################"
echo "# DOCKER PRIVESC LAB KURULUMU"
echo "########################################"

# Güncellemeler
apt-get update
apt-get upgrade -y

# Gerekli paketler
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common \
    apache2 \
    php \
    libapache2-mod-php \
    vim \
    net-tools \
    unzip

# Docker kurulumu
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io
systemctl enable docker
systemctl start docker

# Kullanıcıları oluştur
useradd -m -s /bin/bash -G www-data webuser
echo "webuser:WebUser123!" | chpasswd

useradd -m -s /bin/bash docker_user
usermod -aG docker docker_user
echo "docker_user:DockerUser123!" | chpasswd

# SSH key'ler
ssh-keygen -t rsa -b 2048 -f /home/webuser/.ssh/id_rsa -q -N ""
cat /home/webuser/.ssh/id_rsa.pub > /home/webuser/.ssh/authorized_keys
chmod 600 /home/webuser/.ssh/authorized_keys
chown -R webuser:webuser /home/webuser/.ssh

ssh-keygen -t rsa -b 2048 -f /home/docker_user/.ssh/id_rsa -q -N ""
cat /home/docker_user/.ssh/id_rsa.pub > /home/docker_user/.ssh/authorized_keys
chmod 600 /home/docker_user/.ssh/authorized_keys
chown -R docker_user:docker_user /home/docker_user/.ssh

# Flag'ler
echo "THM{lfi_ile_giris_basarili_8321}" > /home/webuser/user1.txt
chown webuser:webuser /home/webuser/user1.txt
chmod 644 /home/webuser/user1.txt

echo "THM{hostta_root_oldum_docker_escape_7163}" > /root/root.txt
chmod 600 /root/root.txt

# Container flag'i
echo "THM{containerda_root_oldum_4982}" > /tmp/user2.txt
docker pull ubuntu:latest
docker run -d --name flag_container ubuntu sleep infinity
docker cp /tmp/user2.txt flag_container:/root/user2.txt
docker commit flag_container ubuntu:lab
docker rm -f flag_container
rm /tmp/user2.txt

# LFI zafiyetli web uygulaması
cat > /var/www/html/index.php <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>Docker Lab</title>
</head>
<body>
    <h1>Docker Güvenlik Laboratuvarı</h1>
    <form method="GET">
        <label>Sayfa: <input type="text" name="page" size="50" value="welcome.txt"></label>
        <input type="submit" value="Görüntüle">
    </form>
    <hr>
    <pre><?php
        if(isset(\$_GET['page'])) {
            include(\$_GET['page']);  // LFI ZAFİYETİ!
        } else {
            include('welcome.txt');
        }
    ?></pre>
</body>
</html>
EOF

cat > /var/www/html/welcome.txt <<EOF
Docker Privilege Escalation Laboratuvarı'na hoşgeldin!

Bu sistemde bir LFI zafiyeti bulunmaktadır.
Hedefin: /etc/passwd oku -> SSH key bul -> docker_user'a geç -> container'a gir -> host'a kaçış

Başarılar!
EOF

chown -R www-data:www-data /var/www/html
systemctl restart apache2

# Güvenlik duvarı
ufw allow 22/tcp
ufw allow 80/tcp
ufw --force enable

echo "########################################"
echo "# KURULUM TAMAMLANDI!"
echo "########################################"
echo "VM IP: 192.168.56.10"
echo "Web: http://192.168.56.10"
echo "SSH: ssh webuser@192.168.56.10 (şifre: WebUser123!)"
echo "     ssh docker_user@192.168.56.10 (şifre: DockerUser123!)"
```

---

## **4. VM'i Başlat**

```bash
chmod +x provision.sh
vagrant up
```

Kurulum **5-10 dakika** sürebilir. ☕

---

## **5. Lab'a Bağlan**

### **Web arayüzü:**
```
http://192.168.56.10
```

### **SSH ile:**
```bash
# webuser olarak (şifre: WebUser123!)
ssh webuser@192.168.56.10

# veya SSH key ile
ssh -i webuser_id_rsa webuser@192.168.56.10
```

---

## **6. Lab'ı Durdurma**

```bash
vagrant halt
```

## **7. Lab'ı Silme**

```bash
vagrant destroy -f
```

---

# ✅ **KONTROL LİSTESİ**

- [x] Lab senaryosu anlatıldı
- [x] 3 flag tanımlandı
- [x] Aşamalar adım adım açıklandı
- [x] Vagrant ile kurulum eklendi
- [x] Provision script hazır
- [x] LFI zafiyetli web uygulaması
- [x] Docker grubu yetkisi
- [x] Container flag'i

---

Bu lab ile kullanıcılar:
1. LFI zafiyetini keşfedecek
2. Yatay geçiş yapacak
3. Docker yetkilerini kullanacak
4. Container'dan host'a kaçacak
5. 3 farklı flag toplayacak

---
