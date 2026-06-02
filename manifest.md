# مانیفست

## مقدمه

مانیفست فایلی است که هر بخش از سیستم — افزونه، ماژول، پلتفرم، و رابط کاربری — باید داشته باشد. مانیفست تنها منبع معتبر برای شناخت یک بخش است. هسته از روی مانیفست همه اطلاعات لازم برای بارگذاری، بررسی سازگاری، و مدیریت وابستگی‌ها را به دست می‌آورد.

مانیفست با قرارداد فرق دارد. قرارداد مشخص می‌کند یک بخش چه متدهایی ارائه می‌دهد. مانیفست مشخص می‌کند یک بخش چه کسی است، چه نسخه‌ای دارد، به چه چیزی نیاز دارد، و چه رویدادهایی منتشر می‌کند.

هسته همیشه مانیفست و قرارداد را با هم می‌خواند. اگر یکی وجود نداشته باشد، بخش بارگذاری نمی‌شود.

## فرمت مانیفست

مانیفست یک فایل JSON است با نام `manifest.json`. هسته این فایل را قبل از بارگذاری هر بخش می‌خواند و اعتبارسنجی می‌کند.

## ساختار مانیفست

هر مانیفست از شش بخش تشکیل می‌شود:

**identity** — هویت بخش شامل نام، نوع، و نسخه.

**compatibility** — نسخه معماری‌ای که این بخش با آن سازگار است.

**contract** — نام قراردادی که این بخش پیاده‌سازی کرده است.

**dependencies** — وابستگی‌ها به افزونه‌های دیگر یا محیط اجرا.

**config** — تنظیماتی که این بخش برای راه‌اندازی نیاز دارد، به همراه مقادیر پیش‌فرض.

**events** — رویدادهایی که این بخش منتشر می‌کند.

**bus** — اعلام نیاز به گذرگاه خصوصی. این فیلد اختیاری است و فقط در صورتی تعریف می‌شود که بخش داده‌های حساس رد و بدل می‌کند یا رویدادهای پرتکرار داخلی دارد که ربطی به بقیه بخش‌ها ندارد.

## اعتبارسنجی مانیفست

هسته قبل از بارگذاری هر بخش، مانیفست را در دو مرحله اعتبارسنجی می‌کند:

**مرحله اول — اعتبارسنجی ساختار:**
```text
آیا همه فیلدهای اجباری وجود دارند؟
آیا نسخه فرمت درستی دارد؟ (MAJOR.MINOR.PATCH)
آیا نوع بخش یکی از مقادیر مجاز است؟ (module, plugin, platform, ui)
```

**مرحله دوم — اعتبارسنجی محتوا:**
```text
آیا نسخه معماری با هسته سازگار است؟
آیا وابستگی‌های اعلام‌شده در دسترس هستند؟
آیا محیط اجرای مورد نیاز فعلی است؟
آیا پیکربندی‌های required مقدار دارند؟
```

اگر هر کدام از این بررسی‌ها با خطا مواجه شوند، بخش بارگذاری نمی‌شود و هسته خطای واضحی اعلام می‌کند.

## نمونه مانیفست افزونه

```json
{
  "identity": {
    "name": "my-company.plugin.indexed-db",
    "type": "plugin",
    "version": "2.1.0"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "contract": "my-company.contract.storage",
  "dependencies": {
    "plugins": [],
    "platforms": [
      {
        "name": "my-company.platform.browser",
        "version": "1.0.x"
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
  "events": [
    {
      "name": "storage:ready",
      "description": "افزونه آماده استفاده است"
    },
    {
      "name": "storage:synced",
      "description": "داده‌ها با موفقیت ذخیره شدند"
    },
    {
      "name": "storage:failed",
      "description": "عملیات ذخیره‌سازی با خطا مواجه شد"
    }
  ]
}
```

## نمونه مانیفست ماژول

```json
{
  "identity": {
    "name": "my-company.module.task",
    "type": "module",
    "version": "1.0.0"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "contract": "my-company.contract.task",
  "dependencies": {
    "plugins": [
      {
        "name": "my-company.plugin.indexed-db",
        "version": "2.x.x"
      },
      {
        "name": "my-company.plugin.auth",
        "version": "1.x.x"
      }
    ]
  },
  "bus": {
    "private": false
  },
  "config": [
    {
      "key": "max-tasks-per-user",
      "required": false,
      "default": 1000
    }
  ],
  "events": [
    {
      "name": "task:created",
      "description": "وظیفه جدید ایجاد شد"
    },
    {
      "name": "task:updated",
      "description": "وظیفه به‌روز شد"
    },
    {
      "name": "task:deleted",
      "description": "وظیفه حذف شد"
    }
  ]
}
```

## نمونه مانیفست پلتفرم

```json
{
  "identity": {
    "name": "my-company.platform.browser",
    "type": "platform",
    "version": "1.0.0"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "contract": "my-company.contract.platform",
  "dependencies": {
    "plugins": [],
    "platforms": []
  },
  "config": [
    {
      "key": "storage-prefix",
      "required": false,
      "default": "app"
    }
  ],
  "events": [
    {
      "name": "platform:ready",
      "description": "پلتفرم آماده است"
    },
    {
      "name": "platform:offline",
      "description": "اتصال به شبکه قطع شد"
    },
    {
      "name": "platform:online",
      "description": "اتصال به شبکه برقرار شد"
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
    "version": "1.0.0"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "contract": "my-company.contract.ui",
  "dependencies": {
    "platforms": [
      {
        "name": "my-company.platform.browser",
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
  "events": [
    {
      "name": "ui:ready",
      "description": "رابط کاربری آماده است"
    }
  ]
}
```

## مانیفست هسته

هسته برخلاف سایر بخش‌ها فایل manifest.json ندارد. هسته در bootstrap.json معرفی می‌شود و اطلاعاتش مستقیماً توسط خودش خوانده می‌شود. این استثنا به این دلیل است که هسته پیش از هر مکانیزم بارگذاری‌ای اجرا می‌شود.

## قوانین مانیفست

* هر بخش باید یک فایل `manifest.json` داشته باشد
* مانیفست باید پیش از پیاده‌سازی نوشته شود
* هسته همیشه مانیفست و قرارداد را با هم می‌خواند؛ نبود هر کدام خطاست
* پیکربندی مورد نیاز هر بخش فقط در مانیفست همان بخش تعریف می‌شود
* مقادیر پیکربندی در زمان راه‌اندازی از فایل پیکربندی راه‌اندازی خوانده می‌شوند و با مقادیر پیش‌فرض مانیفست تکمیل می‌شوند
* اگر یک تنظیم `required: true` باشد و مقداری برای آن تعریف نشده باشد، هسته بخش را بارگذاری نمی‌کند
* وابستگی‌های اعلام‌شده در مانیفست باید دقیقاً با آنچه در پیکربندی راه‌اندازی فهرست شده مطابقت داشته باشند