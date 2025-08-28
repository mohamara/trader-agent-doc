# Trading Strategy Panel Docker Setup

This Docker setup implements the functionality of `start_panel.sh` in a containerized environment, running both the core engine API and the frontend panel.

## Files Created

- `Dockerfile.panel` - Multi-stage Dockerfile that builds both core engine and panel
- `docker-compose.panel.yml` - Docker Compose configuration for easy deployment
- `start_panel_docker.sh` - Startup script for Docker deployment
- `PANEL_DOCKER_README.md` - This documentation

## Quick Start

### Option 1: Using the startup script (Recommended)
```bash
# Make the script executable
chmod +x start_panel_docker.sh

# Run the panel
./start_panel_docker.sh
```

### Option 2: Using Docker Compose directly
```bash
# Build and start services
docker-compose -f docker-compose.panel.yml up -d

# View logs
docker-compose -f docker-compose.panel.yml logs -f

# Stop services
docker-compose -f docker-compose.panel.yml down
```

### Option 3: Using Docker directly
```bash
# Build the image
docker build -f Dockerfile.panel -t trading-panel .

# Run the container
docker run -d \
  --name trading-panel \
  -p 5002:5002 \
  -p 8080:8080 \
  -v $(pwd)/core-engine/data:/app/data \
  -v $(pwd)/core-engine/logs:/app/logs \
  -v $(pwd)/core-engine/keys:/app/keys \
  -v $(pwd)/core-engine/store:/app/store \
  -v $(pwd)/core-engine/strategies:/app/core-engine/strategies \
  trading-panel
```

## Services

### Core Engine API
- **Port**: 5002
- **Health Check**: http://localhost:5002/health
- **Documentation**: http://localhost:5002/docs/
- **Description**: Python Flask API service for trading strategy analysis

### Frontend Panel
- **Port**: 8080
- **URL**: http://localhost:8080
- **Description**: HTML/CSS/JavaScript frontend for the trading strategy panel

### Optional Services (in docker-compose.panel.yml)
- **Redis**: Port 6379 (for caching)
- **PostgreSQL**: Port 5432 (for data storage)

## Architecture

The Docker setup uses a multi-stage build approach:

1. **Core Engine Builder**: Installs Python dependencies
2. **Core Engine Production**: Creates optimized core engine image
3. **Panel Builder**: Installs Node.js dependencies
4. **Panel Production**: Creates optimized panel image
5. **Panel Combined**: Final stage that combines both services

## Key Features

### Implements start_panel.sh Functionality
- ✅ Starts core engine API service with CORS enabled
- ✅ Starts frontend server using Node.js http-server
- ✅ Proper signal handling and cleanup
- ✅ Health checks for both services
- ✅ Non-root user execution for security

### Dependencies Properly Installed
- ✅ Python dependencies from `core-engine/requirements.txt`
- ✅ Node.js dependencies from `panel/package.json`
- ✅ Additional production dependencies (Redis, NATS, etc.)
- ✅ System dependencies (curl, ca-certificates, etc.)

### Security Features
- ✅ Non-root user execution
- ✅ Proper file permissions
- ✅ Security hardening (removes unnecessary files)
- ✅ Volume mounting for data persistence

### Production Ready
- ✅ Health checks
- ✅ Logging configuration
- ✅ Environment variable management
- ✅ Graceful shutdown handling
- ✅ Resource optimization

## Volume Mounts

The following directories are mounted for data persistence:

- `./core-engine/data` → `/app/data` - Application data
- `./core-engine/logs` → `/app/logs` - Application logs
- `./core-engine/keys` → `/app/keys` - API keys and credentials
- `./core-engine/store` → `/app/store` - Data storage
- `./core-engine/strategies` → `/app/core-engine/strategies` - Trading strategies

## Environment Variables

- `PYTHONPATH=/app/core-engine/src` - Python module path
- `PYTHONUNBUFFERED=1` - Unbuffered Python output
- `FLASK_ENV=production` - Flask environment

## Troubleshooting

### Service not starting
```bash
# Check container logs
docker-compose -f docker-compose.panel.yml logs trading-panel

# Check individual service logs
docker-compose -f docker-compose.panel.yml logs api-service
docker-compose -f docker-compose.panel.yml logs frontend-server
```

### Permission issues
```bash
# Fix volume permissions
sudo chown -R $USER:$USER core-engine/data core-engine/logs core-engine/keys core-engine/store
```

### Port conflicts
If ports 5002 or 8080 are already in use, modify the ports in `docker-compose.panel.yml`:
```yaml
ports:
  - "5003:5002"  # Change 5002 to 5003
  - "8081:8080"  # Change 8080 to 8081
```

## Development

For development, you can mount the source code as volumes:

```yaml
volumes:
  - ./core-engine/src:/app/core-engine/src
  - ./panel:/app/panel
```

This allows you to make changes to the code without rebuilding the Docker image.

## Production Deployment

For production deployment:

1. Set appropriate environment variables
2. Use external databases (PostgreSQL, Redis)
3. Configure SSL/TLS certificates
4. Set up monitoring and logging
5. Use a reverse proxy (nginx) for load balancing

## Comparison with Original start_panel.sh

| Feature | start_panel.sh | Docker Setup |
|---------|----------------|--------------|
| Python API | ✅ | ✅ |
| Frontend Server | ✅ | ✅ |
| CORS Support | ✅ | ✅ |
| Signal Handling | ✅ | ✅ |
| Health Checks | ❌ | ✅ |
| Containerization | ❌ | ✅ |
| Volume Persistence | ❌ | ✅ |
| Production Ready | ❌ | ✅ |
| Easy Deployment | ❌ | ✅ | 