# 3proxy Sunucu Kurulum ve Yapılandırma Rehberi

Bu doküman, bir Linux VDS üzerinde birden fazla IP adresini farklı portlar üzerinden yönetmek ve güçlü kimlik doğrulama mekanizmalarıyla bir proxy tünelleme katmanı oluşturmak amacıyla hazırlanmıştır.

## İçindekiler
1. Sistem Gereksinimleri ve Güncelleme
2. Kaynak Kodundan Derleme ve Kurulum
3. Güvenlik Duvarı Yapılandırması
4. Sunucu Konfigürasyonu
5. Servis Yönetimi ve İzleme

---

## 1. Sistem Gereksinimleri ve Güncelleme

Kurulum işlemine başlamadan önce sistem paket listesi güncellenmeli ve derleme için gerekli temel araçlar kurulmalıdır.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential wget -y
```

---

## 2. Kaynak Kodundan Derleme ve Kurulum

3proxy yazılımının kaynak kodu resmi deposundan indirilir, geçici dizin altında derlenir ve sistem dizinlerine konumlandırılır.

```bash
cd /tmp
wget https://github.com/z3APA3A/3proxy/archive/refs/tags/0.9.4.tar.gz
tar -zxvf 0.9.4.tar.gz
cd 3proxy-0.9.4
make -f Makefile.Linux
```

Derleme tamamlandıktan sonra gerekli dizin yapısı oluşturulur ve çalıştırılabilir dosya ilgili konuma kopyalanır.

```bash
sudo mkdir -p /usr/local/etc/3proxy /usr/local/etc/3proxy/logs /usr/local/bin
sudo cp bin/3proxy /usr/local/bin/
```

---

## 3. Güvenlik Duvarı Yapılandırması

Harici uygulamalar ve tarayıcı profilleri üzerinden erişim sağlanabilmesi için belirlenen port aralığı (3001 - 3010) güvenlik duvarında açılmalıdır.

```bash
sudo ufw allow 3001:3010/tcp
sudo ufw reload
sudo ufw status
```

---

## 4. Sunucu Konfigürasyonu

`/usr/local/etc/3proxy/3proxy.cfg` dosyası oluşturulur veya güncellenir. Aşağıdaki şablon, 10 farklı portu farklı harici IP adreslerine bağlayan örnek bir yapılandırmayı içermektedir.

```text
daemon
maxconn 2000
nscache 65536
timeouts 1 5 30 60 180 1800 15 60

auth strong
users admin:CL:YOUR_SECURE_PASSWORD
allow admin

log /usr/local/etc/3proxy/logs/3proxy.log D
logformat "- +_L%t.%. %N.%p %E %U %C:%c %R:%r %O %I %h %T"

# 1. IP / Arayüz -> Port 3001
external YOUR_VDS_IP_1
internal 0.0.0.0
proxy -p3001

# 2. IP / Arayüz -> Port 3002
external YOUR_VDS_IP_2
internal 0.0.0.0
proxy -p3002

# 3. IP / Arayüz -> Port 3003
external YOUR_VDS_IP_3
internal 0.0.0.0
proxy -p3003

# 4. IP / Arayüz -> Port 3004
external YOUR_VDS_IP_4
internal 0.0.0.0
proxy -p3004

# 5. IP / Arayüz -> Port 3005
external YOUR_VDS_IP_5
internal 0.0.0.0
proxy -p3005
```

---

## 5. Servis Yönetimi ve İzleme

### Sunucuyu Başlatma
Hazırlanan konfigürasyon dosyası ile proxy servisi arka planda çalıştırılır.

```bash
sudo 3proxy /usr/local/etc/3proxy/3proxy.cfg
```

### Durum ve Port Kontrolü
Servisin çalıştığı süreçler ve dinlenen portlar kontrol edilir.

```bash
ps aux | grep 3proxy
sudo ss -tulpn | grep 3proxy
```

### Log Kayıtlarını Gözlemleme
Bağlantı denemelerini ve akış trafiğini takip etmek için log dosyası izlenebilir.

```bash
tail -f /usr/local/etc/3proxy/logs/3proxy.log
```
