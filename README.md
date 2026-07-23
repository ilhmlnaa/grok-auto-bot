# 🚀 Grok Auto-Bot

> Auto-register akun Grok (x.ai) via **Cloakbrowser + 2captcha callback** + **9Router OAuth**

![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python)
![Cloakbrowser](https://img.shields.io/badge/Cloakbrowser-stealth-purple)
![2Captcha](https://img.shields.io/badge/Turnstile-2Captcha-orange)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## ✨ Fitur

| Fitur | Keterangan |
|---|---|
| 🤖 Auto-register Grok | Bot headless via Cloakbrowser (stealth Chromium) |
| 🛡️ Turnstile bypass | **2captcha API** — intercept `turnstile.render()`, callback |
| 📧 Temp mail | MAILLDEZ-compatible API (OTP auto-detect) |
| 📝 Custom email prefix | Load nama dari file `.txt`, sukses → auto terhapus |
| 🔗 9Router OAuth | Device code flow, auto-add akun ke dashboard |
| 👥 Anti-duplicate | Skip akun yang sudah ada di 9router |
| 🔄 Auto-retry | Gagal → retry 2x otomatis |
| ⏱️ Cooldown | 8-15s random antar akun (anti rate-limit) |
| 📊 Dashboard UI | Progress bar, live stats, system logs |
| 📺 Screen support | `screen -S grok` + detach, terminal aman ditutup |
| ⚡ Fast | ~50 detik per akun |

---

## 📦 Prasyarat

- **Python 3.11+**
- **Cloakbrowser** (`pip3 install cloakbrowser`)
- **Xvfb** (virtual display — Linux/WSL wajib)
- **2captcha API key** (top-up minimal $3)

---

## 🔧 Install (VPS Ubuntu)

```bash
git clone https://github.com/ilhmlnaa/grok-auto-bot.git
cd grok-auto-bot

# System deps
apt install -y xvfb python3-pip

# Python deps
pip3 install cloakbrowser playwright curl_cffi --break-system-packages

# Cloakbrowser fetch Chromium binary
cloakbrowser fetch
playwright install chromium
```

---

## ⚙️ Konfigurasi

Copy `.env.example` → `.env`:

```ini
# Temp Mail API (MAILLDEZ-compatible)
MAILLDEZ_URL=https://tempmail-worker.ilhmlnaa.workers.dev
MAILLDEZ_DOMAINS=zenime.online,devnet.my.id

# Password akun Grok (min 16 char)
PASSWORD=GrokAuto123!

# 9Router (opsional — untuk auto-add)
ROUTER9_URL=https://router.hamdiv.me
ROUTER9_PASS=your_password
```

---

## 🎯 Penggunaan

### Screen (biar aman terminal ditutup)

```bash
screen -S grok          # bikin session baru
screen -r grok          # reconnect ke session
screen -ls              # lihat semua session
# Ctrl+A lalu D         # detach (keluar tanpa stop)
```

### Buat Akun Baru

```bash
cd /root/Auto-sign-up-grok-dezz

# 1 akun (prompt y/N untuk add ke 9router)
xvfb-run -a python3 grok-auto-bot.py 1

# 5 akun + AUTO add ke 9router (tanpa prompt)
xvfb-run -a python3 grok-auto-bot.py 5 --auto

# 20 akun + screen (biar aman)
screen -S grok
xvfb-run -a python3 grok-auto-bot.py 20 --auto
# Ctrl+A D → detach, terminal bisa ditutup
```

### Import Akun Existing ke 9Router

```bash
# Tambah semua dari grok_sso.txt
xvfb-run -a python3 grok-auto-bot.py --router

# Tambah 3 akun terakhir aja
xvfb-run -a python3 grok-auto-bot.py --router 3
```

### Custom Email Prefix (dari file)

Bikin file `custom_names.txt` (1 nama per baris):

```txt
chitose
minami
sakura
kaito
yuki
```

Script otomatis pakai nama dari file:

```bash
xvfb-run -a python3 grok-auto-bot.py 5 --auto
```

Output:
```
  → custom names: 5 tersedia
  → using custom: chitose      → chitose@zenime.online
  → using custom: minami       → minami@devnet.my.id
  → using custom: sakura       → sakura@zenime.online
  → custom names habis, fallback → random
```

**Setelah sukses:** nama otomatis kehapus dari `custom_names.txt`. Jadi kalau script dijalankan lagi, gak bakal duplicate.

### Custom Output File

```bash
OUTPUT_FILE=batch_1.txt xvfb-run -a python3 grok-auto-bot.py 10 --auto
```

### Custom Names File

```bash
NAMES_FILE=my_names.txt xvfb-run -a python3 grok-auto-bot.py 4 --auto
```

---

## 📁 File yang Dibutuhkan

| File | Keterangan | Wajib? |
|---|---|---|
| `grok-auto-bot.py` | Script utama | ✅ |
| `.env` | Konfigurasi API key, domain, password | ✅ |
| `custom_names.txt` | Nama custom email (1/baris) | ❌ opsional |

| Output | Isi |
|---|---|
| `grok_sso.txt` | JSON lines — email, password, SSO cookies |

---

## 🔄 Flow

```
1. Buka accounts.x.ai/sign-up
2. Create temp email (MAILLDEZ, domain round-robin)
3. Isi email → tunggu OTP (0.15s polling)
4. Submit OTP → form nama & password
5. Intercept turnstile.render() → 2captcha solve (3s polling)
6. Callback → klik Complete sign up
7. Redirect grok.com → save SSO cookies ke grok_sso.txt
8. [--auto] Add ke 9router via OAuth device flow (anti-duplicate)
```

### Domain Rotation

Domain di-rotate round-robin dari `MAILLDEZ_DOMAINS`:
```
akun-1 → zenime.online
akun-2 → devnet.my.id
akun-3 → zenime.online
akun-4 → devnet.my.id
...
```

---

## 🚨 Troubleshooting

| Error | Fix |
|---|---|
| `FAILED 13s` di step 2 | Rate-limit x.ai — script auto-retry 2x + cooldown 8-15s |
| `OTP timeout 120s` | Domain belum dikonfigurasi di Cloudflare Email Routing |
| `consent gagal` | Script sudah punya fuzzy match fallback untuk tombol |
| `poll timeout` | Normal kalau consent udah OK — dihitung sukses |
| `no consent. buttons:` | "Allow All" punya spasi tersembunyi — script coba exact + fuzzy |

---

## 📝 Catatan

- 2captcha API key hardcoded di script (line `API_KEY = '...'`) — edit kalau perlu
- Anti-duplicate 9router: cek existing providers, skip jika email sudah ada
- Domain round-robin otomatis dari `MAILLDEZ_DOMAINS`
- Custom names habis → fallback ke random 12 char
- Cooldown 8-15s random antar akun biar gak kena rate-limit
