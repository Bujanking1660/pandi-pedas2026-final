# pandi-pedas2026-final

Analisis data DNS traffic authoritative nameserver ccTLD `.id` (PANDI) — dataset `dns.parquet`.

---

## 📁 Dataset

| Parameter | Nilai |
|---|---|
| File | `data/parquet/dns.parquet` |
| Total record | **11.712.623** |
| Rentang waktu | 2026-08-19 07:49:57 — 08:19:57 UTC **(30 menit)** |
| Jumlah kolom | 34 field |

**Kolom utama:** `ts`, `ts_iso`, `ip_ver`, `proto`, `src_ip`, `src_port`, `dst_ip`, `dst_port`, `frame_len`, `dns_len`, `dns_id`, `qr`, `opcode`, `aa`, `tc`, `rd`, `ra`, `ad`, `cd`, `rcode`, `qdcount`, `ancount`, `nscount`, `arcount`, `qname`, `qtype`, `qtype_name`, `qclass`, `edns`, `edns_udpsize`, `do`, `ecs`, `ecs_scope`, `answers`

---

## 🔬 Unit Analisis & Denominator

- **Unit analisis:** Satu record = satu pesan DNS (query **atau** response) pada wire level
- **Denominator total:** 11.712.623 record
- **Denominator query:** 5.861.701 (field `qr = 0`)
- **Denominator response:** 5.850.922 (field `qr = 1`)
- RCODE dianalisis **hanya pada response**; QTYPE dianalisis **hanya pada query**

---

## 📊 Temuan Utama

### 1. Volume & Keseimbangan Traffic

| Metrik | Nilai |
|---|---|
| Total paket/30 menit | 11.712.623 |
| Rata-rata throughput | ~390.000 paket/menit (~6.500 pkt/detik) |
| Query | 5.861.701 (50,0%) |
| Response | 5.850.922 (50,0%) |

> Rasio query:response yang sangat seimbang (50:50) menunjukkan infrastruktur authoritative DNS dalam kondisi sehat.

---

### 2. Protokol Transport

| Protokol | Jumlah | Persentase |
|---|---|---|
| UDP | 11.202.232 | **95,64%** |
| TCP | 510.391 | **4,36%** |

TCP 4,4% tergolong **lebih tinggi dari normal**. Dikonfirmasi oleh Truncated (TC) bit = **2,75%** (321.798 paket) yang memaksa retry via TCP — disebabkan oleh respons DNSSEC berukuran besar.

---

### 3. IPv4 vs IPv6

| Versi | Jumlah | Persentase |
|---|---|---|
| IPv6 | 8.482.271 | **72,4%** |
| IPv4 | 3.230.352 | **27,6%** |

Dominasi IPv6 konsisten dengan infrastruktur authoritative/resolver dual-stack modern.

---

### 4. Response Code (RCODE)
> Denominator: **5.850.922 response**

| RCODE | Nama | Jumlah | Persentase |
|---|---|---|---|
| 0 | NOERROR | 5.173.188 | **88,42%** |
| 3 | **NXDOMAIN** | **675.229** | **11,54%** ⚠️ |
| 1 | FORMERR | 2.155 | 0,037% |
| 4 | NOTIMP | 294 | 0,005% |
| 5 | REFUSED | 37 | 0,0006% |
| 2 | SERVFAIL | 18 | 0,0003% |

> **NXDOMAIN 11,54% jauh di atas threshold normal (< 5%)** — mengindikasikan banyak query ke domain tidak terdaftar, termasuk kemungkinan abuse aktif.

---

### 5. Tipe Query (QTYPE)
> Denominator: **5.857.095 query valid** (5.861.701 − 4.606 null)

| QTYPE | Jumlah | Persentase |
|---|---|---|
| A (IPv4 address) | 2.762.209 | **47,1%** |
| NS (Name Server) | 1.852.410 | **31,6%** |
| AAAA (IPv6 address) | 505.576 | 8,6% |
| DS (DNSSEC delegation) | 494.324 | 8,4% |
| TXT | 59.446 | 1,01% |
| MX | 58.598 | 1,00% |
| HTTPS | 57.854 | 0,99% |
| CNAME | 22.774 | 0,39% |
| DNSKEY | 15.290 | 0,26% |
| SOA | 13.981 | 0,24% |

> NS query 31,6% sangat tinggi — mengonfirmasi server ini adalah **authoritative nameserver** (bukan recursive resolver), banyak menerima delegasi dari resolver lain.

---

### 6. DNSSEC & EDNS Adoption

| Metrik | Jumlah | Persentase |
|---|---|---|
| EDNS aktif | 11.289.620 | **96,4%** |
| DO (DNSSEC OK) bit = 1 | 10.258.247 | **87,6%** |
| DS queries | 494.324 | 8,4% dari query |

> Adopsi DNSSEC sangat tinggi — konsisten dengan pengelolaan ccTLD `.id` yang sudah signed penuh.

---

### 7. Top 20 Domain Paling Banyak Dikueri

| Rank | Domain | Query |
|---|---|---|
| 1 | `co.id.` | 17.179 |
| 2 | `dns2.anymhost.id.` | 7.936 |
| 3 | `dns1.anymhost.id.` | 7.842 |
| 4 | `ns01.cloudhost.id.` | 7.095 |
| 5 | `ns02.cloudhost.id.` | 6.788 |
| 6 | `net.id.` | 6.437 |
| 7 | `f.dns.id.` | 5.559 |
| 8 | `telkom.net.id.` | 4.604 |
| 9 | `go.id.` | 4.219 |
| 10 | `web.id.` | 4.207 |
| 11 | `shopee.co.id.` | 4.004 |
| 12 | `my.id.` | 3.940 |
| 13 | `www.google.co.id.` | 3.836 |
| 14 | `bankmandiri.co.id.` | 3.293 |

---

### 8. Top NXDOMAIN — Domain Tidak Ditemukan Terbanyak ⚠️

| Domain | NXDOMAIN |
|---|---|
| `tracker.itscraftsoftware.my.id.` | 2.574 |
| `ns1.bna.net.id.` | 2.514 |
| `ns2.bna.net.id.` | 2.509 |
| `msoid.CO.ID.` | 1.771 |
| `msoid.ymidn.co.id.` | 1.660 |
| `bna.net.id.` | 1.474 |
| `msoid.sch.id.` | 1.338 |
| `msoid.go.id.` | 1.230 |
| `dns1.bip.net.id.` | 1.201 |
| `msoid.ac.id.` | 1.200 |
| `dns2.bip.net.id.` | 1.199 |
| `6441056b613c32a9.co.id.` | 1.183 |
| `msoid.hirose.co.id.` | 1.109 |
| `msoid.indonesia.hirose.co.id.` | 1.075 |
| `_.co.id.` | 563 |

> **Pola `msoid.*`** muncul di 6 SLD berbeda (`CO.ID`, `sch.id`, `go.id`, `ac.id`, `hirose.co.id`, `ymidn.co.id`) dalam 30 menit — indikator **subdomain scanning/probing sistematis**.
>
> **`6441056b613c32a9.co.id.`** — label berupa hash hex 16-karakter adalah **indikator kuat DGA (Domain Generation Algorithm) atau DNS tunneling**.

---

### 9. Source IP — Klaster Traffic Tertinggi

| Source IP | Query/30 menit |
|---|---|
| `2d68:a529:78c2:4e21:b880:7ac9:3ec:262d` | 31.899 |
| `2d68:a529:78c2:4e00:f4c3:5dc9:8696:d871` | 29.271 |
| `2d68:a529:78c2:4e00:f4c3:5dc9:8696:d87a` | 27.473 |
| `2d68:a529:78c2:4e1d:ab3c:2d41:dba1:76c7` | 26.935 |
| `145.164.38.161` (IPv4) | 24.583 |

Top 19 src IP berasal dari prefix IPv6 `/48` yang sama (`2d68:a529:78c2::/48`) — kemungkinan **anycast resolver besar atau ISP tunggal** dengan load balancing per IP.

---

### 10. Destination IP (Server yang Dianalisis)

| Destination IP | Query Masuk |
|---|---|
| `2d6e:a11:6217:18f6:8f03:a779:5079:a07b` | 4.241.932 (72,4%) |
| `52.165.111.55` | 1.619.031 (27,6%) |

Server utama (`2d6e:a11:...`) menerima mayoritas traffic, `52.165.111.55` sebagai node sekunder/anycast.

---

### 11. Statistik Frame & Answer

| Metrik | Nilai |
|---|---|
| Frame length rata-rata | 343,1 byte |
| Frame length median | 136 byte |
| Frame length maksimum | 2.479 byte |
| Response dengan 0 answers | 5.785.717 (98,9% dari response) |
| Authoritative Answer (AA bit) | 1.240.539 (10,6%) |
| Unique source IP | 88.371 |
| ECS usage | 95.782 (0,82%) |

---

## 🎯 Insight & Rekomendasi

| # | Masalah | Bukti Angka | Rekomendasi |
|---|---|---|---|
| 1 | **NXDOMAIN rate terlalu tinggi** | 675.229 / 5.850.922 = 11,54% (normal < 5%) | Audit zona `.id`, identifikasi NS tidak terdelegasikan. Pertimbangkan negative TTL yang lebih panjang |
| 2 | **Subdomain scanning `msoid.*`** | 1.075–1.771 NXDOMAIN per varian dalam 30 menit, lintas 6 SLD | Rate-limit query `msoid.*` di ACL server; eskalasi ke CERT-ID |
| 3 | **Domain DGA/tunneling** | `6441056b613c32a9.co.id.` = 1.183 NXDOMAIN, label hash hex | Tambahkan deteksi berbasis entropi label di IDS DNS |
| 4 | **TCP 4,4% & Truncation 2,75%** | 321.798 TC-flagged paket memaksa TCP retry | Advertise EDNS buffer ≥ 4096 byte; periksa MTU path antara resolver & authoritative |
| 5 | **Klaster IPv6 `2d68:a529:...`** | 19 IP dari /48 subnet, masing-masing 24k–32k query/30 menit | Verifikasi legitimasi; terapkan rate limiting per /48 prefix |
| 6 | **SERVFAIL sangat rendah** | 18 kasus = 0,0003% | Positif — zona DNSSEC sehat. Pertahankan monitoring signing key |
| 7 | **`bna.net.id.` & NS-nya tidak resolve** | `ns1/ns2.bna.net.id.` = 2.514 / 2.509 NXDOMAIN | Koordinasikan dengan registrar untuk perbaikan NS record atau hapus delegasi |

---

## 🔑 Kesimpulan

Data ini merupakan **capture 30 menit traffic authoritative nameserver ccTLD `.id`** dengan throughput **~6.500 paket/detik**. Infrastruktur secara umum **sehat**: NOERROR 88,4%, SERVFAIL sangat rendah (0,0003%), adopsi DNSSEC tinggi (87,6%).

**Masalah utama** adalah NXDOMAIN rate 11,54% yang jauh melampaui batas normal, didorong oleh:
1. Delegasi NS yang belum beres (contoh: `bna.net.id`)
2. **Potensi abuse aktif** — pola `msoid.*` lintas SLD (scanning) dan domain berbasis hash hex (indikator DGA/tunneling)

Tindakan prioritas: audit delegasi zona `.id`, rate-limit pattern mencurigakan, dan koordinasi dengan CERT-ID untuk investigasi lanjutan.

---

## ⚙️ Cara Replikasi Analisis

```bash
pip install pandas pyarrow numpy
python analyze_dns2.py
```

Script: [`analyze_dns2.py`](analyze_dns2.py)
