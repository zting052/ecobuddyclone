# Docker Quick Start Guide

This guide helps you run EcoBuddy locally using Docker and Docker Compose.

## Prerequisites

- Docker Desktop installed ([Download](https://www.docker.com/products/docker-desktop))
- Git installed
- 4GB+ RAM available

## Quick Start (5 minutes)

### 1. Clone and Navigate

```bash
git clone https://github.com/zting052/ecobuddyclone.git
cd ecobuddyclone
```

### 2. Start All Services

```bash
docker-compose up
```

This command will:
- ✅ Pull PostgreSQL image
- ✅ Build backend Docker image
- ✅ Build frontend Docker image
- ✅ Start all services
- ✅ Run database migrations automatically

### 3. Access Application

Open your browser to:
- **Frontend**: http://localhost
- **Backend API**: http://localhost:3001
- **Health Check**: http://localhost:3001/api/health

### 4. Stop Services

Press `Ctrl+C` in terminal, then:

```bash
docker-compose down
```

---

## Detailed Commands

### Start in Background

```bash
# Start all services in detached mode
docker-compose up -d

# View logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f backend
```

### Rebuild After Code Changes

```bash
# Rebuild and restart
docker-compose up --build

# Rebuild specific service
docker-compose up --build backend
```

### Database Management

```bash
# Connect to database
docker exec -it ecobuddy-db psql -U ecobuddy -d energyteen

# Run manual migration
curl -X POST http://localhost:3001/api/migrate

# Backup database
docker exec ecobuddy-db pg_dump -U ecobuddy energyteen > backup.sql

# Restore database
cat backup.sql | docker exec -i ecobuddy-db psql -U ecobuddy energyteen
```

### Clean Up

```bash
# Stop and remove containers
docker-compose down

# Remove containers and volumes (deletes database data)
docker-compose down -v

# Remove containers, volumes, and images
docker-compose down -v --rmi all
```

---

## Environment Configuration

### Default Configuration

The `docker-compose.yml` includes default values for local development:

```yaml
Database:
  - Database: energyteen
  - User: ecobuddy
  - Password: ecobuddy_password_change_in_production
  - Port: 5432

Backend:
  - Port: 3001
  - JWT Secret: your-secret-key-here-change-in-production

Frontend:
  - Port: 80
```

### Custom Configuration

Create `.env.docker` file to override defaults:

```env
# Database
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
POSTGRES_DB=mydb

# Backend
JWT_SECRET=my-super-secret-key
BACKEND_PORT=3001

# Frontend
FRONTEND_PORT=8080
```

Then use:

```bash
docker-compose --env-file .env.docker up
```

---

## Troubleshooting

### Port Already in Use

**Error**: `Bind for 0.0.0.0:80 failed: port is already allocated`

**Solution**:
```bash
# Option 1: Stop conflicting service
# On Mac/Linux
sudo lsof -i :80 | grep LISTEN
sudo kill -9 <PID>

# Option 2: Change port in docker-compose.yml
# Edit frontend ports to "8080:80"
```

### Database Connection Failed

**Error**: Backend can't connect to database

**Solution**:
```bash
# Check database is running
docker ps | grep ecobuddy-db

# Check database logs
docker-compose logs db

# Restart database
docker-compose restart db
```

### Build Failures

**Error**: Docker build fails

**Solution**:
```bash
# Clean build cache
docker-compose build --no-cache

# Remove old images
docker system prune -a

# Check Dockerfile syntax
docker build -f backend/Dockerfile backend/
```

### Frontend Shows "Cannot connect"

**Error**: Frontend can't reach backend

**Solution**:
1. Verify backend is running: http://localhost:3001/api/health
2. Check browser console for CORS errors
3. Ensure `VITE_API_URL` build arg is correct
4. Rebuild frontend: `docker-compose up --build frontend`

---

## Development Workflow

### Live Development with Hot Reload

For active development, use local dev servers instead of Docker:

```bash
# Terminal 1: Database only
docker-compose up db

# Terminal 2: Backend with hot reload
cd backend
npm install
npm run dev

# Terminal 3: Frontend with hot reload
npm install
npm run dev
```

Access:
- Frontend: http://localhost:5173 (Vite dev server)
- Backend: http://localhost:3001

### Testing Docker Build Before Deploy

```bash
# Build images locally
docker-compose build

# Test production build
docker-compose up

# Verify all services healthy
docker ps
docker-compose logs
```

---

## Health Checks

All services include health checks:

```bash
# Check all services
docker ps

# Frontend health
curl http://localhost/health

# Backend health
curl http://localhost:3001/api/health

# Database health
docker exec ecobuddy-db pg_isready -U ecobuddy
```

---

## Performance Tips

### 1. Allocate More Resources

In Docker Desktop:
- Settings → Resources
- Increase CPUs to 4+
- Increase Memory to 4GB+

### 2. Enable BuildKit

```bash
# Add to ~/.bash_profile or ~/.zshrc
export DOCKER_BUILDKIT=1
export COMPOSE_DOCKER_CLI_BUILD=1
```

### 3. Use Docker Volumes for node_modules

Add to docker-compose.yml:

```yaml
backend:
  volumes:
    - backend_node_modules:/app/node_modules

frontend:
  volumes:
    - frontend_node_modules:/app/node_modules
```

---

## Docker Commands Reference

```bash
# View running containers
docker ps

# View all containers
docker ps -a

# View images
docker images

# View logs
docker-compose logs [service]

# Execute command in container
docker exec -it ecobuddy-backend sh

# View container stats
docker stats

# Clean up everything
docker system prune -a --volumes
```

---

## Multi-Architecture Builds (M1/M2 Mac)

If using Apple Silicon:

```bash
# Build for multiple platforms
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t ecobuddy-backend ./backend

# Or force AMD64 in docker-compose.yml
services:
  backend:
    platform: linux/amd64
```

---

## Production Deployment

This Docker setup is production-ready and can be deployed to:

1. **DigitalOcean App Platform** - See `DIGITALOCEAN_DEPLOYMENT.md`
2. **AWS ECS** - Use Docker images
3. **Google Cloud Run** - Deploy containers
4. **Azure Container Apps** - Use Docker Compose
5. **Any VPS** - Use docker-compose

---

## Next Steps

- ✅ Local development: Use `npm run dev` for hot reload
- ✅ Testing changes: Use `docker-compose up --build`
- ✅ Production deploy: Follow `DIGITALOCEAN_DEPLOYMENT.md`
- ✅ CI/CD: Integrate Docker builds in GitHub Actions

---

*EcoBuddy v1.5.0 - Docker Edition*
