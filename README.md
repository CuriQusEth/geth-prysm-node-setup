# Sepolia Geth + Prysm RPC Setup (Aztec Ready)

Bu rehber, kendi VPS sunucunuzda **Sepolia için Geth (Execution) + Prysm (Beacon) RPC** kurulumunu sadece tek komutla yapmanızı sağlar.
Her şey Docker kullanılarak otomatik kurulur.
Bu RPC'leri **Aztec Sequencer** node'larınızda kullanabilir ve hiçbir sorun yaşamazsınız.

---

## Sistem Gereksinimleri

| Gereksinim      | Detaylar          |
| --------------- | ----------------- |
| Depolama        | 600 GB - 1 TB SSD |
| RAM             | En az 16 GB       |
| CPU             | 6-8 Çekirdek      |
| İşletim Sistemi | Ubuntu 22.04+     |

---

## 1- Tek Komutla Kurulum

```bash
[ -f "new_script.sh" ] || curl -sSL -o new_script.sh https://raw.githubusercontent.com/CuriQusEth/geth-prysm-node-setup/main/script.sh; \
apt update -y && apt install curl -y && \
chmod +x new_script.sh && ./new_script.sh
```

Kurulum sonunda Geth ve Prysm Docker içinde otomatik başlar.

---

## 2- Sync Durumu Kontrolü

Kurulum sonrası **senkronizasyon durumunu** anlık kontrol etmek için:

```bash
bash <(curl -s https://raw.githubusercontent.com/CuriQusEth/geth-prysm-node-setup/main/sekronize-kontrol.sh)
```

### Örnek Çıktı - Senkronize OLDU

![Ekran görüntüsü 2025-05-15 144800](https://github.com/user-attachments/assets/aeffb9e8-3e9f-4232-804b-4429ea75a62f)

### Örnek Çıktı - HENÜZ Devam Ediyor

![WhatsApp Görsel 2025-05-15 saat 18 53 38_0ae9c891](https://github.com/user-attachments/assets/c8a0924b-4cd7-4c70-838f-fc713ef6c686)

---

## 3- Logları İzleme

```bash
docker logs -f geth
```

### Örnek Çıktı - Kurulum Devam Ediyorsa

![Ekran görüntüsü 2025-05-15 013036](https://github.com/user-attachments/assets/abe32766-a61b-4131-a21c-9a8c2412bdcc)

### Örnek Çıktı - Kurulum Bittiyse

![Ekran görüntüsü 2025-05-16 192151](https://github.com/user-attachments/assets/d81a0281-ab91-471f-af62-bc0063485313)

---

### Geth (Execution Node)

* Sepolia ağı FULL NODE kuruyorsun, ilk senkronizasyonu çok uzun sürer.
* **600 GB - 1 TB** arası veri indirir.
* **1 gün kadar sürebilir**, ağ hızın ve VPS performansına bağlı.
* Disk'i aşağıdaki komut ile dolup dolmadığını kontrol edebilirsin:

```bash
df -h
```

### Prysm (Beacon Node)

* Çok daha hızlı senkronize olur.
* Genelde 1-2 saatte başlar ama tam senkronizasyon Geth'e bağlıdır.

- **Sabırlı olun.**
- Geth %100 olmadan Aztec node hatalar verir.

---

## 4- VPS Güvenlik Duvarı Ayarları (Mutlaka Yapılmalı)

### A- Duvarı Aktif Et

```bash
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw enable
```

### B- Geth P2P Bağlantılarına İzin Ver

```bash
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
```

### C- Hem Lokal Hem Dış IP İçin Geth & Prysm RPC Portlarını Aç

```bash
sudo ufw allow 8545/tcp
sudo ufw allow 3500/tcp
sudo ufw allow from 127.0.0.1 to any port 8545 proto tcp
sudo ufw allow from 127.0.0.1 to any port 3500 proto tcp
```

### D- Duvar Ayarlarını Uygula

```bash
sudo ufw reload
```

---

## 5- RPC Adreslerini Doğru Kullanma (CLI ve Docker Ayrımı)

📌 Aztec Sequencer kurarken **Geth ve Prysm RPC'lerini doğru yazmazsan node hata verir.**
Bu yüzden **Docker kuranlarla CLI kuranların kullanacağı adresler farklıdır.**

---

### ✅ Geth Execution RPC

| Kurulum Türü                         | Doğru Adres Örneği                             | Açıklama                                                           |
| ------------------------------------ | ---------------------------------------------- | ------------------------------------------------------------------ |
| **Docker Compose ile kurulum**       | http://127.0.0.1:8545                          | Docker Bridge kullanır. Bu yüzden **localhost** zorunlu.           |
| **CLI (bu script) ile kurulum**      | http://vps-ip:8545                             | Dış IP üzerinden kullanılmalı. Docker olmadığı için IP fark etmez. |

### Özet

* **Docker ile kurduysan:** Aztec Sequencer node içinde **mutlaka `http://127.0.0.1:8545` kullan.**
* **Bu script ile kurduysan:** `http://vps-ip:8545` şeklinde sunucunun dış IP adresini kullan.

---

### ✅ Prysm Beacon RPC

| Kurulum Türü                         | Doğru Adres Örneği                             | Açıklama                                                         |
| ------------------------------------ | ---------------------------------------------- | ---------------------------------------------------------------- |
| **Docker Compose ile kurulum**       | http://127.0.0.1:3500                          | Docker Bridge kullanır. Sadece **localhost** üzerinden erişilir. |
| **CLI (bu rehber) ile kurulum**      | http://kendi-sunucu-ip:3500                    | Direkt VPS IP'si üzerinden kullanılır.                           |

### Özet

* **Docker ile kurduysan:** Aztec Sequencer node'a `http://127.0.0.1:3500` yaz.
* **Bu script ile kurduysan:** `http://kendi-sunucu-ip:3500` şeklinde VPS IP kullanılır.

---

### 📌 KURAL

* **Docker Compose:**
  * Sequencer node'un **aynı sunucuda çalışıyorsa:** `localhost`
  * Farklı sunucudan kullanılamaz.

* **CLI (bu rehber ile yaptıysan):**
  * Hem içeride hem dışarıda **VPS IP'si** kullanılır.
  * **localhost kullanırsan hata alırsın.**

---

### ⚠️ Unutma

* **Docker Compose izolasyonlu çalışır. Sadece iç ağdan (localhost) erişilir.**
* **Bu rehber yani CLI ile kurduysan VPS IP'ni girerek direkt kullanabilirsin.**

---

## Ulaşmak & Sorularınız İçin

[https://x.com/wilson_syo](https://x.com/wilson_syo)

---
