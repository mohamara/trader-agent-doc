# Visual Flow Diagram

This document provides a visual representation of the complete analysis flow from user input to final results, with detailed technical explanations.

## Complete Visual Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PANEL (Frontend)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📊 Analysis Form                    📝 User Input                          │
│  ┌─────────────────┐            ┌─────────────────────────────────┐        │
│  │ OHLC Data       │            │ • strategy_name: "orderblock"   │        │
│  │ Strategy        │            │ • strategy_mode: "bullish"      │        │
│  │ Analysis Mode   │            │ • settings: {...}              │        │
│  │ Configuration   │            │ • data: [{...}, {...}, ...]     │        │
│  └─────────────────┘            └─────────────────────────────────┘        │
│           │                                    │                           │
│           ▼                                    ▼                           │
│  🔗 handleAnalysisSubmit()           📦 Build requestData                  │
│           │                                    │                           │
│           └──────────────┬─────────────────────┘                           │
│                          ▼                                                 │
│                 🌐 makeAPIRequest("/analyze", "POST")                      │
│                          │                                                 │
└──────────────────────────┼─────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        API SERVICE (Backend)                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📥 /analyze endpoint                                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 📦 Receive JSON payload                                              │   │
│  │ 🔍 Extract parameters:                                               │   │
│  │    • data_records                                                   │   │
│  │    • strategy_name                                                  │   │
│  │    • strategy_mode                                                  │   │
│  │    • settings                                                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                          │                                                 │
│                          ▼                                                 │
│  📚 Strategy Registry                                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 🔍 get_strategy_manifest(strategy_name)                            │   │
│  │ 📦 load_strategy_from_manifest(manifest)                           │   │
│  │ ⚙️ Create strategy instance                                        │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                          │                                                 │
│                          ▼                                                 │
│  📊 Data Processing                                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 🔄 Convert to DataFrame                                            │   │
│  │ 🧹 Preprocessing:                                                  │   │
│  │    • Add volume data                                               │   │
│  │    • Convert timestamps                                            │   │
│  │    • Sort chronologically                                          │   │
│  │ ✅ Validate data integrity                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                          │                                                 │
│                          ▼                                                 │
│  🎯 Execute Strategy Analysis                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ strategy.analyze(df, mode, settings)                               │   │
│  │   │                                                                │   │
│  │   ├─ 🔍 detect_order_blocks()                                     │   │
│  │   │   │                                                            │   │
│  │   │   ├─ 📈 detect_moves()                                        │   │
│  │   │   │   • Identify price movements                               │   │
│  │   │   │   • Calculate movement strength                            │   │
│  │   │   │                                                            │   │
│  │   │   ├─ 📊 For each movement:                                    │   │
│  │   │   │   ├─ 🔍 analyze_candle()                                  │   │
│  │   │   │   │   • Candle type (bullish/bearish/doji)                │   │
│  │   │   │   │   • Body size                                          │   │
│  │   │   │   │   • Wick ratio                                         │   │
│  │   │   │   │                                                        │   │
│  │   │   │   ├─ 📊 analyze_volume()                                  │   │
│  │   │   │   │   • Calculate volume_score                             │   │
│  │   │   │   │   • Volume confirmation                                │   │
│  │   │   │   │                                                        │   │
│  │   │   │   └─ 💪 calculate_strength()                              │   │
│  │   │   │       • price_change_score                                │   │
│  │   │   │       • consistency_score                                 │   │
│  │   │   │       • volume_score                                      │   │
│  │   │   │       • body_ratio                                        │   │
│  │   │   │                                                            │   │
│  │   │   └─ 🔧 Filtering:                                            │   │
│  │   │       • Remove overlapping order blocks                        │   │
│  │   │       • Filter by strength threshold                          │   │
│  │   │       • Limit result count                                    │   │
│  │   │                                                                │   │
│  │   └─ 📋 Build OrderBlockResult                                    │   │
│  │       • bullish_blocks: [...]                                     │   │
│  │       • bearish_blocks: [...]                                     │   │
│  │       • total_blocks: count                                       │   │
│  │       • strong_blocks: count                                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                          │                                                 │
│                          ▼                                                 │
│  📤 Build Final Response                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ {                                                                   │   │
│  │   "status": "success",                                             │   │
│  │   "strategy": {                                                    │   │
│  │     "name": "OrderBlock Strategy",                                 │   │
│  │     "version": "1.0.0",                                            │   │
│  │     "mode": "bullish"                                              │   │
│  │   },                                                               │   │
│  │   "analysis": {                                                    │   │
│  │     "bullish_blocks": [...],                                       │   │
│  │     "bearish_blocks": [...],                                       │   │
│  │     "total_blocks": 15,                                            │   │
│  │     "strong_blocks": 8                                             │   │
│  │   },                                                               │   │
│  │   "timestamp": "2024-01-15T10:30:00Z"                             │   │
│  │ }                                                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                          │                                                 │
└──────────────────────────┼─────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PANEL (Frontend)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📱 Response Processing                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 📥 Receive JSON Response                                            │   │
│  │ 🎨 updateAnalysisResults(result)                                   │   │
│  │ 📊 Display results in UI:                                           │   │
│  │    • Order block count                                             │   │
│  │    • Strength metrics                                              │   │
│  │    • Summary statistics                                            │   │
│  │ 🔘 Enable operation buttons:                                       │   │
│  │    • Download results                                              │   │
│  │    • Generate charts                                               │   │
│  │    • Save configuration                                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Key Stages Summary

### 🔄 Stage 1: Input
- User enters data and configuration in panel
- Form submission triggers analysis

### 🔄 Stage 2: API Processing
- API receives request
- Strategy is loaded
- Data is processed

### 🔄 Stage 3: Analysis
- Price movements are detected
- Order blocks are identified for each movement
- Strength is calculated for each order block

### 🔄 Stage 4: Output
- Results are filtered and sorted
- JSON response is constructed
- Results are displayed in panel

## Key Variables

| Stage | Input Variables | Output Variables |
|-------|----------------|----------------|
| Panel | `data`, `strategy_name`, `strategy_mode`, `settings` | `requestData` |
| API | `payload` | `strategy`, `df` |
| Analysis | `df`, `strategy`, `mode`, `settings` | `OrderBlockResult` |
| Output | `OrderBlockResult` | `JSON Response` |

## Technical Implementation Details

### Frontend Processing
- **Data Validation**: Client-side JSON validation before submission
- **Error Handling**: User-friendly error messages for invalid inputs
- **State Management**: Maintains analysis state for result display

### Backend Processing
- **Strategy Loading**: Dynamic strategy discovery and instantiation
- **Data Validation**: Multi-layer validation (format, schema, business rules)
- **Performance Optimization**: Efficient DataFrame operations and caching

### Data Flow
- **Input**: OHLC data in JSON/CSV format
- **Processing**: Pandas DataFrame operations with custom indicators
- **Output**: Structured analysis results with metadata

## Error Handling & Validation

### Input Validation
- Data format verification (JSON/CSV)
- Required field presence
- Data type validation
- Range and constraint checking

### Strategy Validation
- Strategy existence verification
- Mode compatibility checking
- Dependency validation
- Configuration parameter validation

### Data Processing Validation
- OHLC data integrity
- Minimum data requirements
- Time series continuity
- Volume data availability

## Performance Optimizations

### Caching Strategy
- Strategy instance caching
- Configuration parameter caching
- Result caching for repeated requests

### Data Processing
- Efficient DataFrame operations
- Memory-optimized data structures
- Batch processing for large datasets

### API Response
- Compressed JSON responses
- Pagination for large result sets
- Streaming for real-time data
