# بنیان — معماری ماژولار برای توسعه محصول

بنیان یک معماری ماژولار و چندسکویی برای توسعه محصولات نرم‌افزاری مبتنی بر JavaScript، HTML و CSS است. این معماری بدون وابستگی به چارچوب‌های سنگین طراحی شده و هدف اصلی آن ساده‌سازی توسعه توسط انسان و مدل‌های هوش مصنوعی است.

[![هوش‌مصنوعی](https://img.shields.io/badge/Built%20with-AI-blueviolet)](#)
[![نسخه](https://img.shields.io/badge/version-0.1-blue.svg)](package.json)
[![مجوز](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

> این معماری با کمک مدل‌های هوش مصنوعی Claude Sonnet 4.6 و GLM 5.1 نوشته شده است.

## چرا بنیان؟

بسیاری از معماری‌های موجود برای تیم‌های بزرگ و پروژه‌های پیچیده طراحی شده‌اند و وابستگی‌های سنگینی دارند. بنیان در نقطه مقابل این رویکرد قرار دارد:

- **بدون چارچوب سنگین** — هیچ وابستگی خارجی در هسته وجود ندارد
- **قراردادمحور** — هر بخش از طریق قرارداد مشخص با بقیه ارتباط برقرار می‌کند
- **قابل فهم برای هوش مصنوعی** — ساختارهای واضح و قراردادهای صریح توسعه با مدل‌های هوش مصنوعی را ساده می‌کند
- **قابل تکامل** — بدون بازنویسی کامل می‌توان آن را در بلندمدت توسعه داد

## ساختار کلی

سامانه از پنج بخش اصلی تشکیل شده است:

```
        ماژول‌ها
       ↙         ↘
رابط کاربری    افزونه‌ها
       ↘         ↙
          هسته
            ↓
          پلتفرم
```

| بخش | مسئولیت |
|-----|---------|
| **هسته** | هماهنگی میان بخش‌ها از طریق گذرگاه رویداد |
| **ماژول‌ها** | منطق و قابلیت‌های اصلی محصول |
| **افزونه‌ها** | زیرساخت‌های مشترک مثل ذخیره‌سازی و احراز هویت |
| **رابط کاربری** | نمایش داده و دریافت تعامل کاربر |
| **پلتفرم** | اتصال به محیط اجرا (مرورگر، موبایل، دسکتاپ) |

## اسناد

اسناد به دو زبان فارسی و انگلیسی در دسترس هستند:

| سند | فارسی | English |
|-----|-------|---------|
| معماری کلی | [architecture.md](docs/fa/architecture.md) | [architecture.md](docs/en/architecture.md) |
| هسته | [core.md](docs/fa/core.md) | [core.md](docs/en/core.md) |
| ماژول | [module.md](docs/fa/module.md) | [module.md](docs/en/module.md) |
| افزونه | [plugin.md](docs/fa/plugin.md) | [plugin.md](docs/en/plugin.md) |
| پلتفرم | [platform.md](docs/fa/platform.md) | [platform.md](docs/en/platform.md) |
| رابط کاربری | [ui.md](docs/fa/ui.md) | [ui.md](docs/en/ui.md) |
| قرارداد | [contract.md](docs/fa/contract.md) | [contract.md](docs/en/contract.md) |
| مانیفست | [manifest.md](docs/fa/manifest.md) | [manifest.md](docs/en/manifest.md) |
| پیکربندی راه‌اندازی | [bootstrap.md](docs/fa/bootstrap.md) | [bootstrap.md](docs/en/bootstrap.md) |
| نام‌گذاری | [naming.md](docs/fa/naming.md) | [naming.md](docs/en/naming.md) |
| نسخه‌بندی | [versioning.md](docs/fa/versioning.md) | [versioning.md](docs/en/versioning.md) |
| مدیریت خطا | [error.md](docs/fa/error.md) | [error.md](docs/en/error.md) |
| امنیت | [security.md](docs/fa/security.md) | [security.md](docs/en/security.md) |

## اصول کلیدی

**سادگی بر پیچیدگی مقدم است** — هر بخش باید قابل درک، رفتارش قابل پیش‌بینی، و بدون نیاز به دانش پنهان قابل توسعه باشد.

**همه چیز از طریق قراردادها** — اجزای محصول نباید مستقیماً به پیاده‌سازی یکدیگر وابسته باشند.

**ایزوله‌سازی خطا** — خرابی یک بخش نباید کل سیستم را متوقف کند.

**مالکیت صریح داده** — هر داده یک مالک مشخص دارد و هیچ بخشی داده بخش دیگر را مستقیماً تغییر نمی‌دهد.

## شروع کار

برای شروع پیشنهاد می‌شود این اسناد را به ترتیب بخوانید:

1. [architecture.md](docs/fa/architecture.md) — درک کلی معماری
2. [manifest.md](docs/fa/manifest.md) و [contract.md](docs/fa/contract.md) — درک قراردادها
3. [bootstrap.md](docs/fa/bootstrap.md) — درک نحوه راه‌اندازی
4. [core.md](docs/fa/core.md) — شروع پیاده‌سازی از هسته

## مشارکت

این معماری یک سند زنده است و با توجه به تجربه پیاده‌سازی‌های واقعی تکامل پیدا می‌کند. اگر در پیاده‌سازی به موردی برخوردید که معماری پاسخ روشنی به آن نمی‌دهد، یک issue باز کنید.

## مجوز

MIT