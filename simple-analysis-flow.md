# Simple Analysis Flow

This document provides a clear, step-by-step breakdown of how the trading strategy analysis works, from user input to final results.

## Stage 1: Start from Panel
```
📊 User Panel
    ↓
📝 Analysis Form (Data + Strategy + Mode)
    ↓
🔗 Send HTTP POST to /analyze
```

## Stage 2: API Processing
```
🌐 API Service (/analyze)
    ↓
📦 Receive JSON payload
    ↓
🔍 Extract parameters:
   - data: OHLC data
   - strategy_name: Strategy identifier
   - strategy_mode: Analysis mode
   - settings: Configuration
```

## Stage 3: Strategy Loading
```
📚 Strategy Registry
    ↓
📋 get_strategy_manifest()
    ↓
📦 load_strategy_from_manifest()
    ↓
⚙️ Create strategy instance
```

## Stage 4: Data Processing
```
📊 Convert data to DataFrame
    ↓
🧹 Data preprocessing:
   - Add volume (if missing)
   - Convert timestamps
   - Sort chronologically
    ↓
✅ Validate data integrity
```

## Stage 5: Execute Strategy Analysis
```
🎯 strategy.analyze()
    ↓
🔍 detect_order_blocks()
    ↓
📈 detect_moves() - Identify price movements
    ↓
📊 For each movement:
   ├─ analyze_candle() - Candle analysis
   ├─ analyze_volume() - Volume analysis
   └─ calculate_strength() - Strength calculation
```

## Stage 6: Process Results
```
📋 Build OrderBlockResult
    ↓
🔧 Filter results:
   ├─ Remove overlapping order blocks
   ├─ Filter by strength threshold
   └─ Limit result count
    ↓
📊 Build final response
```

## Stage 7: Return to Panel
```
📤 Send JSON Response
    ↓
📱 Panel - Receive response
    ↓
🎨 updateAnalysisResults()
    ↓
📊 Display results in UI
```

---

## Key Variables at Each Stage

### Input
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

### Output
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

## Key Files by Stage

| Stage | File | Main Method |
|-------|------|-------------|
| Panel | `panel/script.js` | `handleAnalysisSubmit()` |
| API | `api_service.py` | `analyze_data()` |
| Strategy | `orderblock_strategy.py` | `analyze()` |
| Registry | `strategy_registry.py` | `load_strategy()` |
| Engine | `strategy_engine.py` | `execute_analysis()` |

## Technical Details

### Data Flow Summary
1. **User Input**: OHLC data + strategy configuration
2. **API Processing**: Request validation and parameter extraction
3. **Strategy Loading**: Dynamic strategy discovery and instantiation
4. **Data Preparation**: DataFrame conversion and preprocessing
5. **Analysis Execution**: Strategy-specific logic and signal generation
6. **Result Processing**: Filtering, sorting, and formatting
7. **Response Delivery**: Structured JSON with analysis results

### Key Components
- **Panel**: User interface for data input and result visualization
- **API Service**: REST endpoint for analysis requests
- **Strategy Registry**: Dynamic strategy management system
- **Strategy Engine**: Core analysis orchestration
- **Data Processor**: OHLC data validation and preparation

### Performance Considerations
- **Early Validation**: Input validation prevents expensive processing failures
- **Caching**: Strategy instances cached after first load
- **Batch Processing**: Efficient DataFrame operations for large datasets
- **Error Handling**: Comprehensive error reporting with context

## Business Value

### For Traders
- **Clear Process**: Step-by-step understanding of analysis flow
- **Transparent Results**: Structured output with detailed metrics
- **Customizable**: Configurable parameters for different market conditions
- **Fast Execution**: Optimized processing for real-time analysis

### For Investors
- **Systematic Approach**: Consistent, repeatable analysis methodology
- **Risk Management**: Built-in validation and error handling
- **Scalable Architecture**: Modular design for strategy expansion
- **Professional Quality**: Institutional-grade analysis and reporting

### For Developers
- **Clear Architecture**: Well-defined component responsibilities
- **Extensible Design**: Easy addition of new strategies
- **Standardized Interface**: Consistent API and data formats
- **Comprehensive Testing**: Built-in validation and error handling
