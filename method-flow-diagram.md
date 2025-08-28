# Method Flow Diagram

This document provides a detailed breakdown of method calls and variable flow through the Trading Strategy Analysis Platform.

## Method Flow from Panel to Results

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

## Key Variables at Each Stage

### 1️⃣ Panel Input
```javascript
// Input variables
const formData = {
  data: "[{...}, {...}, ...]",  // JSON string
  strategy_name: "orderblock",
  strategy_mode: "bullish",
  settings: "{}"  // JSON string
}

// Processed variables
const requestData = {
  data: [{...}, {...}, ...],    // Parsed JSON array
  strategy_name: "orderblock",
  strategy_mode: "bullish",
  settings: {...}               // Parsed JSON object
}
```

### 2️⃣ API Processing
```python
# Received variables
payload = {
    "data": [...],              # List of OHLC records
    "strategy_name": "orderblock",
    "strategy_mode": "bullish",
    "settings": {...}
}

# Processed variables
df = pd.DataFrame(data_records)         # DataFrame
strategy = engine.load_strategy(strategy_name)  # Strategy instance
```

## Key Files and Main Methods

| File | Main Method | Input Variables | Output Variables |
|------|-------------|----------------|------------------|
| `panel/script.js` | `handleAnalysisSubmit()` | `event` | `requestData` |
| `api_service.py` | `analyze_data()` | `payload` | `JSON Response` |
| `strategy_registry.py` | `load_strategy()` | `strategy_name` | `Strategy instance` |
| `orderblock_strategy.py` | `analyze()` | `df, mode, settings` | `OrderBlockResult` |
| `strategy_engine.py` | `execute_analysis()` | `data, strategy_name, mode` | `Analysis results` |

## Technical Implementation Details

### Method Call Chain
1. **Frontend**: `handleAnalysisSubmit()` → `makeAPIRequest()`
2. **API Layer**: `analyze_data()` → `load_strategy()` → `strategy.analyze()`
3. **Strategy Engine**: `detect_order_blocks()` → `detect_moves()` → `analyze_candle()`
4. **Result Assembly**: `OrderBlockResult` → `JSON Response` → `updateAnalysisResults()`

### Data Transformation Flow
- **Raw Input**: Form data and JSON strings
- **API Processing**: JSON parsing and DataFrame conversion
- **Strategy Analysis**: OHLC data analysis and pattern detection
- **Result Generation**: Structured analysis results
- **Frontend Display**: UI updates and visualization
