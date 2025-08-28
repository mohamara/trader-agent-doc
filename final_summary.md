# خلاصه نهایی فرآیند آنالیز استراتژی معاملاتی

## 🔄 مراحل کلی (7 مرحله)

### 1️⃣ **ورودی کاربر** → پنل
```
📊 داده‌های OHLC + ⚙️ تنظیمات استراتژی
```

### 2️⃣ **پردازش پنل** → API
```
🔧 handleAnalysisSubmit() → 🌐 HTTP POST /analyze
```

### 3️⃣ **دریافت API** → استراتژی
```
📥 دریافت JSON → 📚 بارگذاری استراتژی → 📊 تبدیل به DataFrame
```

### 4️⃣ **تحلیل استراتژی** → نتایج
```
🎯 detect_order_blocks() → 📈 detect_moves() → 💪 calculate_strength()
```

### 5️⃣ **فیلتر نتایج** → پاسخ
```
🔧 حذف همپوشان → 📋 ساخت OrderBlockResult → 📤 JSON Response
```

### 6️⃣ **بازگشت به پنل** → نمایش
```
📱 دریافت پاسخ → 🎨 updateAnalysisResults() → 📊 نمایش در UI
```

### 7️⃣ **نمایش نهایی** → کاربر
```
📈 نمودارها + 📊 آمار + 🔘 دکمه‌های عملیات
```

---

## 📋 متغیرهای کلیدی

### ورودی
```javascript
{
  "data": [{"date": "...", "open": 1850, "high": 1855, "low": 1848, "close": 1852}],
  "strategy_name": "orderblock",
  "strategy_mode": "bullish",
  "settings": {"min_strength_threshold": 0.3}
}
```

### خروجی
```javascript
{
  "status": "success",
  "analysis": {
    "bullish_blocks": [{"index": 15, "strength": 0.85, "high": 1865.5, "low": 1860.2}],
    "bearish_blocks": [...],
    "total_blocks": 15,
    "strong_blocks": 8
  }
}
```

---

## 🔧 متدهای اصلی

| مرحله | فایل | متد | وظیفه |
|-------|------|-----|-------|
| **پنل** | `panel/script.js` | `handleAnalysisSubmit()` | دریافت فرم و ارسال درخواست |
| **API** | `api_service.py` | `analyze_data()` | دریافت و پردازش درخواست |
| **رجیستری** | `strategy_registry.py` | `load_strategy()` | بارگذاری استراتژی |
| **استراتژی** | `orderblock_strategy.py` | `analyze()` | اجرای تحلیل اصلی |
| **موتور** | `strategy_engine.py` | `execute_analysis()` | هماهنگی تحلیل |

---

## 📊 فرآیند تحلیل Order Blocks

### مرحله 1: تشخیص حرکت‌ها
```
📈 detect_moves()
├─ اسکن داده‌ها برای یافتن حرکت‌های قوی
├─ محاسبه قدرت هر حرکت
└─ تعیین جهت (صعودی/نزولی)
```

### مرحله 2: یافتن Order Blocks
```
🔍 برای هر حرکت:
├─ analyze_candle() - تحلیل شمع
├─ analyze_volume() - تحلیل حجم  
└─ calculate_strength() - محاسبه قدرت
```

### مرحله 3: فیلتر کردن
```
🔧 فیلترهای اعمال شده:
├─ حذف order blocks همپوشان
├─ فیلتر بر اساس حداقل قدرت
└─ محدودیت تعداد نتایج
```

---

## 🎯 محاسبات کلیدی

### قدرت Order Block
```
strength = (price_change_score × 0.5) + 
          (consistency_score × 0.3) + 
          (volume_score × 0.2)
```

### تحلیل شمع
```
body_size = abs(close - open)
price_range = high - low
wick_ratio = (upper_wick + lower_wick) / price_range
body_ratio = body_size / price_range
```

### تایید حجم
```
volume_score = average_volume / lookback_average
volume_confirmed = volume_score > threshold
```

---

## 📁 ساختار فایل‌ها

```
trader-agent/
├── panel/                          # پنل کاربری
│   ├── script.js                   # منطق پنل
│   └── index.html                  # ساختار HTML
│
└── core-engine/                    # موتور اصلی
    └── src/
        ├── services/
        │   └── api_service.py      # API endpoints
        ├── analysis/
        │   └── strategy_engine.py  # موتور تحلیل
        ├── core/
        │   ├── strategy_registry.py # مدیریت استراتژی‌ها
        │   └── strategy_interface.py # رابط استراتژی‌ها
        └── strategies/
            └── orderblock_strategy.py # پیاده‌سازی استراتژی
```

---

## ⚡ نکات مهم

### عملکرد
- **Batch Mode**: پردازش کل داده‌ها یکجا
- **Caching**: ذخیره نتایج برای داده‌های مشابه
- **Validation**: اعتبارسنجی داده‌ها قبل از تحلیل

### امنیت
- **Input Validation**: بررسی ورودی‌های کاربر
- **Error Handling**: مدیریت خطاها
- **Logging**: ثبت عملیات

### قابلیت توسعه
- **Modular Design**: طراحی ماژولار
- **Strategy Interface**: رابط یکسان برای همه استراتژی‌ها
- **Registry System**: سیستم ثبت خودکار استراتژی‌ها

---

## 🎉 نتیجه نهایی

وقتی کاربر درخواست آنالیز می‌دهد:

1. **داده‌ها** از پنل به API ارسال می‌شوند
2. **استراتژی** بارگذاری و اجرا می‌شود  
3. **Order Blocks** تشخیص داده می‌شوند
4. **نتایج** فیلتر و مرتب می‌شوند
5. **پاسخ** به پنل بازگردانده می‌شود
6. **نتایج** در UI نمایش داده می‌شوند

کل این فرآیند در کمتر از چند ثانیه انجام می‌شود و نتایج دقیق و قابل اعتماد تولید می‌کند. 