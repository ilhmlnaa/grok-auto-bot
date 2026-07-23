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
| 🔗 9Router OAuth | Device code flow, auto-add akun ke dashboard |
| 👥 Anti-duplicate | Skip akun yang sudah ada di 9router |
| 📊 Dashboard UI | Progress bar, live stats, system logs |
| ⚡ Fast | ~50 detik per akun |

---

## 📦 Prasyarat

- **Python 3.11+**
- **Cloakbrowser** (auto-install Chromium binary)
- **Xvfb** (virtual display — Linux/WSL wajib)
- **2captcha API key** (top-up minimal $3)

---

## 🔧 Install

```bash
git clone https://github.com/ilhmlnaa/grok-auto-bot.git
cd grok-auto-bot

# Install deps (Ubuntu/Debian)
apt install -y xvfb python3-pip
pip3 install cloakbrowser playwright curl_cffi --break-system-packages

# Cloakbrowser setup
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

# 9Router
ROUTER9_URL=https://router.hamdiv.me
ROUTER9_PASS=your_password
```

---

## 🎯 Penggunaan

### Buat Akun Baru

```bash
# 1 akun (prompt Y/N untuk add ke 9router)
xvfb-run -a python3 grok-auto-bot.py 1

# 5 akun + AUTO add ke 9router (tanpa prompt)
xvfb-run -a python3 grok-auto-bot.py 5 --auto
```

### Import Akun Existing ke 9Router

```bash
# Tambah semua dari grok_sso.txt
xvfb-run -a python3 grok-auto-bot.py --router

# Tambah 3 akun terakhir
xvfb-run -a python3 grok-auto-bot.py --router 3
```

---

## 📁 Output

`grok_sso.txt` — JSON lines:

```json
{"email":"xxx@zenime.online","password":"...","code":"ABC123","sso_cookies":[...],"timestamp":1720000000}
```

---

## 🔄 Flow

```
1. Buka accounts.x.ai/sign-up
2. Create temp email (MAILLDEZ)
3. Isi email → tunggu OTP
4. Submit OTP → form nama & password
5. Intercept turnstile.render() → 2captcha solve
6. Callback → Complete sign up
7. Redirect grok.com → save SSO cookies
8. [--auto] Add ke 9router via OAuth device flow
```

---

## 📝 Catatan

- Cloakbrowser harus di-install terpisah (`pip3 install cloakbrowser`)
- 2captcha API key hardcoded — edit di script jika perlu
- Anti-duplicate 9router: cek existing providers, skip jika sudah ada
- Kalau polling timeout tapi consent sudah OK, tetap dihitung sukses
