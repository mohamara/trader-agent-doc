# نمودار متدها و متغیرها

## جریان متدها از پنل تا نتیجه

```
📊 PANEL
├─ handleAnalysisSubmit(event)
│  ├─ formData = new FormData(event.target)
│  ├─ dataInput = formData.get("data")
│  ├─ parsedData = JSON.parse(dataInput)
│  ├─ requestData = {
│  │   data: parsedData,
│  │   strategy_name: strategyName,
│  │   strategy_mode: strategyMode,
│  │   settings: savedSettings
│  │ }
│  └─ result = await makeAPIRequest("/analyze", "POST", requestData)
│
└─ updateAnalysisResults(result)
   ├─ analysis = result.analysis
   ├─ summary = result.summary
   └─ displayResults(analysis, summary)
```

```
🌐 API SERVICE
├─ analyze_data() [POST /analyze]
│  ├─ payload = request.get_json()
│  ├─ data_records = payload.get('data')
│  ├─ strategy_name = payload.get('strategy_name')
│  ├─ strategy_mode = payload.get('strategy_mode')
│  ├─ settings = payload.get('settings')
│  ├─ strategy = self.engine.load_strategy(strategy_name)
│  ├─ df = pd.DataFrame(data_records)
│  ├─ res = strategy.analyze(df, mode=strategy_mode, settings=settings)
│  └─ return jsonify({"status": "success", "analysis": res})
│
└─ load_strategy(strategy_name)
   ├─ manifest = self.get_strategy_manifest(strategy_name)
   ├─ strategy = self.strategy_loader.load_strategy_from_manifest(manifest)
   └─ return strategy
```

```
🎯 STRATEGY ENGINE
├─ strategy.analyze(df, mode, settings)
│  ├─ if len(data) < min_candles: return OrderBlockResult()
│  ├─ result = self.order_block_detector.detect_order_blocks(data)
│  └─ return result
│
└─ detect_order_blocks(data)
   ├─ moves = self.move_detector.detect_moves(data)
   ├─ for move in moves:
   │  ├─ if move.direction == BULLISH:
   │  │  └─ bullish_blocks = self._find_bullish_order_blocks(data, move)
   │  └─ if move.direction == BEARISH:
   │     └─ bearish_blocks = self._find_bearish_order_blocks(data, move)
   ├─ filtered_blocks = self._filter_overlapping_blocks(all_blocks)
   └─ return OrderBlockResult(bullish_blocks, bearish_blocks)
```

```
📈 ANALYSIS COMPONENTS
├─ detect_moves(data)
│  ├─ moves = []
│  ├─ for i in range(len(data)):
│  │  ├─ if is_bullish_move(data, i):
│  │  │  ├─ move = _analyze_move(data, start_idx, end_idx, BULLISH)
│  │  │  └─ moves.append(move)
│  │  └─ if is_bearish_move(data, i):
│  │     ├─ move = _analyze_move(data, start_idx, end_idx, BEARISH)
│  │     └─ moves.append(move)
│  └─ return moves
│
├─ _find_bullish_order_blocks(data, move)
│  ├─ blocks = []
│  ├─ for i in range(move.start_index - lookback, move.start_index):
│  │  ├─ candle = data.iloc[i]
│  │  ├─ candle_type, body_size, wick_ratio = analyze_candle(candle)
│  │  ├─ volume_confirmed = analyze_volume_confirmation(data, i, move)
│  │  ├─ strength = calculate_order_block_strength(move, volume_score, body_ratio)
│  │  └─ if strength >= min_threshold:
│  │     └─ blocks.append(OrderBlock(...))
│  └─ return blocks
│
└─ analyze_candle(candle)
   ├─ body_size = abs(close - open)
   ├─ price_range = high - low
   ├─ wick_ratio = (upper_wick + lower_wick) / price_range
   ├─ body_ratio = body_size / price_range
   └─ return candle_type, body_size, wick_ratio, body_ratio
```

## متغیرهای کلیدی در هر مرحله:

### 1️⃣ ورودی پنل
```javascript
// متغیرهای ورودی
const formData = {
  data: "[{...}, {...}, ...]",  // JSON string
  strategy_name: "orderblock",
  strategy_mode: "bullish",
  settings: "{}"  // JSON string
}

// متغیرهای پردازش شده
const requestData = {
  data: [{...}, {...}, ...],    // Parsed JSON array
  strategy_name: "orderblock",
  strategy_mode: "bullish",
  settings: {...}               // Parsed JSON object
}
```

### 2️⃣ پردازش API
```python
# متغیرهای دریافتی
payload = {
    "data": [...],              # List of OHLC records
    "strategy_name": "orderblock",
    "strategy_mode": "bullish",
    "settings": {...}
}

# متغیرهای پردازش شده
data_records = payload.get('data')      # List
strategy_name = payload.get('strategy_name')  # String
strategy_mode = payload.get('strategy_mode')  # String
settings = payload.get('settings')      # Dict
df = pd.DataFrame(data_records)         # DataFrame
strategy = engine.load_strategy(strategy_name)  # Strategy instance
```

### 3️⃣ تحلیل استراتژی
```python
# متغیرهای ورودی
df = pd.DataFrame(...)          # OHLCV DataFrame
mode = "bullish"               # String
settings = {...}               # Dict

# متغیرهای پردازش شده
moves = detect_moves(df)        # List[MoveAnalysis]
bullish_blocks = []            # List[OrderBlock]
bearish_blocks = []            # List[OrderBlock]

# برای هر حرکت
for move in moves:
    candle_type, body_size, wick_ratio = analyze_candle(candle)
    volume_confirmed = analyze_volume_confirmation(...)
    strength = calculate_strength(...)
```

### 4️⃣ نتایج نهایی
```python
# ساختار OrderBlock
class OrderBlock:
    index: int                  # Index in DataFrame
    high: float                 # High price
    low: float                  # Low price
    open: float                 # Open price
    close: float                # Close price
    volume: float               # Volume
    block_type: OrderBlockType  # BULLISH/BEARISH
    strength: float             # 0.0 to 1.0
    timestamp: pd.Timestamp     # Date/time
    move_strength: float        # Strength of the move
    volume_confirmation: bool   # Volume confirmed
    price_range: float          # High - Low
    body_size: float            # abs(Close - Open)
    wick_ratio: float           # Wick to body ratio
    candle_type: CandleType     # BULLISH/BEARISH/DOJI

# ساختار OrderBlockResult
class OrderBlockResult:
    bullish_blocks: List[OrderBlock]   # Bullish order blocks
    bearish_blocks: List[OrderBlock]   # Bearish order blocks
    total_blocks: int                  # Total count
    strong_blocks: int                 # Strong blocks count
```

### 5️⃣ پاسخ JSON
```json
{
  "status": "success",
  "strategy": {
    "name": "OrderBlock Strategy",
    "version": "1.0.0",
    "mode": "bullish"
  },
  "analysis": {
    "bullish_blocks": [
      {
        "index": 15,
        "high": 1865.5,
        "low": 1860.2,
        "open": 1861.0,
        "close": 1864.8,
        "volume": 1000000,
        "block_type": "bullish",
        "strength": 0.85,
        "timestamp": "2023-01-15T10:30:00",
        "move_strength": 0.92,
        "volume_confirmation": true,
        "price_range": 5.3,
        "body_size": 3.8,
        "wick_ratio": 0.28,
        "candle_type": "bullish"
      }
    ],
    "bearish_blocks": [...],
    "total_blocks": 15,
    "strong_blocks": 8
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## فایل‌های کلیدی و متدهای اصلی:

| فایل | متد اصلی | متغیرهای ورودی | متغیرهای خروجی |
|------|----------|----------------|----------------|
| `panel/script.js` | `handleAnalysisSubmit()` | `event` | `requestData` |
| `api_service.py` | `analyze_data()` | `payload` | `JSON Response` |
| `strategy_registry.py` | `load_strategy()` | `strategy_name` | `Strategy instance` |
| `orderblock_strategy.py` | `analyze()` | `df, mode, settings` | `OrderBlockResult` |
| `strategy_engine.py` | `execute_analysis()` | `data, strategy_name, mode` | `Analysis results` | 