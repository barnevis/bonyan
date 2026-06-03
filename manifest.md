# مانیفست

## مقدمه

مانیفست فایلی است که هر بخش از سیستم — افزونه، ماژول، پلتفرم، و رابط کاربری — باید داشته باشد. مانیفست تنها منبع معتبر برای شناخت یک بخش است. هسته از روی مانیفست همه اطلاعات لازم برای بارگذاری، بررسی سازگاری، و مدیریت وابستگی‌ها را به دست می‌آورد.

مانیفست با قرارداد فرق دارد. قرارداد مشخص می‌کند یک بخش چه متدهایی ارائه می‌دهد و چه پیام‌هایی منتشر می‌کند. مانیفست مشخص می‌کند یک بخش چه کسی است، چه نسخه‌ای دارد، چه قراردادی را پیاده‌سازی کرده، و به چه چیزی نیاز دارد.

هسته همیشه مانیفست و قرارداد را با هم می‌خواند. اگر یکی وجود نداشته باشد، بخش بارگذاری نمی‌شود.

## فرمت مانیفست

مانیفست یک فایل JSON است با نام `manifest.json`. هسته این فایل را قبل از بارگذاری هر بخش می‌خواند و اعتبارسنجی می‌کند.

## ساختار مانیفست

هر مانیفست از شش بخش تشکیل می‌شود:

**identity** — هویت بخش شامل نام، نوع، نسخه، توضیحات، نویسنده، و مجوز.

**compatibility** — نسخه معماری‌ای که این بخش با آن سازگار است.

**implements** — نام و نسخه قراردادی که این بخش پیاده‌سازی کرده است.

**dependencies** — وابستگی‌ها به دو دسته تقسیم می‌شوند. وابستگی‌های ضروری که بدون آن‌ها بخش کار نمی‌کند و وابستگی‌های اختیاری که اگر در دسترس بودند استفاده می‌شوند اما نبودشان مانع کار نمی‌شود.

**config** — تنظیماتی که این بخش برای راه‌اندازی نیاز دارد، به همراه مقادیر پیش‌فرض.

**bus** — اعلام نیاز به گذرگاه خصوصی. این فیلد اختیاری است و فقط در صورتی تعریف می‌شود که بخش به گذرگاه خصوصی نیاز دارد.

## اعتبارسنجی مانیفست

هسته قبل از بارگذاری هر بخش، مانیفست را در دو مرحله اعتبارسنجی می‌کند:

**مرحله اول — اعتبارسنجی ساختار:**
```text
آیا همه فیلدهای اجباری وجود دارند؟
آیا نسخه فرمت درستی دارد؟ (MAJOR.MINOR.PATCH)
آیا نوع بخش یکی از مقادیر مجاز است؟ (module, plugin, platform, ui)
آیا فیلد implements نام و نسخه قرارداد را دارد؟
```

**مرحله دوم — اعتبارسنجی محتوا:**
```text
آیا نسخه معماری با هسته سازگار است؟
آیا قرارداد اعلام‌شده در implements وجود دارد؟
آیا وابستگی‌های ضروری در دسترس هستند؟
آیا محیط اجرای مورد نیاز فعلی است؟
آیا پیکربندی‌های required مقدار دارند؟
```

اگر هر کدام از این بررسی‌ها با خطا مواجه شوند، بخش بارگذاری نمی‌شود و هسته خطای واضحی اعلام می‌کند. وابستگی‌های اختیاری در اعتبارسنجی بررسی نمی‌شوند و نبودشان خطا نیست.

## نمونه مانیفست افزونه

```json
{
  "identity": {
    "name": "my-company.plugin.indexed-db",
    "type": "plugin",
    "version": "2.1.0",
    "description": "افزونه ذخیره‌سازی مبتنی بر IndexedDB",
    "author": "my-company",
    "license": "MIT"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "implements": {
    "contract": "my-company.contract.storage",
    "version": "1.0"
  },
  "dependencies": {
    "required": [
      {
        "name": "my-company.platform.browser",
        "version": "1.0.x"
      }
    ],
    "optional": [
      {
        "name": "my-company.plugin.logger",
        "version": "1.x.x"
      }
    ]
  },
  "config": [
    {
      "key": "db-name",
      "required": true,
      "default": null
    },
    {
      "key": "db-version",
      "required": false,
      "default": 1
    },
    {
      "key": "max-size-mb",
      "required": false,
      "default": 50
    }
  ],
  "bus": {
    "private": false
  }
}
```

## نمونه مانیفست ماژول

```json
{
  "identity": {
    "name": "my-company.module.task",
    "type": "module",
    "version": "1.0.0",
    "description": "ماژول مدیریت وظایف",
    "author": "my-company",
    "license": "MIT"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "implements": {
    "contract": "my-company.contract.task",
    "version": "1.0"
  },
  "dependencies": {
    "required": [
      {
        "name": "my-company.plugin.indexed-db",
        "version": "2.x.x"
      },
      {
        "name": "my-company.plugin.auth",
        "version": "1.x.x"
      }
    ],
    "optional": [
      {
        "name": "my-company.plugin.logger",
        "version": "1.x.x"
      }
    ]
  },
  "config": [
    {
      "key": "max-tasks-per-user",
      "required": false,
      "default": 1000
    }
  ],
  "bus": {
    "private": false
  }
}
```

## نمونه مانیفست پلتفرم

```json
{
  "identity": {
    "name": "my-company.platform.browser",
    "type": "platform",
    "version": "1.0.0",
    "description": "پلتفرم مرورگر",
    "author": "my-company",
    "license": "MIT"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "implements": {
    "contract": "my-company.contract.platform",
    "version": "1.0"
  },
  "dependencies": {
    "required": [],
    "optional": []
  },
  "config": [
    {
      "key": "storage-prefix",
      "required": false,
      "default": "app"
    }
  ]
}
```

## نمونه مانیفست رابط کاربری

```json
{
  "identity": {
    "name": "my-company.ui.web",
    "type": "ui",
    "version": "1.0.0",
    "description": "رابط کاربری وب",
    "author": "my-company",
    "license": "MIT"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "implements": {
    "contract": "my-company.contract.ui",
    "version": "1.0"
  },
  "dependencies": {
    "required": [
      {
        "name": "my-company.platform.browser",
        "version": "1.x.x"
      }
    ],
    "optional": [
      {
        "name": "my-company.plugin.logger",
        "version": "1.x.x"
      }
    ]
  },
  "config": [
    {
      "key": "theme",
      "required": false,
      "default": "light"
    },
    {
      "key": "language",
      "required": false,
      "default": "fa"
    }
  ],
  "bus": {
    "private": true
  }
}
```

## مانیفست هسته

هسته برخلاف سایر بخش‌ها فایل manifest.json ندارد. هسته در bootstrap.json معرفی می‌شود و اطلاعاتش مستقیماً توسط خودش خوانده می‌شود. این استثنا به این دلیل است که هسته پیش از هر مکانیزم بارگذاری‌ای اجرا می‌شود.

## قوانین مانیفست

* هر بخش باید یک فایل `manifest.json` داشته باشد
* مانیفست باید پیش از پیاده‌سازی نوشته شود
* هسته همیشه مانیفست و قرارداد را با هم می‌خواند؛ نبود هر کدام خطاست
* وابستگی‌های ضروری باید پیش از راه‌اندازی بخش در دسترس باشند
* وابستگی‌های اختیاری در صورت وجود استفاده می‌شوند و نبودشان خطا نیست
* پیکربندی مورد نیاز هر بخش فقط در مانیفست همان بخش تعریف می‌شود
* اگر یک تنظیم `required: true` باشد و مقداری برای آن تعریف نشده باشد، هسته بخش را بارگذاری نمی‌کند