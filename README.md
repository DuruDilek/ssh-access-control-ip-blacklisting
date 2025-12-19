🔐 SSH Erişim Denetimi ve Otomatik IP Kara Listeleme
(Fail2Ban + iptables + cron + e-posta bildirimi)

Bu proje, Linux sistemlerde SSH brute-force saldırılarını önlemek amacıyla
Fail2Ban, iptables, cron ve e-posta bildirimi kullanılarak geliştirilmiştir.

Amaç; SSH servisine yapılan başarısız giriş denemelerini otomatik olarak tespit etmek,
saldırgan IP adreslerini geçici olarak engellemek ve yöneticiye mail üzerinden rapor göndermektir.

🎯 Projenin Amacı

SSH brute-force saldırılarını engellemek

Belirli sayıda hatalı denemeden sonra IP’yi otomatik banlamak

Banlanan IP’leri iptables üzerinden yönetmek

Günlük olarak banlanan IP’leri e-posta ile raporlamak


🛠 Kullanılan Teknolojiler

Linux

Fail2Ban

iptables

cron

msmtp (mail gönderimi)

Bash Script



⚙️ Sistem Mimarisi

SSH Logları
     ↓
Fail2Ban
     ↓
iptables (IP Ban)
     ↓
Bash Script (banlı-ipler.sh)
     ↓
cron (günlük çalıştırma)
     ↓
E-posta Raporu



🔐 Fail2Ban Yapılandırması (jail.local)

Genel Ayarlar
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1
bantime = 180
findtime = 600
maxretry = 3
backend = systemd
usedns = warn


📌 Açıklama:

3 başarısız deneme → IP banlanır

10 dakika içinde sayım yapılır

3 dakika boyunca erişim engellenir

SSH Jail Tanımı
[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
backend = systemd
findtime = 600
bantime = 180
maxretry = 3
action = %(action_mwl)s


✔️ SSH servisi aktif olarak izlenir
✔️ Banlanan IP’ler otomatik olarak iptables’a eklenir
✔️ E-posta bildirimi tetiklenir

🔥 IP Kara Listeleme (iptables)

Fail2Ban, SSH saldırısı algıladığında ilgili IP adresini otomatik olarak:

iptables -A INPUT -s <IP> -j DROP


komutu ile engeller.


📜 Banlanan IP’leri Listeleyen Script

banli-ipler.sh
#!/bin/bash
echo "Banlanan IP adresleri:"
sudo iptables -L -n --line-numbers | grep "DROP"


📌 Bu script:

Sistemde DROP edilen IP’leri listeler

cron tarafından otomatik çalıştırılır

⏰ Otomatik Günlük Raporlama (cron)
0 8 * * * /home/aysegul/banli-ipler.sh | mail -s "Banlanan IP'ler" tatliboncukodev@gmail.com


📬 Her gün saat 08:00’de:

Banlanan IP’ler listelenir

Yöneticiye e-posta olarak gönderilir

📧 E-posta Yapılandırması (msmtp)
.msmtprc
defaults
auth on
tls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt

account gmail
host smtp.gmail.com
port 587
from tatliboncukodev@gmail.com
user tatliboncukodev@gmail.com
password ************

account default : gmail


📌 Gmail SMTP kullanılarak güvenli mail gönderimi sağlanmıştır.



🧪 Test Ortamı ve Doğrulama (PuTTY)

Projenin test sürecinde, SSH brute-force saldırılarını gerçekçi bir senaryo ile simüle etmek amacıyla PuTTY kullanılmıştır.

🔹 Kullanılan Araç

PuTTY (Windows SSH Client)

🔹 Test Senaryosu

Windows işletim sistemi üzerinden PuTTY açıldı

Linux sunucunun IP adresi ve SSH (22) portu girildi

Aynı kullanıcı adı ile bilerek hatalı parola kullanılarak
3 kez başarısız giriş denemesi yapıldı

Fail2Ban, SSH loglarını izleyerek saldırıyı tespit etti

İlgili IP adresi otomatik olarak iptables üzerinden banlandı

🔹 Beklenen ve Gerçekleşen Sonuçlar

✔️ 3 hatalı SSH denemesi sonrası IP adresi banlandı
✔️ PuTTY üzerinden SSH bağlantısı kesildi
✔️ Sunucuya erişim engellendi
✔️ IP adresi iptables DROP kuralına eklendi
✔️ Banlanan IP günlük rapora dahil edildi

<img width="793" height="464" alt="resim" src="https://github.com/user-attachments/assets/a01c990c-1ba0-4cfb-9b64-3fba8f1831e4" />


🧠 Projede Kazanımlar

Linux sistemlerde SSH güvenliği

Fail2Ban ile saldırı tespiti

iptables ile otomatik IP engelleme

cron ile zamanlanmış görevler

Bash script yazımı

Gerçek dünya siber güvenlik senaryosu


