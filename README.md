# Quantus Network — راهنمای کامل راه‌اندازی نود و ماینر

<div dir="rtl">

راهنمای فارسی و گام‌به‌گام اجرای نود مین‌نت **Quantus** و ماینینگ کوین **QTC** روی ویندوز (WSL2) و لینوکس.

منبع: [آموزش رسمی 0xmoei](https://github.com/0xmoei/quantus/) • [مستندات رسمی](https://docs.quantus.com)

</div>

---

## فهرست مطالب

1. [Quantus چیست؟](#1-quantus-چیست)
2. [پیش‌نیازها](#2-پیشنیازها)
3. [ساخت والت و آدرس Wormhole](#3-ساخت-والت-و-آدرس-wormhole)
4. [روش ۱: اسکریپت رسمی (پیشنهادی)](#4-روش-۱-اسکریپت-رسمی-پیشنهادی)
5. [روش ۲: نصب دستی روی لینوکس](#5-روش-۲-نصب-دستی-روی-لینوکس)
6. [روش ۳: ماینینگ پولی با Quanpool](#6-روش-۳-ماینینگ-پولی-با-quanpool)
7. [راه‌اندازی روی ویندوز با WSL2](#7-راهاندازی-روی-ویندوز-با-wsl2)
8. [مانیتورینگ](#8-مانیتورینگ)
9. [آپدیت نود و ماینر](#9-آپدیت-نود-و-ماینر)
10. [نکات امنیتی](#10-نکات-امنیتی)

---

## 1. Quantus چیست؟

Quantus یک بلاکچین **اثبات کار پساکوانتومی (QPoW)** است که از هش **Poseidon2** استفاده می‌کند. پاداش ماینینگ به‌جای والت معمولی به یک **آدرس Wormhole** ارسال می‌شود.

| مشخصه | مقدار |
|---|---|
| نماد | **QTC** |
| حداکثر عرضه | 21,000,000 |
| اعشار | 12 رقم |
| پیشوند آدرس | `qz...` (SS58 prefix: 189) |
| زمان بلوک | ~12 ثانیه |
| نسخه نود | v1.0.1 |
| نسخه ماینر | v4.1.0 |
| شبکه | `--chain mainnet` |
| پروتکل ماینر | `quantus-miner/2` (احراز هویت + TLS pin الزامی) |

> ⚠️ قبل از نصب، حتماً صفحه ریلیزهای نود و ماینر را جداگانه چک کنید — ممکن است نسخه‌های جدیدتر منتشر شده باشند.

---

## 2. پیش‌نیازها

### سخت‌افزار

| منبع | حداقل | پیشنهادی |
|---|---|---|
| CPU | 2 هسته | 4+ هسته |
| RAM | 4 GB | 8+ GB |
| دیسک | 100 GB SSD | 500+ GB SSD |
| اینترنت | 3 Mbps | 10+ Mbps |

### سیستم‌عامل

- Ubuntu 20.04+
- macOS 10.15+
- Windows 10/11 (WSL2)

### نکات مهم

- باینری رسمی ماینر برای **Linux ARM64 وجود ندارد**.
- **GPU** به‌شدت توصیه می‌شود: Metal / Vulkan / DirectX یا NVIDIA CUDA (`--cuda-gpu`).
- ماینینگ با نود داخلی روی CPU فقط **~15 MH/s به‌ازای هر thread** می‌دهد — از ماینر خارجی استفاده کنید.

---

## 3. ساخت والت و آدرس Wormhole

1. والت رسمی را دانلود کنید: [linktr.ee/quantusnetwork](https://linktr.ee/quantusnetwork)
2. والت جدید بسازید یا عبارت ۲۴ کلمه‌ای موجود را بازیابی کنید.
3. برای ساخت آدرس Wormhole **همان عبارت ۲۴ کلمه‌ای** را استفاده کنید تا پاداش‌ها داخل اپ نمایش داده شوند.
4. خروجی‌های زیر را ذخیره کنید:
   - **Address** (با پیشوند `qz...`)
   - **Inner Hash** (برای فلگ `--rewards-inner-hash`)
   - ۲۴ کلمه بازیابی (بک‌آپ آفلاین)

> 🔴 عبارت ۲۴ کلمه‌ای را هرگز با کسی به اشتراک نگذارید.

---

## 4. روش ۱: اسکریپت رسمی (پیشنهادی)

ساده‌ترین راه برای سولو ماینینگ با نود اختصاصی.

### نصب

```bash
curl -fsSL https://docs.quantus.com/scripts/quantus-mining.sh -o quantus-mining.sh
chmod +x quantus-mining.sh
./quantus-mining.sh setup
```

### اجرا و توقف

```bash
./quantus-mining.sh start -d      # نود + ماینر در بک‌گراند
./quantus-mining.sh start         # اجرا با screen
./quantus-mining.sh start-node    # فقط نود
./quantus-mining.sh start-miner   # فقط ماینر
./quantus-mining.sh stop          # توقف همه
```

### لاگ‌ها

```bash
tail -f ~/quantus-mining/logs/node.log
tail -f ~/quantus-mining/logs/miner.log
```

### کانفیگ

```bash
./quantus-mining.sh config show
./quantus-mining.sh config set NODE_NAME "MyNode"
./quantus-mining.sh config set CPU_WORKERS 4
./quantus-mining.sh config set GPU_DEVICES 1
```

> تغییر نسخه نیازمند `setup --force` است — INNER_HASH و آدرس شما حفظ می‌شوند.

---

## 5. روش ۲: نصب دستی روی لینوکس

### 5.1 آماده‌سازی سیستم و فایروال

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl wget tar unzip jq screen ufw ca-certificates -y

sudo ufw allow 22
sudo ufw allow ssh
sudo ufw allow 30333/tcp    # فقط پورت P2P عمومی باشد
sudo ufw enable
```

### 5.2 دانلود باینری‌ها

از صفحه ریلیزهای رسمی، تاربال نود (v1.0.1) و باینری ماینر (v4.1.0) را برای **linux x86_64** دانلود کنید:

```bash
mkdir -p ~/quantus && cd ~/quantus
# فایل‌ها را دانلود و اکسترکت کنید، سپس:
chmod +x quantus-node quantus-miner

# صحت فلگ‌ها را چک کنید:
./quantus-node --help | grep miner-listen-port
./quantus-miner --help | grep auth-token-file
```

### 5.3 ساخت کلید P2P نود

```bash
./quantus-node key generate-node-key --file node_key.p2p
```

### 5.4 ساخت کلیدهای Wormhole

```bash
./quantus-node key quantus --scheme wormhole --words
```

ورودی مخفی است (عبارت در shell history ذخیره نمی‌شود). سه خروجی را ذخیره کنید: **Address** ، **Inner Hash** ، ۲۴ کلمه.

### 5.5 اجرای نود

```bash
screen -S quantus-node

./quantus-node \
  --name YOUR_NODE_NAME \
  --validator \
  --miner-listen-port 9833 \
  --chain mainnet \
  --node-key-file node_key.p2p \
  --rewards-inner-hash YOUR_INNER_HASH \
  --max-blocks-per-request 64 \
  --sync full
```

خروج از screen: `Ctrl+A` بعد `D` • بازگشت: `screen -r quantus-node`

### 5.6 صبر برای سینک کامل

سینک از چند دقیقه تا چند ساعت طول می‌کشد. وقتی وضعیت از **Syncing** به **Idle** رسید آماده‌اید.

> ⚠️ ماینینگ قبل از رسیدن به سر زنجیره = بلوک‌های یتیم (orphan) و از دست رفتن پاداش.

### 5.7 اجرای ماینر

فایل‌های توکن و TLS pin را از مسیر دیتای نود بردارید:

```
~/.local/share/quantus-node/chains/mainnet/
```

(مسیر در macOS متفاوت است)

```bash
screen -S quantus-miner

./quantus-miner \
  --node-addr 127.0.0.1:9833 \
  --auth-token-file /path/to/auth_token \
  --tls-cert-sha256-file /path/to/tls_sha256 \
  --cpu-workers 4 \
  --gpu-devices 1
```

نمونه‌های دیگر:

| سناریو | فلگ‌ها |
|---|---|
| GPU + CPU | `--cpu-workers 4 --gpu-devices 1` |
| فقط CUDA | `--cuda-gpu --gpu-devices 1 --cpu-workers 0` |

> 🔴 توکن را **هرگز inline** پاس ندهید — همیشه از `--auth-token-file` استفاده کنید.
> توکن یا TLS pin اشتباه = **خطای دائمی**.

---

## 6. روش ۳: ماینینگ پولی با Quanpool

بدون نیاز به نود — فقط یک آدرس پرداخت `qz...` می‌خواهد.

| مشخصه | مقدار |
|---|---|
| ثبت‌نام | لازم ندارد |
| کارمزد | 1% پول + 5% ماینر = 6% |
| روش پرداخت | PPLNS یا Solo |
| حداقل پرداخت | 0.25 QTC |
| بازه پرداخت | ~1 ساعت |
| تاییدیه | 105 بلاک |

### راه‌اندازی

1. ماینر را دانلود کنید: `quanpool-miner-6.0.0-linux-x86_64` از `download.quanpool.com`
2. به [quanpool.com](https://quanpool.com) بروید → تب **Start mining** → دستور را **همان لحظه از سایت کپی کنید** (IP پول و TLS pin ممکن است تغییر کنند).

شکل کلی دستور:

```bash
screen -S quanpool

./quanpool-miner-6.0.0-linux-x86_64 \
  --node-addr <pool-ip>:9834 \
  --auth-token qzYOUR_ADDRESS.worker \
  --tls-cert-sha256 <pin>
```

- مانیتورینگ: جستجوی آدرس شما در quanpool.com (ورکرها، پندینگ، پرداخت‌ها)
- هر GPU = یک ورکر — ماینر را دوبار روی یک کارت اجرا نکنید.

> 🔴 هرگز عبارت بازیابی (seed phrase) را در ماینر پولی وارد نکنید — فقط آدرس `qz...`.

---

## 7. راه‌اندازی روی ویندوز با WSL2

### نصب WSL2 (PowerShell به‌صورت Administrator)

```powershell
wsl --install -d Ubuntu
```

### نکات مخصوص ویندوز

- پوشه دیتابیس نود را در **Windows Defender** استثنا (exclude) کنید، وگرنه سینک بسیار کند می‌شود.
- فلگ‌های پیشنهادی همیشه: `--sync full --max-blocks-per-request 64`
- برای GPU در WSL: درایور NVIDIA ویندوز باید نصب باشد و `nvidia-smi` داخل اوبونتو کار کند.
- اگر WSL خطای `Wsl/Service/E_UNEXPECTED` داد:
  ```powershell
  wsl --shutdown
  wsl --update
  wsl --unregister Ubuntu   # آخرین راه‌حل — دیتای داخل اوبونتو پاک می‌شود
  wsl --install -d Ubuntu
  ```

---

## 8. مانیتورینگ

| ابزار | آدرس |
|---|---|
| تله‌متری شبکه | [telemetry.quantus.cat](https://telemetry.quantus.cat) |
| متریکس نود | `http://localhost:9615/metrics` |
| RPC نود | `http://localhost:9944` |
| متریکس ماینر | `http://localhost:9900/metrics` |

مدیریت سشن‌های screen:

```bash
screen -ls       # لیست سشن‌ها
screen -r NAME   # ورود به سشن
```

---

## 9. آپدیت نود و ماینر

نسخه نود و ماینر باید همیشه **جفت و هماهنگ** باشند:

1. ماینر و نود را `stop` کنید
2. باینری‌های جدید را جایگزین کنید
3. نود را اجرا کنید
4. صبر کنید سرور ماینرِ نود بالا بیاید
5. ماینر را اجرا کنید

---

## 10. نکات امنیتی

- ۲۴ کلمه بازیابی را **آفلاین** بک‌آپ کنید (کاغذ، نه اسکرین‌شات).
- فایل auth token را مثل **رمز عبور** محافظت کنید.
- فقط پورت `30333` را عمومی expose کنید؛ 9944 و 9615 روی localhost بمانند.
- ماینرهای ریموت فقط از طریق **VPN** به نود وصل شوند.
- این **مین‌نت زنده** است — QTC ارزش واقعی دارد.

---

## نود تمام‌قد بدون ماینینگ

همان دستور نود را اجرا کنید ولی `--miner-listen-port` و `--rewards-inner-hash` را حذف کنید.

---

<div dir="rtl">

## سلب مسئولیت

این مخزن یک راهنمای آموزشی مستقل و جامعه‌محور است و وابستگی رسمی به پروژه Quantus ندارد. مسئولیت استفاده با خود شماست — همیشه نسخه‌ها و دستورات را از منابع رسمی چک کنید.

</div>
