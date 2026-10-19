<div align="center">

# 🚀 HestiaCP di Proxmox + Cloudflare Tunnel + AI Worker (Gemini)

**Mini project dokumentasi:** dari install panel hosting di VM Proxmox, publikasi lewat domain tanpa port forwarding, sampai website dengan chat AI yang aman.

![Proxmox](https://img.shields.io/badge/Proxmox-VE-E57000?logo=proxmox&logoColor=white)
![HestiaCP](https://img.shields.io/badge/HestiaCP-Control%20Panel-D6336C)
![Cloudflare](https://img.shields.io/badge/Cloudflare-Tunnel%20%26%20Workers-F38020?logo=cloudflare&logoColor=white)
![Gemini](https://img.shields.io/badge/Google-Gemini%20API-4285F4?logo=googlegemini&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES%20Module-F7DF1E?logo=javascript&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-green)

🇬🇧 [Read in English](README.en.md)

</div>

---

## 📌 Ringkasan

Repo ini adalah dokumentasi hasil belajar saya (siswa TKJ SMKN 3 Kota Tangerang) saat membangun satu sistem hosting kecil end-to-end:

1. **Proxmox VE** menjalankan sebuah VM Linux (IP `192.168.1.233`).
2. **HestiaCP** dipasang di VM sebagai control panel hosting.
3. **Cloudflare Tunnel** mempublikasikan panel dan website lewat domain, tanpa membuka port di router.
4. Website **Qyv Hermes Agent OS** diunggah ke `public_html` memakai **WinSCP (SFTP)**.
5. Fitur chat AI **Qyv Agent** memanggil **Cloudflare Worker** yang menjadi perantara ke **Gemini API**, sehingga API key tidak pernah muncul di browser.

| Domain | Fungsi |
| --- | --- |
| `qyvv.site` | Website Qyv Hermes Agent OS |
| `panel.qyvv.site` | Panel HestiaCP |
| `api.qyvv.site` | Endpoint `/chat` (Cloudflare Worker `qyvv-proxy`) |

## 🏗️ Arsitektur
```
flowchart LR
    U[Pengunjung] -->|HTTPS| CF[Cloudflare]
    CF -->|qyvv.site| T[Cloudflare Tunnel Hestia Panel]
    CF -->|panel.qyvv.site| T
    T --> VM[VM HestiaCP 192.168.1.233]
    U -->|POST /chat| W[Worker qyvv-proxy api.qyvv.site]
    W -->|API key dari secret| G[Gemini API]
```

Alur satu pesan chat:

1. Pengunjung mengetik pesan di Qyv Agent pada `qyvv.site`.
2. Browser mengirim `POST https://api.qyvv.site/chat` berisi riwayat percakapan.
3. Worker memeriksa origin, memvalidasi pesan, lalu memanggil Gemini dengan API key dari secret.
4. Gemini membalas, Worker meneruskan `{ "reply": "..." }`, browser menampilkannya.

## 🧰 Teknologi

| Komponen | Peran |
| --- | --- |
| Proxmox VE | Virtualisasi, menjalankan VM server |
| HestiaCP | Control panel hosting (domain web, user, file) |
| Cloudflare Zero Trust Tunnel | Publikasi layanan tanpa port forwarding |
| Cloudflare Workers | AI proxy serverless (JavaScript) |
| Gemini API | Model AI penjawab chat (`gemini-3.5-flash-lite`) |
| WinSCP | Unggah file lewat SFTP/SSH |
| Tailwind CSS, Phosphor Icons, Google Fonts | Tampilan website (dimuat dari CDN) |

## 📁 Struktur Repo

```text
.
├── README.md               # Bahasa Indonesia
├── README.en.md            # English
├── LICENSE
├── .gitignore
├── website/
│   └── index.html          # Website Qyv Hermes Agent OS (satu file)
├── worker/
│   └── index.js            # Kode Cloudflare Worker (AI proxy ke Gemini)
└── docs/
    └── screenshots/        # Bukti hasil praktik
```

## 🛠️ Langkah Implementasi

### 1. Menyiapkan VM di Proxmox VE

Buat VM Linux, atur IP lokal statis `192.168.1.233`, dan pastikan VM punya akses internet serta SSH bisa dipakai dari komputer klien.

### 2. Memasang HestiaCP

Masuk ke VM sebagai root lewat SSH, lalu unduh dan jalankan skrip instalasi resmi:

```bash
wget https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh
bash hst-install.sh
```

Setelah selesai, panel dapat dibuka di port `8083`. Akun dipisahkan antara administrator dan pengguna biasa untuk hosting web.

## Dashboard Hestia <img width="1050" height="402" alt="image" src="https://github.com/user-attachments/assets/a4130332-0086-49f5-98ca-5fb4efad9d2d" />


### 3. Cloudflare Zero Trust Tunnel

1. Buka **Zero Trust → Networks → Tunnels**, buat tunnel bernama `Hestia Panel`.
2. Pasang konektor `cloudflared` di VM sesuai perintah dari Cloudflare (token tunnel rahasia, jangan di-commit).
3. Tambahkan dua route *published application* pada tunnel yang sama:

| Hostname | Tujuan (contoh) |
| --- | --- |
| `panel.qyvv.site` | `https://localhost:8083` |
| `qyvv.site` | `http://localhost:80` |

4. Pastikan status tunnel **Healthy**, lalu uji kedua domain di browser.

## Daftar tunnel <img width="1050" height="384" alt="image" src="https://github.com/user-attachments/assets/a20c8b03-049a-4b02-9112-2f78d68fb329" />


## Route tunnel <img width="1050" height="520" alt="image" src="https://github.com/user-attachments/assets/9d010707-b128-4d08-885f-e32ae4b86c9f" />


## Login Hestia lewat domain <img width="1366" height="715" alt="{726E21B5-C40C-48BB-9425-FF393C196842}" src="https://github.com/user-attachments/assets/a64f5118-77e4-4574-9a48-c89df99dbc05" />


### 4. Domain web dan unggah file lewat WinSCP

1. Di Hestia, buka menu **Web** dan tambahkan domain `qyvv.site`.
2. Di WinSCP buat koneksi **SFTP** ke `192.168.1.233` (port 22).
3. Buka direktori `public_html` di panel kanan, lalu seret file dari `website/` ke sana.
4. Buka `https://qyvv.site` untuk memastikan website tampil.

## WinSCP <img width="834" height="266" alt="image" src="https://github.com/user-attachments/assets/aa16b4d2-e0ed-4c53-aa29-8b29d330f34f" />


## Website <img width="1050" height="564" alt="image" src="https://github.com/user-attachments/assets/03782b38-b5de-4522-8331-a8b85b927f22" />


### 5. Cloudflare Worker sebagai AI proxy

1. Di Cloudflare buka **Workers & Pages**, buat Worker bernama `qyvv-proxy`.
2. Tempel isi [`worker/index.js`](worker/index.js), lalu deploy.
3. Tambahkan custom domain `api.qyvv.site`.
4. Isi variabel dan secret berikut:

| Nama | Tipe | Fungsi |
| --- | --- | --- |
| `ALLOWED_ORIGIN` | Variable | Daftar origin yang boleh memanggil Worker, dipisah koma (mis. `https://qyvv.site`) |
| `GEMINI_API_KEY` | Secret | Kunci Gemini API, tidak pernah dikirim ke browser |
| `GEMINI_MODEL` | Variable | Nama model Gemini (di sini `gemini-3.5-flash-lite`) |
| `RATE_LIMITER` | Binding (opsional) | Batas laju per IP; dilewati bila belum dipasang |

## Overview Worker <img width="1050" height="616" alt="image" src="https://github.com/user-attachments/assets/c308203f-9b4d-400c-a71d-62e16bfe30cd" />


## Variabel Worker <img width="1050" height="445" alt="image" src="https://github.com/user-attachments/assets/cd449406-7016-46ca-bcd7-d422c1e88ade" />


### 6. Pengujian

Kirim pesan uji lewat Qyv Agent. Indikator Worker berwarna hijau dan AI menjawab sesuai persona.

## Chat Qyv Agent <img width="1366" height="729" alt="{63228ED9-554D-4D66-ADE8-276F8450AA2A}" src="https://github.com/user-attachments/assets/bd95f085-2f97-4a80-be47-cee5a2370b67" />

## 🔌 API Worker

`POST https://api.qyvv.site/chat`

```json
{ "messages": [ { "role": "user", "content": "tes" } ] }
```

| Kondisi | HTTP | Respons |
| --- | --- | --- |
| Berhasil | 200 | `{ "reply": "..." }` |
| Bukan `POST /chat` | 404 | `not_found` |
| Origin tidak diizinkan | 403 | `origin_blocked` |
| Terlalu banyak permintaan | 429 | `rate_limited` + `retryAfter` |
| Body atau pesan tidak valid | 400 | `bad_request` |
| API key atau model belum diisi | 500 | `server_misconfigured` |
| Gemini lewat 20 detik | 504 | `upstream_timeout` |
| Gemini gagal atau balasan kosong | 502 | `upstream_error` |

Batas yang dipakai Worker: maksimal **10 pesan** per permintaan, **500 karakter** per pesan, **512 token** keluaran, waktu tunggu **20 detik**, suhu **0,9**. Role `system` dari klien ditolak.

Di sisi website, riwayat dipangkas menjadi 9 pesan terakhir dan 450 karakter per pesan agar selalu di bawah batas Worker. Jika Worker tidak dapat dihubungi, Qyv Agent masuk mode offline dan menjawab dari materi bawaan.

## 🔐 Catatan Keamanan

- API key Gemini hanya disimpan sebagai **secret** di Worker, bukan di file HTML.
- Worker menolak origin yang tidak ada di `ALLOWED_ORIGIN`. Header Origin bisa dipalsukan klien non-browser, jadi ini perlu dilengkapi batas laju.
- Di deployment ini binding `RATE_LIMITER` belum dipasang, jadi batas laju per IP belum aktif.
- `ALLOWED_ORIGIN` di deployment ini juga memuat `null` (origin dari file lokal atau iframe sandbox). Hapus bila tidak diperlukan.
- Panel Hestia terbuka di internet lewat `panel.qyvv.site`. Disarankan mengaktifkan 2FA di Hestia dan mempertimbangkan Cloudflare Access.
- Jangan commit token tunnel, API key, atau password. Sensor email, tunnel ID, dan IP pada screenshot sebelum dipublikasikan.

## 📚 Masalah dan Pelajaran

| Masalah | Solusi / pelajaran |
| --- | --- |
| Unggah lewat browser terasa lag | Pindah ke WinSCP (SFTP lewat SSH) |
| Indikator Qyv Agent sempat menampilkan `worker 502` | Kode 502 berarti `upstream_error`: panggilan ke Gemini gagal atau balasan kosong; pada pengujian berikutnya sudah normal |
| Teks `let _lastAct=Date.now();` tampil di luar halaman | Baris itu berada setelah `</html>`; sudah dihapus pada `website/index.html` (variabelnya sudah dideklarasikan di skrip utama) |

Pelajaran utama: satu tunnel dapat melayani beberapa hostname, rahasia harus berada di sisi server (Worker), dan pembatasan perlu dipasang di dua sisi (website dan Worker).

## 🗺️ Rencana Perbaikan

- [ ] Pasang binding rate limiting (`RATE_LIMITER`) pada Worker
- [ ] Hapus `null` dari `ALLOWED_ORIGIN` bila tidak diperlukan
- [ ] Finalisasi `SYSTEM_PROMPT` (hapus baris placeholder)
- [ ] Aktifkan 2FA Hestia dan pertimbangkan Cloudflare Access untuk panel

## 📄 Lisensi

Dirilis di bawah lisensi [MIT](LICENSE).

## 👤 Penulis

* **Nama:** [Rizqi Maulana Yusuf]
* **GitHub:** [@rizqi-yusuf](https://github.com/rizqi-yusuf)
