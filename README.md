# Trading Strategy Analysis Platform

A comprehensive trading strategy analysis platform consisting of a Python-based core engine and a web-based analysis panel.

## Architecture

- **Core Engine** - Python backend with FastAPI providing strategy analysis
- **Panel** - Web frontend for strategy configuration and visualization
- **Infrastructure** - InfluxDB, Redis, PostgreSQL, Grafana, Prometheus

## Quick Start with Docker Compose

### Prerequisites

- Docker Desktop or Docker Engine with Docker Compose
- 8GB+ RAM recommended
- 20GB+ disk space

### 1. Clone and Start

```bash
git clone <repository-url>
cd trading-strategy-agent

# Development mode
docker-compose up -d

# Production mode
docker-compose -f docker-compose.prod.yml up -d

# Or use the helper scripts
./scripts/deploy.sh start           # Development
./scripts/deploy.sh start --prod    # Production
```

### 2. Access Services

| Service | URL | Credentials |
|---------|-----|-------------|
| **Trading Panel** | http://localhost | - |
| **Core API** | http://localhost/api | - |
| **Grafana** | http://localhost/grafana | admin/trading123 |
| **Jupyter** | http://localhost:8888 | Token: trading123 |
| **InfluxDB** | http://localhost:8086 | admin/trading123 |
| **Prometheus** | http://localhost:9090 | - |
| **pgAdmin** | http://localhost:5050 | admin@trading.local/trading123 |

### 3. Verify Installation

```bash
# Check all services are running
docker-compose ps

# Check API health
curl http://localhost/api/health

# View logs
docker-compose logs -f core-engine
docker-compose logs -f panel
```

## Service Details

### Core Engine (Port 5002)
- FastAPI-based REST API
- Strategy analysis and backtesting
- Real-time data processing
- Configuration management

### Panel (Port 8080)
- React-based web interface
- Strategy configuration UI
- Real-time analysis visualization
- Results dashboard

### Infrastructure Services

#### InfluxDB (Port 8086)
- Time-series database for market data
- High-performance data storage
- Real-time analytics

#### PostgreSQL with TimescaleDB (Port 5432)
- Relational database with time-series extensions
- Strategy configurations and metadata
- Historical analysis results

#### Redis (Port 6379)
- Caching layer
- Session management
- Real-time data streaming

#### Grafana (Port 3000)
- Monitoring dashboards
- Performance metrics visualization
- Alerting system

#### Prometheus (Port 9090)
- Metrics collection
- System monitoring
- Performance tracking

## Development

### Environment Variables

Create a `.env` file in the root directory:

```env
# API Configuration
API_HOST=0.0.0.0
API_PORT=5002

# Database Configuration
POSTGRES_HOST=timescaledb
POSTGRES_PORT=5432
POSTGRES_DB=market_data
POSTGRES_USER=trader
POSTGRES_PASSWORD=trading123

# InfluxDB Configuration
INFLUXDB_URL=http://influxdb:8086
INFLUXDB_TOKEN=trading-token-123456789
INFLUXDB_ORG=trading_org
INFLUXDB_BUCKET=market_data

# Redis Configuration
REDIS_HOST=redis
REDIS_PORT=6379

# Monitoring
PROMETHEUS_GATEWAY=prometheus:9090
```

### Build from Source

```bash
# Build specific services using unified Dockerfile
./scripts/build.sh core-engine     # Build only core engine
./scripts/build.sh panel           # Build only panel
./scripts/build.sh all             # Build both services

# Build with options
./scripts/build.sh all --no-cache  # Build without cache
./scripts/build.sh all --push      # Build and push to registry

# Traditional Docker Compose approach
docker-compose build core-engine
docker-compose build panel
docker-compose up --build
```

### Development Mode

```bash
# Start with file watching (for development)
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# View real-time logs
docker-compose logs -f core-engine panel

# Access container shell
docker-compose exec core-engine bash
docker-compose exec panel sh
```

## Configuration

### Strategy Configuration

Access the panel at http://localhost and navigate to the "Strategies" tab to:
- Configure trading strategies
- Set risk parameters
- Validate configurations
- Run backtests

### Database Initialization

The databases are automatically initialized with:
- Default schemas and tables
- Sample data for testing
- User accounts and permissions

### Custom Strategies

Place custom strategy files in:
```
core-engine/src/strategies/
```

The system will automatically detect and load new strategies.

## Monitoring

### Health Checks

```bash
# Check service health
curl http://localhost/health
curl http://localhost/api/health

# Detailed service status
docker-compose ps
docker-compose top
```

### Logs

```bash
# View all logs
docker-compose logs

# Follow specific service logs
docker-compose logs -f core-engine
docker-compose logs -f panel
docker-compose logs -f influxdb

# View with timestamps
docker-compose logs -t
```

### Performance Monitoring

Access Grafana dashboards at http://localhost/grafana for:
- System performance metrics
- Database performance
- API response times
- Trading strategy metrics

## Data Management

### Backup

```bash
# Backup all data volumes
docker-compose exec -T timescaledb pg_dump -U trader market_data > backup.sql
docker-compose exec influxdb influx backup /backup

# Copy backup files
docker cp trading_timescaledb:/backup ./db_backup/
```

### Restore

```bash
# Restore PostgreSQL data
docker-compose exec -T timescaledb psql -U trader -d market_data < backup.sql

# Restore InfluxDB data
docker-compose exec influxdb influx restore /backup
```

## Troubleshooting

### Common Issues

#### Port Conflicts
```bash
# Check port usage
netstat -tulpn | grep :8080
netstat -tulpn | grep :5002

# Stop conflicting services
sudo systemctl stop apache2
sudo systemctl stop nginx
```

#### Memory Issues
```bash
# Increase Docker memory limit to 8GB+
# Check current usage
docker stats

# Clean up unused resources
docker system prune -a
```

#### Database Connection Issues
```bash
# Check database containers
docker-compose logs timescaledb
docker-compose logs influxdb

# Verify network connectivity
docker-compose exec core-engine ping timescaledb
docker-compose exec core-engine ping influxdb
```

### Service Restart

```bash
# Restart specific service
docker-compose restart core-engine

# Restart all services
docker-compose restart

# Restart with rebuild
docker-compose up --build
```

### Clean Restart

```bash
# Stop all services
docker-compose down

# Remove volumes (WARNING: deletes all data)
docker-compose down -v

# Remove images
docker-compose down --rmi all

# Clean start
docker-compose up -d
```

## API Documentation

Once running, access the interactive API documentation at:
- Swagger UI: http://localhost/api/docs
- ReDoc: http://localhost/api/redoc

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes and test with Docker Compose
4. Submit a pull request

## License

[MIT License](LICENSE) 