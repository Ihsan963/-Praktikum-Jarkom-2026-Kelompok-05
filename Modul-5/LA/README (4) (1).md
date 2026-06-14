






## 📋 Daftar Isi

- [Gambaran Umum](#gambaran-umum)
- [Topologi Jaringan](#topologi-jaringan)
- [Perangkat yang Digunakan](#perangkat-yang-digunakan)
- [Tabel Addressing](#tabel-addressing)
- [Modul 1 — Cisco Switch Jakarta](#modul-1--cisco-switch-jakarta)
- [Modul 2 — Cisco Router Jakarta](#modul-2--cisco-router-jakarta)
- [Modul 3 — MikroTik Router Jakarta](#modul-3--mikrotik-router-jakarta)
- [Modul 4 — Ubuntu Server Jakarta](#modul-4--ubuntu-server-jakarta)
- [Modul 5 — FortiGate Jakarta](#modul-5--fortigate-jakarta)
- [Modul 6 — MikroTik ISP](#modul-6--mikrotik-isp)
- [Modul 7 — Switch & MikroTik Surabaya](#modul-7--switch--mikrotik-surabaya)
- [Modul 8 — FortiGate Surabaya](#modul-8--fortigate-surabaya)
- [Modul 9 — Verifikasi GRE Tunnel & OSPF](#modul-9--verifikasi-gre-tunnel--ospf)
- [Modul 10 — Pengujian Akhir](#modul-10--pengujian-akhir)

---

## 📖 Gambaran Umum

Proyek ini merupakan simulasi jaringan enterprise skala menengah yang terdiri dari dua site:

| Site | Lokasi | Fungsi |
|------|--------|--------|
| **HQ** | Jakarta | Kantor Pusat — Cisco Router, MikroTik Router, Ubuntu Server, FortiGate |
| **Branch** | Surabaya | Kantor Cabang — MikroTik Router, Cisco Switch, FortiGate |
| **ISP** | — | Simulasi Provider — MikroTik RouterOS |

Kedua site dihubungkan menggunakan **GRE Tunnel** antar FortiGate, dengan **OSPF** berjalan di atas tunnel untuk pertukaran routing dinamis antar site.

---

## 🗺️ Topologi Jaringan
<img width="1278" height="732" alt="Screenshot 2026-06-14 055013" src="https://github.com/user-attachments/assets/cab9a957-90e4-4291-a2e2-3cab57ba16b0" />


---

## 🖥️ Perangkat yang Digunakan

| Perangkat | OS / Tipe | Fungsi |
|-----------|-----------|--------|
| **vIOS-Jakarta** | Cisco IOS (vIOS) | Sub-interface VLAN, VRRP Master VLAN 10 & 60, DHCP Relay |
| **Switch-Jakarta** | Cisco IOS Switch | VLAN 10/20/60, Trunk ke Cisco & MikroTik |
| **Mikrotik-Jakarta** | MikroTik RouterOS | VLAN interface, VRRP Master VLAN 20, DHCP Relay |
| **Ubuntu VLAN60** | Ubuntu Server | ISC-DHCP Server (VLAN 10 & 20) + Nginx Web Server |
| **FortiGate Jakarta** | FortiOS (VM64-KVM) | Edge Firewall, NAT, GRE Tunnel endpoint, OSPF |
| **Mikrotik-ISP** | MikroTik RouterOS | Simulasi ISP, NAT Masquerade ke Cloud |
| **FortiGate Surabaya** | FortiOS (VM64-KVM) | Edge Firewall, NAT, GRE Tunnel endpoint, OSPF |
| **Mikrotik Surabaya** | MikroTik RouterOS | Gateway VLAN 30 (DHCP) & VLAN 40 |
| **Switch Surabaya** | Cisco IOS Switch | VLAN 30/40, Trunk ke MikroTik Surabaya |
| **VLAN10 / VLAN20** | VPCS | Client Jakarta (Finance & IT) |
| **VLAN30** | VPCS | Client Surabaya Sales (DHCP) |
| **VLAN40** | VPCS | Client Surabaya Operations (Static IP) |
| **Tinycore VLAN40** | Tinycore Linux | Client Surabaya Operations (browser test) |

---

## 📊 Tabel Addressing

### Jakarta / HQ

#### VLAN Jakarta
| VLAN | Nama | Network | Gateway Virtual (VRRP) | DHCP Server |
|------|------|---------|------------------------|-------------|
| 10 | FINANCE | 192.168.10.0/24 | 192.168.10.1 | Ubuntu Server Jakarta |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 | Ubuntu Server Jakarta |
| 60 | SERVER-HQ | 192.168.60.0/24 | 192.168.60.1 | — (Static) |

#### Cisco Router Jakarta (vIOS-Jakarta)
| Interface | IP Address | Keterangan |
|-----------|------------|------------|
| Gi0/0 | 10.10.100.1/30 | Link ke FortiGate Jakarta port1 |
| Gi0/1.10 | 192.168.10.2/24 | Sub-interface VLAN 10 |
| Gi0/1.20 | 192.168.20.2/24 | Sub-interface VLAN 20 |
| Gi0/1.60 | 192.168.60.2/24 | Sub-interface VLAN 60 |

#### MikroTik Router Jakarta
| Interface | IP Address | Keterangan |
|-----------|------------|------------|
| ether1 | 10.10.101.2/30 | Link ke FortiGate Jakarta port2 |
| vlan10-finance | 192.168.10.3/24 | IP fisik VLAN 10 |
| vlan20-it | 192.168.20.3/24 | IP fisik VLAN 20 |
| vlan60-server | 192.168.60.3/24 | IP fisik VLAN 60 |

#### VRRP Jakarta
| VLAN | Virtual IP | Master | Backup |
|------|------------|--------|--------|
| 10 | 192.168.10.1 | Cisco Router (priority 110) | MikroTik (priority 90) |
| 20 | 192.168.20.1 | MikroTik (priority 110) | Cisco Router (priority 90) |
| 60 | 192.168.60.1 | Cisco Router (priority 110) | MikroTik (priority 90) |

#### FortiGate Jakarta
| Interface | IP | Terhubung ke |
|-----------|-----|-------------|
| port1 | 10.10.100.2/30 | Cisco Router Jakarta |
| port2 | 10.10.101.1/30 | MikroTik Jakarta |
| port3 | 10.0.12.2/30 | MikroTik ISP |
| GRE-JKT-SBY | 172.16.0.1/32 | Tunnel ke FortiGate Surabaya |

#### Ubuntu Server Jakarta
| IP Address | Gateway | Service |
|------------|---------|---------|
| 192.168.60.10/24 | 192.168.60.1 | ISC-DHCP Server + Nginx Web Server |

#### DHCP Pool Jakarta
| VLAN | Range | Gateway |
|------|-------|---------|
| 10 | 192.168.10.100 – 192.168.10.200 | 192.168.10.1 |
| 20 | 192.168.20.100 – 192.168.20.200 | 192.168.20.1 |

---

### ISP

| Interface | IP | Keterangan |
|-----------|-----|-----------|
| ether1 | DHCP dinamis (Cloud PNETLab) | Akses internet |
| ether2 | 10.0.12.1/30 | Link ke FortiGate Jakarta |
| ether3 | 10.0.13.1/30 | Link ke FortiGate Surabaya |

---

### Surabaya / Branch

#### VLAN Surabaya
| VLAN | Nama | Network | Gateway | Keterangan |
|------|------|---------|---------|------------|
| 30 | SALES | 192.168.30.0/24 | 192.168.30.1 | DHCP dari MikroTik Surabaya |
| 40 | OPERATIONS | 192.168.40.0/24 | 192.168.40.1 | IP Static manual |

#### MikroTik Router Surabaya
| Interface | IP | Keterangan |
|-----------|-----|-----------|
| ether1 | 10.10.200.2/30 | Link ke FortiGate Surabaya |
| vlan30-sales | 192.168.30.1/24 | Gateway VLAN 30 |
| vlan40-operations | 192.168.40.1/24 | Gateway VLAN 40 |

#### FortiGate Surabaya
| Interface | IP | Terhubung ke |
|-----------|-----|-------------|
| port1 | 10.0.13.2/30 | MikroTik ISP |
| port2 | 10.10.200.1/30 | MikroTik Surabaya |
| GRE-SBY-JKT | 172.16.0.2/32 | Tunnel ke FortiGate Jakarta |

#### IP Client Surabaya
| Client | VLAN | IP | Gateway |
|--------|------|----|---------|
| PC Sales (VPCS) | 30 | DHCP (range 192.168.30.100–200) | 192.168.30.1 |
| PC Operations (VPCS) | 40 | 192.168.40.10/24 (static) | 192.168.40.1 |
| Tinycore Linux | 40 | 192.168.40.20/24 (static) | 192.168.40.1 |

---

### GRE Tunnel

| Tunnel | Perangkat | Local WAN | Remote WAN | Tunnel IP |
|--------|-----------|-----------|------------|-----------|
| GRE-JKT-SBY | FortiGate Jakarta | 10.0.12.2 | 10.0.13.2 | 172.16.0.1/32 |
| GRE-SBY-JKT | FortiGate Surabaya | 10.0.13.2 | 10.0.12.2 | 172.16.0.2/32 |

### Network via OSPF

| Site | Network | Keterangan |
|------|---------|------------|
| Jakarta | 192.168.10.0/24 | VLAN 10 Finance |
| Jakarta | 192.168.20.0/24 | VLAN 20 IT |
| Jakarta | 192.168.60.0/24 | VLAN 60 Server |
| Jakarta | 172.16.0.1/32 | GRE Tunnel Jakarta |
| Surabaya | 192.168.30.0/24 | VLAN 30 Sales |
| Surabaya | 192.168.40.0/24 | VLAN 40 Operations |
| Surabaya | 172.16.0.2/32 | GRE Tunnel Surabaya |

---

## Modul 1 — Cisco Switch Jakarta

**Hal yang dikonfigurasi:** VLAN 10 (FINANCE), VLAN 20 (IT), VLAN 60 (SERVER-HQ), access port client, trunk ke Cisco Router dan MikroTik Router.

### Bukti

**1. `show vlan brief`**



<img width="1944" height="1162" alt="Screenshot 2026-06-14 014917" src="https://github.com/user-attachments/assets/2ab2d8bd-3d64-4e30-8313-24786ccf4618" />


**2. `show interfaces trunk`**

<img width="2002" height="1218" alt="Screenshot 2026-06-14 015019" src="https://github.com/user-attachments/assets/4074d255-28a9-4176-9c6b-d9fc2175ec54" />

---

## Modul 2 — Cisco Router Jakarta

**Hal yang dikonfigurasi:** Sub-interface VLAN 10/20/60, VRRP (Master VLAN 10 & 60), DHCP Relay ke Ubuntu Server, link ke FortiGate Jakarta.

### Bukti
**1. Ping dari Cisco Router ke FortiGate Jakarta (`ping 10.10.100.1`)**
<img width="2016" height="1224" alt="Screenshot 2026-06-14 015028" src="https://github.com/user-attachments/assets/29f79062-8c26-47af-b8b7-fc3a49331763" />



---

## Modul 3 — MikroTik Router Jakarta

**Hal yang dikonfigurasi:** VLAN interface 10/20/60, VRRP (Master VLAN 20), DHCP Relay, default route ke FortiGate Jakarta.

### Bukti

**1. `/ip address print dan interface vrrp print`**

<img width="2220" height="1226" alt="Screenshot 2026-06-14 015609" src="https://github.com/user-attachments/assets/aa51c079-8430-4054-9d22-dd362232719c" />



**3. `/ip dhcp-relay print ip route print`**


<img width="2108" height="1164" alt="Screenshot 2026-06-14 015626" src="https://github.com/user-attachments/assets/4b42d639-fe12-4554-be60-c195986aa950" />




**5. Ping dari MikroTik ke FortiGate Jakarta (`ping 10.10.101.1`)**

<img width="2358" height="1232" alt="Screenshot 2026-06-14 015652" src="https://github.com/user-attachments/assets/473ec849-d9c3-4815-a1ac-14c9c0708f66" />

---

## Modul 4 — Ubuntu Server Jakarta

**Hal yang dikonfigurasi:** IP static 192.168.60.10/24, ISC-DHCP Server (VLAN 10 & 20), Nginx Web Server.


### Bukti

**1. `ip a` dan `ip route`**
<img width="2880" height="1250" alt="Screenshot 2026-06-14 020724" src="https://github.com/user-attachments/assets/68315b9f-50b8-4757-a162-2b8e9f115d38" />


**3. Isi file `/etc/dhcp/dhcpd.conf`**

<img width="1302" height="1234" alt="Screenshot 2026-06-14 015707" src="https://github.com/user-attachments/assets/c8f64cfc-ac02-474a-b773-51a54115e54f" />


**4. `ping 8.8.8.8`**

<img width="960" height="1236" alt="Screenshot 2026-06-14 015753" src="https://github.com/user-attachments/assets/9278865b-b585-42b7-83b2-817e4f6aea78" />

---

## Modul 5 — FortiGate Jakarta

**Hal yang dikonfigurasi:** Interface port1/port2/port3, default route, static route internal Jakarta, firewall policy + NAT, GRE Tunnel ke Surabaya, OSPF over GRE, redistribute static.

### Bukti

**1. `get system interface physical`**

<img width="1212" height="1222" alt="Screenshot 2026-06-14 015640" src="https://github.com/user-attachments/assets/5eb1578e-1f92-4457-99cc-2fd9e40d4903" />

**2. `get router info routing-table all`**

<img width="1864" height="1224" alt="Screenshot 2026-06-14 015700" src="https://github.com/user-attachments/assets/36fe327c-245d-4547-bc96-208e96ea85be" />


**3. Screenshot firewall policy**


<img width="1056" height="1224" alt="Screenshot 2026-06-14 015807" src="https://github.com/user-attachments/assets/c060868f-6e96-436d-b37e-9457f26bc9a8" />
<img width="2454" height="1164" alt="Screenshot 2026-06-14 015819" src="https://github.com/user-attachments/assets/a504f8b5-abb2-4e6b-9169-53be1d400546" />

**4. `execute ping 8.8.8.8`,  `execute ping 172.16.0.2`, `get router info ospf neighbor` dan `get router info routing-table ospf`**

<img width="790" height="1224" alt="Screenshot 2026-06-14 015828" src="https://github.com/user-attachments/assets/18d54ca3-1bf6-40c4-be69-2c79ab6fc7d7" />


---

## Modul 6 — MikroTik ISP

**Hal yang dikonfigurasi:** IP ether2 & ether3, DHCP client ether1 (Cloud NAT), default route, NAT masquerade.


### Bukti

**1`/ip address print
/ip route print
/ip firewall nat print
ping 8.8.8.8
ping fortigate Jakarta
ping fortigate Surabaya`**

<img width="908" height="1234" alt="Screenshot 2026-06-14 015841" src="https://github.com/user-attachments/assets/b2cac863-ad6a-4fbe-8feb-41613d53383b" />



---

## Modul 7 — Switch & MikroTik Surabaya

**Hal yang dikonfigurasi:** VLAN 30 (SALES) & VLAN 40 (OPERATIONS) di Switch, trunk ke MikroTik Surabaya, VLAN interface di MikroTik, DHCP Server VLAN 30, IP static VLAN 40.

### Bukti Switch Surabaya

**1. `show vlan brief` dan  `show interfaces trunk`**

<img width="1480" height="1234" alt="Screenshot 2026-06-14 015851" src="https://github.com/user-attachments/assets/c4fc5745-248a-4b4b-8ca5-ed3026dfb0a5" />


### Bukti MikroTik Surabaya

**3. `/ip address print
/ip dhcp-server print
/ip pool print
/ip route print`**

<img width="1048" height="1236" alt="Screenshot 2026-06-14 015914" src="https://github.com/user-attachments/assets/b7fe138a-42a9-4c07-8f05-ecd467c31f6c" />


**7. Client VLAN 30 mendapat IP DHCP dan Ping client Surabaya ke 8.8.8.8**

<img width="1498" height="1262" alt="Screenshot 2026-06-14 015932" src="https://github.com/user-attachments/assets/6eafd58f-6c2e-4017-90bf-1704ce62222b" />


---

## Modul 8 — FortiGate Surabaya

**Hal yang dikonfigurasi:** Interface port1/port2, default route ke ISP, static route VLAN Surabaya, firewall policy + NAT, GRE Tunnel ke Jakarta, OSPF over GRE, redistribute static.

### Bukti

**1. `get system interface physical`**

<img width="1222" height="1226" alt="Screenshot 2026-06-14 020514" src="https://github.com/user-attachments/assets/eddaa106-3ae0-449b-b9bf-a7a1e8e7a75f" />


**2. `get router info routing-table all` dan  Screenshot firewall policy**

<img width="1124" height="1234" alt="Screenshot 2026-06-14 020522" src="https://github.com/user-attachments/assets/b7d471ad-29a1-4f87-a658-fe74b5707a5e" />
<img width="2200" height="1244" alt="Screenshot 2026-06-14 020528" src="https://github.com/user-attachments/assets/50d58c0d-2e01-43b8-a5c6-0fd2e88ce31f" />

**4. `ping 8.8.8.8
ping IP tunnel Jakarta
get router info ospf neighbor
get router info routing-table ospf`**

<img width="1038" height="1246" alt="Screenshot 2026-06-14 020534" src="https://github.com/user-attachments/assets/ec846d39-e769-41be-ad95-ba35b1f5191d" />


---

## Modul 9 — Verifikasi GRE Tunnel & OSPF

**Verifikasi:** Ping WAN antar FortiGate, ping IP tunnel, OSPF neighbor Full, routing table OSPF, ping antar client Jakarta dan Surabaya.

### Bukti FortiGate Jakarta

**1. Ping WAN ke Surabaya & Ping IP Tunnel + OSPF neighbor + routing table OSPF**

<img width="1066" height="1240" alt="Screenshot 2026-06-14 020541" src="https://github.com/user-attachments/assets/ae13e55e-4003-40c1-83dd-9bd848b324cc" />

### Bukti FortiGate Surabaya

**2. Ping WAN ke Jakarta & Ping IP Tunnel + OSPF neighbor + routing table OSPF**

<img width="990" height="1214" alt="Screenshot 2026-06-14 020548" src="https://github.com/user-attachments/assets/8a3098f5-95b2-4e05-a58f-67355b3d4952" />

### Ping Antar Client

**3. Ping dari VLAN10 (Client Jakarta) ke VLAN30 (Client Surabaya)**

<img width="2880" height="1140" alt="Screenshot 2026-06-14 020620" src="https://github.com/user-attachments/assets/8b7e0aee-0d9c-4c8a-b99b-4b0cf2a7529f" />


**4. Ping dari VLAN30 (Client Surabaya) ke VLAN10 (Client Jakarta)**
<img width="2880" height="1010" alt="Screenshot 2026-06-14 020628" src="https://github.com/user-attachments/assets/2007b0e2-8839-4487-895d-a860545d8c7d" />


---

## Modul 10 — Pengujian Akhir

### Bukti

**1. Client VLAN10 Jakarta — IP DHCP dari Ubuntu Server**
<img width="2880" height="1128" alt="Screenshot 2026-06-14 020634" src="https://github.com/user-attachments/assets/d11e421e-6709-4c0c-9ae6-f9de2d32321e" />


**2. Client VLAN20 Jakarta — IP DHCP dari Ubuntu Server**
<img width="2880" height="426" alt="Screenshot 2026-06-14 020640" src="https://github.com/user-attachments/assets/d458ce29-3d87-4715-9eb7-511214e14205" />


**3. Client Jakarta ping internet (8.8.8.8)**

<img width="1996" height="1244" alt="Screenshot 2026-06-14 020647" src="https://github.com/user-attachments/assets/35561c56-02c7-494b-a82d-13cc3845893a" />


**4. Client Surabaya ping internet (8.8.8.8)**
<img width="2030" height="1238" alt="Screenshot 2026-06-14 020654" src="https://github.com/user-attachments/assets/95e58236-3ec9-4702-acb2-9eb1cbdd629f" />



**5. Ping antar site**

<img width="1986" height="1222" alt="Screenshot 2026-06-14 020700" src="https://github.com/user-attachments/assets/3d25d47c-df68-4fb9-b800-ec2804193980" />


**6. Akses Web Server Jakarta dari Tinycore Surabaya (`http://192.168.60.10`)**
<img width="1240" height="1246" alt="Screenshot 2026-06-14 020705" src="https://github.com/user-attachments/assets/47548a24-70ed-47c5-bf22-802dfe0f6efc" />


**7. `get router info routing-table ospf` — FortiGate Jakarta**
<img width="2880" height="1254" alt="Screenshot 2026-06-14 020712" src="https://github.com/user-attachments/assets/19912c9c-be94-4f09-baf9-8dbce957ced5" />


**8. `get router info routing-table ospf` — FortiGate Surabaya**
<img width="2880" height="1212" alt="Screenshot 2026-06-14 020718" src="https://github.com/user-attachments/assets/90971471-aae5-404f-8554-c65962a5435b" />


**9. Analisis Singkat Jalur Traffic Jakarta ke Surabaya**

> "Traffic dari client Jakarta (VLAN10/20/60) diteruskan melalui Cisco Router atau MikroTik Router Jakarta menuju FortiGate Jakarta via link transit (10.10.100.0/30 dan 10.10.101.0/30). FortiGate Jakarta kemudian meneruskan traffic melalui GRE Tunnel (GRE-JKT-SBY) dengan IP tunnel 172.16.0.1 menuju FortiGate Surabaya (172.16.0.2) melewati jaringan WAN ISP (10.0.12.0/30 → 10.0.13.0/30). Route antar-site didistribusikan secara dinamis menggunakan OSPF over GRE. Setelah sampai di FortiGate Surabaya, traffic diteruskan ke MikroTik Surabaya dan akhirnya ke client VLAN30/40."

---

## ✅ Hasil Pengujian Akhir

| Pengujian | Hasil |
|-----------|-------|
| Client VLAN 10 Jakarta mendapat IP DHCP | ✅ |
| Client VLAN 20 Jakarta mendapat IP DHCP | ✅ |
| Client VLAN 30 Surabaya mendapat IP DHCP | ✅ |
| Client VLAN 40 Surabaya menggunakan IP static | ✅ |
| Client Jakarta ping 8.8.8.8 | ✅ |
| Client Surabaya ping 8.8.8.8 | ✅ |
| Client Jakarta ping Client Surabaya (antar site) | ✅ |
| Client Surabaya ping Client Jakarta | ✅ |
| OSPF neighbor antar FortiGate berstatus Full | ✅ |
| Route antar site tersebar via OSPF (O E2) | ✅ |
| Tinycore Surabaya akses `http://192.168.60.10` | ✅ |

---

## 🚦 Jalur Traffic

```
Client Jakarta → Internet
  VLAN 10/20 Client
    → VRRP Gateway (Cisco/MikroTik Jakarta)
    → FortiGate Jakarta (NAT + Firewall)
    → MikroTik ISP
    → Cloud NAT / Internet

Client Surabaya → Web Server Jakarta
  VLAN 30/40 Client
    → MikroTik Surabaya
    → FortiGate Surabaya
    → GRE Tunnel (172.16.0.2 → 172.16.0.1)
    → FortiGate Jakarta
    → Cisco/MikroTik Jakarta
    → Ubuntu Server (192.168.60.10)
```

---


---

> **Laboratorium M-IoT — Computer Engineering**
> Platform: PNETLab | Tools: FortiOS, MikroTik RouterOS, Cisco IOS, Ubuntu Server, Tinycore Linux
