# Quantus Network — راهنمای کامل راه‌اندازی نود و ماینر (QTC Mining Guide)

> راهنمای جامع فارسی برای اجرای نود مین‌نت **Quantus** و ماینینگ کوین **QTC**
> بر اساس آموزش رسمی جامعه: [github.com/0xmoei/quantus](https://github.com/0xmoei/quantus/)

---

## 📌 Quantus چیست؟

Quantus یک بلاکچین **اثبات کار پساکوانتومی (QPoW)** است که از هشینگ **Poseidon2** استفاده می‌کند و پاداش ماینینگ به یک **آدرس Wormhole** ارسال می‌شود (نه یک والت معمولی).

| مشخصه | مقدار |
|---|---|
| نماد | QTC |
| حداکثر عرضه | 21,000,000 |
| اعشار | 12 رقم |
| پیشوند آدرس | `qz...` (SS58 prefix: 189) |
| زمان بلوک هدف | ~12 ثانیه |
| نسخه نود | v1.0.1 |
| نسخه ماینر | v4.1.0 |
| چین | `--chain mainnet` |
| پروتکل ماینر | `quantus-miner/2` (احراز هویت + TLS pin الزامی) |

⚠️ **مهم:** قبل از نصب، صفحه ریلیزهای نود و ماینر را جداگانه چک کنید چون ممکن است نسخه‌ها به‌روز شده باشند.

---

## 💻 پیش‌نیازهای سخت‌افزاری

| منبع | حداقل | پیشنهادی |
|---|---|---|
| CPU | 2+ هسته | 4+ هسته |
| RAM | 4 GB | 8 GB+ |
| دیسک | 100 GB SSD | 500 GB+ SSD |
| اینترنت | 3+ Mbps | 10+ Mbps |

- **سیستم‌عامل:** Ubuntu 20.04+ / macOS 10.15+ / Windows 10-11 (از طریق WSL2 یا بیلد MSVC)
- باینری ماینر رسمی برای Linux ARM64 وجود ندارد.
- **GPU** به‌شدت پیشنهاد می‌شود (Metal / Vulkan / DirectX یا NVIDIA CUDA با `--cuda-gpu`).
- ماینینگ با نود داخلی روی CPU فقط حدود ~15 MH/s به‌ازای هر thread است؛ از **ماینر خارجی** استفاده کنید.

---

## 👛 مرحله ۱ — ساخت والت

1. والت رسمی را از اینجا بگیرید: [linktr.ee/quantusnetwork](https://linktr.ee/quantusnetwork)
2. یک والت جدید بسازید یا عبارت ۲۴ کلمه‌ای (seed phrase) خود را بازیابی کنید.
3. **نکته حیاتی:** برای ساخت آدرس Wormhole **همان عبارت ۲۴ کلمه‌ای** را استفاده کنید تا پاداش‌ها داخل اپ نمایش داده شوند.
4. عبارت ۲۴ کلمه‌ای را **آفلاین** بک‌آپ کنید. هرگز آن را با کسی به اشتراک نگذارید.

---

## 🔥 مرحله ۲ — فایروال (لینوکس)

فقط پورت P2P باید عمومی باشد:

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl wget tar unzip jq screen ufw ca-certificates -y

sudo ufw allow 22
sudo ufw allow ssh
sudo ufw allow 30333/tcp
sudo ufw enable
```

---

## 🚀 روش ۱ — اسکریپت رسمی (ساده‌ترین — نود خودت + سولو ماینینگ)

```bash
curl -fsSL https://docs.quantus.com/scripts/quantus-mining.sh -o quantus-mining.sh
chmod +x quantus-mining.sh
./quantus-mining.sh setup
```

### اجرا و توقف

```bash
./quantus-mining.sh start -d      # اجرای نود + ماینر در بک‌گراند
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
./quantus-mining.sh config set GPU_DEVICES 1
./quantus-mining.sh config set CPU_WORKERS 4
```

> تغییر نسخه نیاز به `setup --force` دارد؛ این کار INNER_HASH و آدرس شما را حفظ می‌کند.

---

## 🛠️ روش ۲ — نصب دستی (Linux x86_64)

### 1) دانلود باینری‌ها

```bash
mkdir -p ~/quantus && cd ~/quantus
# تاربال نود (v1.0.1) و باینری ماینر (v4.1.0) را از صفحه GitHub Releases رسمی دانلود و اکسترکت کنید
chmod +x quantus-node quantus-miner

# چک کردن فلگ‌ها
./quantus-node --help | grep miner-listen-port
./quantus-miner --help | grep auth-token-file
```

### 2) ساخت کلید P2P نود

```bash
./quantus-node key generate-node-key --file node_key.p2p
```

### 3) ساخت کلیدهای Wormhole (ورودی مخفی — عبارت در history ذخیره نمی‌شود)

```bash
./quantus-node key quantus --scheme wormhole --words
```

خروجی را ذخیره کنید: **Address** ، **Inner Hash** و ۲۴ کلمه بک‌آپ.

### 4) اجرای نود (داخل screen)

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

- خروج از screen: `Ctrl+A` سپس `D`
- بازگشت به screen: `screen -r quantus-node`

### 5) صبر برای سینک کامل

از چند دقیقه تا چند ساعت طول می‌کشد. وقتی وضعیت از `Syncing` به `Idle` رسید، نود آماده است.

> ⚠️ ماینینگ قبل از رسیدن به سر زنجیره باعث تولید بلوک‌های یتیم (orphan) می‌شود.

### 6) اجرای ماینر (screen جدید)

فایل‌های `--auth-token-file` و `--tls-cert-sha256-file` را از مسیر زیر بردارید:
`~/.local/share/quantus-node/chains/mainnet/` (مسیر در macOS متفاوت است)

```bash
screen -S quantus-miner

./quantus-miner \
  --node-addr 127.0.0.1:9833 \
  --auth-token-file /path/to/auth_token \
  --tls-cert-sha256-file /path/to/tls_sha256 \
  --cpu-workers 4 \
  --gpu-devices 1
```

- فقط CUDA: `--cuda-gpu --gpu-devices 1 --cpu-workers 0`
- ❗ توکن را هرگز inline پاس ندهید. توکن یا TLS pin اشتباه یک **خطای دائمی** است.

---

## 🏊 روش ۳ — ماینینگ پولی Quanpool (بدون نیاز به نود)

- فقط یک آدرس پرداخت `qz...` لازم است (**هرگز** عبارت بازیابی را وارد نکنید).
- بدون ثبت‌نام.
- کارمزد: 1% پول + 5% ماینر (مجموعاً 6%)
- روش: PPLNS یا Solo — حداقل پرداخت: 0.25 QTC — بازه پرداخت: ~1 ساعت — تاییدیه: 105

### راه‌اندازی

1. دانلود ماینر: `quanpool-miner-6.0.0-linux-x86_64` از `download.quanpool.com`
2. به سایت [quanpool.com](https://quanpool.com) بروید → تب **Start mining** → دستور را **زنده** از سایت کپی کنید (IP پول و TLS pin ممکن است تغییر کنند).

شکل کلی دستور:

```bash
screen -S quanpool

./quanpool-miner-6.0.0-linux-x86_64 \
  --node-addr <pool-ip>:9834 \
  --auth-token qzYOUR_ADDRESS.worker \
  --tls-cert-sha256 <pin>
```

- مانیتورینگ: جستجوی آدرس در quanpool.com (ورکرها، پندینگ، پرداخت‌ها)
- هر GPU 4090 = یک ورکر؛ ماینر را **دو بار** روی یک کارت اجرا نکنید.

---

## 🪟 نکات ویندوز (WSL2)

- پوشه دیتابیس نود را در **Windows Defender** استثنا کنید، وگرنه سینک کند می‌شود.
- فلگ‌های پیشنهادی: `--sync full --max-blocks-per-request 64`
- برای GPU (CUDA) در WSL، درایور NVIDIA ویندوز باید نصب باشد و `nvidia-smi` داخل WSL کار کند.

نصب سریع WSL2 (PowerShell با دسترسی ادمین):

```powershell
wsl --install -d Ubuntu
```

---

## 🔒 نود تمام‌قد بدون ماینینگ

همان دستور نود را اجرا کنید ولی `--miner-listen-port` و `--rewards-inner-hash` را حذف کنید.
پورت‌های 9944 (RPC) و 9615 (متریکس) فقط روی localhost بمانند.

---

## 📊 مانیتورینگ

| ابزار | آدرس |
|---|---|
| تله‌متری | [telemetry.quantus.cat](https://telemetry.quantus.cat) |
| متریکس نود | `http://localhost:9615/metrics` |
| RPC نود | `http://localhost:9944` |
| متریکس ماینر | `http://localhost:9900/metrics` |

مدیریت screen:

```bash
screen -ls      # لیست سشن‌ها
screen -r NAME  # ورود به سشن
```

---

## 🔄 آپدیت نود و ماینر

نسخه نود و ماینر باید **جفت** باشند:

1. `stop` کنید
2. باینری‌های جدید را جایگزین کنید
3. نود را اجرا کنید
4. منتظر بالا آمدن سرور ماینر نود بمانید
5. ماینر را اجرا کنید

---

## 🛡️ نکات امنیتی

- ۲۴ کلمه بازیابی را **آفلاین** بک‌آپ کنید.
- با فایل auth token مثل **رمز عبور** رفتار کنید.
- فقط پورت `30333` را عمومی expose کنید.
- ماینرهای ریموت را فقط از طریق **VPN** به نود وصل کنید.
- این **مین‌نت زنده** است — QTC ارزش واقعی دارد.

---

## 🙏 منابع

- آموزش کامل جامعه: [github.com/0xmoei/quantus](https://github.com/0xmoei/quantus/)
- مستندات رسمی: [docs.quantus.com](https://docs.quantus.com)
- والت و لینک‌ها: [linktr.ee/quantusnetwork](https://linktr.ee/quantusnetwork)
- پول Quanpool: [quanpool.com](https://quanpool.com)

---

## ⚖️ سلب مسئولیت

این مخزن یک راهنمای آموزشی مستقل و جامعه‌محور است و وابستگی رسمی به پروژه Quantus ندارد. مسئولیت هرگونه استفاده از این آموزش با خود شماست. همیشه نسخه‌های رسمی و دستورات زنده را از منابع رسمی چک کنید.
