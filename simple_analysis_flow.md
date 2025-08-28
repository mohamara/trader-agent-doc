# نمودار ساده فرآیند آنالیز

## مرحله 1: شروع از پنل
```
📊 پنل کاربری
    ↓
📝 فرم آنالیز (داده + استراتژی + حالت)
    ↓
🔗 ارسال HTTP POST به /analyze
```

## مرحله 2: پردازش در API
```
🌐 API Service (/analyze)
    ↓
📦 دریافت JSON payload
    ↓
🔍 استخراج پارامترها:
   - data: داده‌های OHLC
   - strategy_name: نام استراتژی
   - strategy_mode: حالت تحلیل
   - settings: تنظیمات
```

## مرحله 3: بارگذاری استراتژی
```
📚 Strategy Registry
    ↓
📋 get_strategy_manifest()
    ↓
📦 load_strategy_from_manifest()
    ↓
⚙️ ایجاد instance استراتژی
```

## مرحله 4: پردازش داده‌ها
```
📊 تبدیل داده‌ها به DataFrame
    ↓
🧹 پیش‌پردازش داده‌ها:
   - اضافه کردن volume (در صورت عدم وجود)
   - تبدیل تاریخ
   - مرتب‌سازی
    ↓
✅ اعتبارسنجی داده‌ها
```

## مرحله 5: اجرای تحلیل استراتژی
```
🎯 strategy.analyze()
    ↓
🔍 detect_order_blocks()
    ↓
📈 detect_moves() - تشخیص حرکت‌های قیمت
    ↓
📊 برای هر حرکت:
   ├─ analyze_candle() - تحلیل شمع
   ├─ analyze_volume() - تحلیل حجم
   └─ calculate_strength() - محاسبه قدرت
```

## مرحله 6: پردازش نتایج
```
📋 ساخت OrderBlockResult
    ↓
🔧 فیلتر کردن نتایج:
   ├─ حذف order blocks همپوشان
   ├─ فیلتر بر اساس قدرت
   └─ محدودیت تعداد
    ↓
📊 ساخت پاسخ نهایی
```

## مرحله 7: بازگشت به پنل
```
📤 ارسال JSON Response
    ↓
📱 پنل - دریافت پاسخ
    ↓
🎨 updateAnalysisResults()
    ↓
📊 نمایش نتایج در UI
```

---

## متغیرهای مهم در هر مرحله:

### ورودی
```javascript
{
  "data": [
    {"date": "2023-01-01", "open": 1850, "high": 1855, "low": 1848, "close": 1852},
    // ... more candles
  ],
  "strategy_name": "orderblock",
  "strategy_mode": "bullish",
  "settings": {
    "min_strength_threshold": 0.3,
    "max_ob_lookback": 50
  }
}
```

### خروجی
```javascript
{
  "status": "success",
  "strategy": {
    "name": "OrderBlock Strategy",
    "version": "1.0.0",
    "mode": "bullish"
  },
  "analysis": {
    "bullish_blocks": [...],
    "bearish_blocks": [...],
    "total_blocks": 15,
    "strong_blocks": 8
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## فایل‌های کلیدی:

| مرحله | فایل | متد اصلی |
|-------|------|----------|
| پنل | `panel/script.js` | `handleAnalysisSubmit()` |
| API | `api_service.py` | `analyze_data()` |
| استراتژی | `orderblock_strategy.py` | `analyze()` |
| رجیستری | `strategy_registry.py` | `load_strategy()` |
| موتور | `strategy_engine.py` | `execute_analysis()` | 