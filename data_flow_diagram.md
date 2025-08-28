# نمودار جریان داده (Data Flow Diagram)

## جریان کلی داده

```
[USER DATA] → [PANEL] → [API] → [STRATEGY] → [ANALYSIS] → [RESULTS] → [PANEL] → [DISPLAY]
```

## جزئیات هر مرحله:

### 1️⃣ ورودی کاربر
```
📊 داده‌های OHLC:
[
  {"date": "2023-01-01", "open": 1850, "high": 1855, "low": 1848, "close": 1852},
  {"date": "2023-01-02", "open": 1852, "high": 1860, "low": 1850, "close": 1858},
  ...
]

⚙️ تنظیمات:
{
  "strategy_name": "orderblock",
  "strategy_mode": "bullish", 
  "settings": {
    "min_strength_threshold": 0.3,
    "max_ob_lookback": 50
  }
}
```

### 2️⃣ پردازش در پنل
```
🔧 handleAnalysisSubmit():
├─ دریافت فرم
├─ parse JSON data
├─ ساخت requestData
└─ ارسال HTTP POST
```

### 3️⃣ دریافت در API
```
📥 /analyze endpoint:
├─ دریافت JSON payload
├─ استخراج پارامترها
├─ بارگذاری استراتژی
└─ تبدیل به DataFrame
```

### 4️⃣ پردازش استراتژی
```
🎯 OrderBlockStrategy.analyze():
├─ detect_moves() - تشخیص حرکت‌ها
├─ برای هر حرکت:
│  ├─ analyze_candle() - تحلیل شمع
│  ├─ analyze_volume() - تحلیل حجم
│  └─ calculate_strength() - محاسبه قدرت
└─ فیلتر نتایج
```

### 5️⃣ ساخت نتایج
```
📋 OrderBlockResult:
{
  "bullish_blocks": [
    {
      "index": 15,
      "high": 1865.5,
      "low": 1860.2,
      "strength": 0.85,
      "timestamp": "2023-01-15T10:30:00",
      "block_type": "bullish"
    }
  ],
  "bearish_blocks": [...],
  "total_blocks": 15,
  "strong_blocks": 8
}
```

### 6️⃣ پاسخ نهایی
```
📤 JSON Response:
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

### 7️⃣ نمایش در پنل
```
🎨 updateAnalysisResults():
├─ دریافت JSON response
├─ استخراج آمار
├─ نمایش در UI
└─ فعال کردن دکمه‌ها
```

---

## نمودار ساده جریان:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   USER      │    │   PANEL     │    │     API     │    │  STRATEGY   │
│             │    │             │    │             │    │             │
│ 📊 OHLC     │───▶│ 📝 Form     │───▶│ 📥 /analyze │───▶│ 🎯 analyze  │
│ ⚙️ Settings │    │ 🔗 Submit   │    │ 📦 Payload  │    │ 📈 detect   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                              │
                                                              ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   DISPLAY   │    │   PANEL     │    │     API     │    │   RESULTS   │
│             │    │             │    │             │    │             │
│ 📊 Charts   │◀───│ 🎨 Update   │◀───│ 📤 Response │◀───│ 📋 OrderBlock│
│ 📈 Stats    │    │ 📱 Results  │    │ 🏷️ JSON     │    │ 🎯 Result   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

## متغیرهای کلیدی در هر مرحله:

| مرحله | متغیر ورودی | متغیر خروجی | فایل |
|-------|-------------|-------------|-------|
| **پنل** | `formData` | `requestData` | `panel/script.js` |
| **API** | `payload` | `strategy`, `df` | `api_service.py` |
| **استراتژی** | `df`, `mode`, `settings` | `OrderBlockResult` | `orderblock_strategy.py` |
| **نتایج** | `OrderBlockResult` | `JSON Response` | `strategy_engine.py` |
| **نمایش** | `JSON Response` | `UI Update` | `panel/script.js` |

## فایل‌های کلیدی:

### Frontend
- `panel/script.js` - مدیریت UI و API calls
- `panel/index.html` - ساختار پنل

### Backend  
- `core-engine/src/services/api_service.py` - API endpoints
- `core-engine/src/analysis/strategy_engine.py` - موتور تحلیل
- `core-engine/src/strategies/orderblock_strategy.py` - پیاده‌سازی استراتژی
- `core-engine/src/core/strategy_registry.py` - مدیریت استراتژی‌ها

### Interface
- `core-engine/src/core/strategy_interface.py` - رابط استراتژی‌ها 