# Tugas Modul 4 — DMZ & Firewall Configuration

---

## 1. Topologi Jaringan
<img width="1296" height="1158" alt="Screenshot 2026-06-06 232115" src="https://github.com/user-attachments/assets/c6263efb-56e9-4cda-841c-55d0bd923ffd" />

```


```

Topologi yang digunakan terdiri dari beberapa zona jaringan:

| Zona | Deskripsi |
|------|-----------|
| **Outside / WAN** | Jaringan luar, diakses oleh Client WAN (Linux2) via MikroTik ISP |
| **DMZ** | Zona semi-publik tempat Ubuntu Server (web server) berada |
| **LAN** | Jaringan internal, diakses Client LAN (Linux3) via Cisco Router |

---

## 2. Tabel IP Address

| Perangkat | Interface | IP Address | Gateway | Keterangan |
|-----------|-----------|------------|---------|------------|
| MikroTik ISP | ether1 | DHCP Client | DHCP dari jaringan lab | Terhubung ke Cloud/lab |
| MikroTik ISP | ether2 | 10.10.10.1/30 | — | Terhubung ke FortiGate port1 |
| MikroTik ISP | ether3 | 172.16.100.1/24 | — | Gateway untuk Client WAN |
| FortiGate | port1 | 10.10.10.2/30 | 10.10.10.1 | Interface WAN |
| FortiGate | port2 | 10.20.20.1/30 | — | Interface INSIDE ke Cisco |
| FortiGate | port3 | 192.168.20.1/24 | — | Interface DMZ |
| Cisco Router | G0/0 | 10.20.20.2/30 | — | Terhubung ke FortiGate port2 |
| Cisco Router | G0/1 | 192.168.10.1/24 | — | Gateway LAN |
| Client LAN (Linux3) | eth0 | 192.168.10.10/24 | 192.168.10.1 | Client internal |
| Client WAN (Linux2) | eth0 | 172.16.100.10/24 | 172.16.100.1 | Client luar |
| Ubuntu Server DMZ (Linux) | eth0 | 192.168.20.10/24 | 192.168.20.1 | Web server DMZ |

---

## 3. Konfigurasi Tiap Perangkat

### 3.1 MikroTik ISP
<img width="1336" height="1220" alt="Screenshot 2026-06-06 232523" src="https://github.com/user-attachments/assets/2a0cd9d4-480a-42db-9f00-a1cb0f84b65d" />
```
```

Konfigurasi yang diterapkan:
- DHCP Client pada ether1 untuk mendapat IP dari jaringan lab
- IP statis pada ether2 (10.10.10.1/30) ke FortiGate
- IP statis pada ether3 (172.16.100.1/24) sebagai gateway Client WAN
- NAT Masquerade pada out-interface ether1
- Static route ke 192.168.10.0/24 dan 192.168.20.0/24 via 10.10.10.2

---

### 3.2 FortiGate (Firewall Utama)

<img width="876" height="1228" alt="Screenshot 2026-06-06 232530" src="https://github.com/user-attachments/assets/4d48009c-fb10-4b1e-b7b8-b0a02fa12e10" />


```
[Screenshot: Konfigurasi interface port1, port2, port3]
```

<img width="1812" height="1230" alt="Screenshot 2026-06-06 232538" src="https://github.com/user-attachments/assets/7d4f9ff5-d2de-4a86-a943-3553ef94cb91" />


```
[Screenshot: Static route dan default route]
```

<img width="1336" height="1220" alt="Screenshot 2026-06-06 232551" src="https://github.com/user-attachments/assets/c5447c13-819f-4d3b-8f86-04a83f2659d8" />

```
[Screenshot: Policy LAN_to_WAN, LAN_to_DMZ, WAN_to_DMZ_HTTP]
```

<img width="2230" height="1216" alt="Screenshot 2026-06-06 232601" src="https://github.com/user-attachments/assets/167f51da-0e78-4d19-8e05-de0b8bb7e086" />


```
[Screenshot: VIP VIP_DMZ — port forwarding 10.10.10.2:80 → 192.168.20.10:80]
```

Konfigurasi yang diterapkan:
- Interface port1 (WAN): 10.10.10.2/30
- Interface port2 (INSIDE): 10.20.20.1/30
- Interface port3 (DMZ): 192.168.20.1/24
- Default route via 10.10.10.1 (MikroTik)
- Static route ke LAN 192.168.10.0/24 via 10.20.20.2
- Policy `LAN_to_WAN`: port2 → port1, NAT enable
- Policy `LAN_to_DMZ`: port2 → port3, NAT disable
- Policy `WAN_to_DMZ_HTTP`: port1 → port3, service HTTP
- VIP `VIP_DMZ`: extip 10.10.10.2 → mappedip 192.168.20.10, port 80

---

### 3.3 Cisco Router

<img width="1348" height="1232" alt="Screenshot 2026-06-06 232609" src="https://github.com/user-attachments/assets/8a0d7e78-f103-4431-bd52-e88dc50f5a92" />


```
[Screenshot: Interface G0/0 dan G0/1 dengan IP masing-masing]
```

<img width="1360" height="1232" alt="Screenshot 2026-06-06 232617" src="https://github.com/user-attachments/assets/c1007674-5417-4919-8c6a-74143488d0ab" />


```
[Screenshot: Default route S* 0.0.0.0/0 via 10.20.20.1]
```

Konfigurasi yang diterapkan:
- G0/0: 10.20.20.2/30 (ke FortiGate)
- G0/1: 192.168.10.1/24 (gateway LAN)
- Default route: 0.0.0.0/0 via 10.20.20.1

---

### 3.4 Ubuntu Server DMZ

<img width="1336" height="1232" alt="Screenshot 2026-06-06 232638" src="https://github.com/user-attachments/assets/9303cc2e-76f2-4351-b2a2-4c0b2eb34065" />


```
[Screenshot: IP address 192.168.20.10/24, gateway 192.168.20.1, nginx active]
```

Konfigurasi yang diterapkan:
- IP statis: 192.168.20.10/24
- Gateway: 192.168.20.1
- Nginx terinstall dan aktif (`systemctl enable nginx`)
- Halaman default diubah sesuai format: `Tumod_4_DMZ_Firewall_No.kel-nama`

---

### 3.5 Client LAN (Linux3)

<img width="1356" height="1252" alt="Screenshot 2026-06-06 233821" src="https://github.com/user-attachments/assets/adb971b3-633a-4178-88b2-6bc1e74f0e79" />


```
[Screenshot: ifconfig eth0 dan route -n Client LAN]
```

Konfigurasi yang diterapkan (dari `ifconfig eth0` dan `route -n`):
- IP: `192.168.10.10/24` (Mask: 255.255.255.0)
- Default gateway: `192.168.10.1` (via eth0 → Cisco Router G0/1)
- Network: `192.168.10.0/24` terhubung langsung via eth0

---

### 3.6 Client WAN (Linux2)
<img width="1370" height="1236" alt="Screenshot 2026-06-06 233829" src="https://github.com/user-attachments/assets/ed1bc137-6956-4e5f-ac8c-8be771fde3dc" />

```
[Screenshot: ifconfig eth0 dan route -n Client WAN]
```

Konfigurasi yang diterapkan (dari `ifconfig eth0` dan `route -n`):
- IP: `172.16.100.10/24` (Mask: 255.255.255.0)
- Default gateway: `172.16.100.1` (via eth0 → MikroTik ISP ether3)
- Network: `172.16.100.0/24` terhubung langsung via eth0

---

## 4. Hasil Pengujian

### 4.1 Client LAN ping ke Gateway Cisco (192.168.10.1)
<img width="1630" height="1220" alt="Screenshot 2026-06-06 232354" src="https://github.com/user-attachments/assets/425417ad-c448-4fa7-90ad-6b85cb1631e7" />

```
```

**Hasil:** ✅ Berhasil — 5 packets transmitted, 5 received, 0% packet loss

---

### 4.2 Client LAN ping ke FortiGate (10.20.20.1)

<img width="1580" height="1210" alt="Screenshot 2026-06-06 232403" src="https://github.com/user-attachments/assets/8a4a9767-3fa9-4cef-820f-dd3c3e838e3d" />

```
[Screenshot: ping 10.20.20.1 — BERHASIL]
```

**Hasil:** ✅ Berhasil — 5 packets transmitted, 5 received, 0% packet loss

---

### 4.3 Client LAN ping ke Server DMZ (192.168.20.10)

<img width="1654" height="1230" alt="Screenshot 2026-06-06 232409" src="https://github.com/user-attachments/assets/8491c37d-3022-49d4-865d-514878eda564" />


```
[Screenshot: ping 192.168.20.10 — BERHASIL]
```

**Hasil:** ✅ Berhasil — 5 packets transmitted, 5 received, 0% packet loss

---

### 4.4 Client LAN akses web DMZ via browser (http://192.168.20.10)

<img width="1646" height="1214" alt="Screenshot 2026-06-06 232417" src="https://github.com/user-attachments/assets/f63f408f-fcf7-41b5-923a-bdc6037980ec" />


```
[Screenshot: Firefox menampilkan halaman Nginx DMZ — BERHASIL]
```

**Hasil:** ✅ Berhasil — halaman web menampilkan teks identitas kelompok

---

### 4.5 Client WAN ping ke FortiGate (10.10.10.2)

<img width="1612" height="1232" alt="Screenshot 2026-06-06 232425" src="https://github.com/user-attachments/assets/511c09e6-ec90-4fb7-af0c-f9cf9352c76d" />


```
[Screenshot: ping 10.10.10.2 — BERHASIL]
```

**Hasil:** ✅ Berhasil — 5 packets transmitted, 5 received, 0% packet loss

---

### 4.6 Client WAN ping ke ISP MikroTik (172.16.100.1)

<img width="1612" height="1214" alt="Screenshot 2026-06-06 232433" src="https://github.com/user-attachments/assets/03910cc8-9777-4aad-adea-54f4b65574da" />


```
[Screenshot: ping 172.16.100.1 — BERHASIL]
```

**Hasil:** ✅ Berhasil — 5 packets transmitted, 5 received, 0% packet loss

---

### 4.7 Client WAN akses web DMZ via VIP (http://10.10.10.2)

<img width="1656" height="1216" alt="Screenshot 2026-06-06 232441" src="https://github.com/user-attachments/assets/096e74a5-c928-4422-9225-fdda20db7cd9" />


```
[Screenshot: Firefox menampilkan halaman Nginx via VIP — BERHASIL]
```

**Hasil:** ✅ Berhasil — VIP berhasil meneruskan request HTTP ke server DMZ (192.168.20.10)

---

### 4.8 Client WAN ping ke Client LAN (192.168.10.10) — HARUS GAGAL

<img width="1614" height="1226" alt="Screenshot 2026-06-06 232448" src="https://github.com/user-attachments/assets/e3cf47f6-69e0-4bbf-a9fc-dc51c09a7d26" />

```
[Screenshot: ping 192.168.10.10 dari Client WAN — GAGAL / 100% packet loss]
```

**Hasil:** ✅ Gagal (sesuai ekspektasi) — LAN tidak dapat diakses langsung dari WAN, membuktikan firewall berfungsi

---

### 4.9 Client WAN ping ke IP asli DMZ (192.168.20.10) — HARUS GAGAL

<img width="2164" height="1258" alt="Screenshot 2026-06-06 232504" src="https://github.com/user-attachments/assets/e3bd7e1e-3862-4766-b178-4c93f2400238" />


```
[Screenshot: ping 192.168.20.10 dari Client WAN — GAGAL / 100% packet loss]
```

**Hasil:** ✅ Gagal (sesuai ekspektasi) — IP asli server DMZ tidak dapat diakses langsung dari WAN, hanya bisa via VIP 10.10.10.2

---

### 4.10 Server DMZ ping ke Client LAN (192.168.10.10) — HARUS GAGAL

<img width="1624" height="1238" alt="Screenshot 2026-06-06 232514" src="https://github.com/user-attachments/assets/fc59224f-c9ff-4bab-aadc-78fe8eedd2a1" />


```
[Screenshot: ping 192.168.10.10 dari Server DMZ — GAGAL / 100% packet loss]
```

**Hasil:** ✅ Gagal (sesuai ekspektasi) — Server DMZ tidak dapat mengakses jaringan LAN secara langsung

---

### Ringkasan Hasil Pengujian

| No | Skenario Pengujian | Ekspektasi | Hasil |
|----|-------------------|------------|-------|
| 1 | Client LAN → Gateway Cisco | Berhasil | ✅ |
| 2 | Client LAN → FortiGate port2 | Berhasil | ✅ |
| 3 | Client LAN → Server DMZ (ping) | Berhasil | ✅ |
| 4 | Client LAN → Server DMZ (HTTP browser) | Berhasil | ✅ |
| 5 | Client WAN → FortiGate port1 | Berhasil | ✅ |
| 6 | Client WAN → ISP MikroTik | Berhasil | ✅ |
| 7 | Client WAN → Server DMZ via VIP (HTTP browser) | Berhasil | ✅ |
| 8 | Client WAN → Client LAN (ping) | **Gagal** | ✅ |
| 9 | Client WAN → IP asli DMZ (ping) | **Gagal** | ✅ |
| 10 | Server DMZ → Client LAN (ping) | **Gagal** | ✅ |

---

## 5. Analisis dan Kesimpulan

### 5.1 Analisis Keamanan Jaringan

Implementasi topologi DMZ dengan FortiGate sebagai firewall utama berhasil menciptakan tiga zona jaringan yang terisolasi dengan baik:

**Zona WAN (Luar):**  
Client WAN yang berada di jaringan 172.16.100.0/24 hanya dapat mengakses server DMZ melalui mekanisme VIP (Virtual IP / Port Forwarding) pada alamat 10.10.10.2 port 80. Akses langsung ke IP asli server DMZ (192.168.20.10) maupun ke jaringan LAN (192.168.10.0/24) diblokir sepenuhnya oleh firewall policy FortiGate. Hal ini membuktikan bahwa implementasi NAT dan policy firewall berjalan dengan benar.

**Zona DMZ:**  
Ubuntu Server yang berfungsi sebagai web server berhasil melayani request HTTP dari dua arah: dari Client LAN secara langsung (via IP asli 192.168.20.10) maupun dari Client WAN melalui VIP. Namun, server DMZ tidak memiliki akses ke jaringan LAN, sehingga jika terjadi kompromi pada server DMZ, dampaknya tidak langsung merambat ke jaringan internal. Ini adalah prinsip utama arsitektur DMZ.

**Zona LAN (Dalam):**  
Client LAN dapat mengakses internet (melalui FortiGate dengan NAT ke WAN) maupun server DMZ. Routing berjalan melalui Cisco Router → FortiGate port2 → FortiGate port3 → Server DMZ. Jaringan LAN sepenuhnya tersembunyi dari sisi WAN karena NAT dilakukan di FortiGate.

### 5.2 Peran Setiap Perangkat

- **MikroTik ISP** mensimulasikan peran ISP dengan memberikan konektivitas ke "internet" (jaringan lab), sekaligus menjadi gateway untuk Client WAN. NAT Masquerade pada ether1 memastikan semua traffic dari jaringan internal dapat keluar ke jaringan lab.

- **FortiGate** adalah inti dari sistem keamanan. Dengan tiga interface (WAN, INSIDE, DMZ), FortiGate mengatur semua traffic antar zona menggunakan firewall policy yang granular. VIP memungkinkan akses dari luar ke server internal tanpa mengekspos IP aslinya.

- **Cisco Router** berperan sebagai router internal yang menghubungkan FortiGate dengan jaringan LAN. Default route ke FortiGate memastikan semua traffic keluar melewati firewall terlebih dahulu.

### 5.3 Kesimpulan

Tugas modul ini berhasil diimplementasikan sesuai target yang ditetapkan. Semua skenario pengujian menghasilkan output yang diharapkan — baik pengujian yang seharusnya berhasil maupun yang seharusnya gagal. 

Arsitektur DMZ terbukti efektif dalam memisahkan server publik dari jaringan internal. Pengguna dari luar hanya dapat berinteraksi dengan layanan yang sengaja dipublikasikan (web server via VIP), tanpa bisa mengakses infrastruktur internal secara langsung. Firewall FortiGate berhasil menegakkan kebijakan keamanan tersebut melalui kombinasi firewall policy, NAT, dan Virtual IP.

Konsep yang dipelajari dalam modul ini — segmentasi jaringan, DMZ, NAT, firewall policy, dan port forwarding — merupakan fondasi penting dalam desain keamanan jaringan enterprise di dunia nyata.

---


