# 🐳 Dashboard Stack - Docker Setup

This directory contains Docker Compose configurations for running the complete Contact Dashboard application stack.

## 📁 Files Overview

```
dashboard-stack/
├── docker-compose.yml        # Full stack development (all services in Docker)
├── docker-compose.dev.yml    # Database only (run backend/frontend locally)
├── docker-compose.prod.yml   # Production-ready configuration
└── README.md                 # This file
```

## 🚀 Quick Start

### Option 1: Full Docker Stack (Recommended for testing)
```bash
# Start all services in Docker
docker compose -f docker-compose.yml up -d --build

# Access the application
# Frontend: http://localhost:3000
# Backend API: http://localhost:1337/api
# Strapi Admin: http://localhost:1337/admin
# Database: localhost:5432
```

### Option 2: Development Mode (Database only)
```bash
# Start only PostgreSQL
docker compose -f docker-compose.dev.yml up -d

# Run backend locally (in another terminal)
cd ../dashboard-backend
yarn install
yarn develop

# Run frontend locally (in another terminal)
cd ../dashboard-client
yarn install
yarn dev
```

### Option 3: Production Deployment
```bash
# Copy environment template
cp ../.env.example ../.env
# Edit .env with your production values

# Deploy with production configuration
docker compose -f docker-compose.prod.yml up -d --build
```

## ⚙️ Configuration Comparison

| Feature | dev.yml | docker-compose.yml | prod.yml |
|---------|---------|-------------------|----------|
| **Services** | DB only | All services | All services |
| **Backend** | Run locally | Docker container | Docker container |
| **Frontend** | Run locally | Docker container | Docker container |
| **Environment** | Hard-coded | Hard-coded | Variables |
| **Security** | Basic | Basic | Enhanced |
| **Use Case** | Fast development | Full stack testing | Production deployment |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Docker Network                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │  Frontend   │  │   Backend   │  │   PostgreSQL    │ │
│  │   (Vue.js)  │  │   (Strapi)  │  │   (Database)    │ │
│  │   Port: 80  │  │  Port: 1337 │  │   Port: 5432    │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

## 🔧 Services Details

### 📊 PostgreSQL Database
- **Image**: `postgres:15-alpine`
- **Container**: `dashboard-postgres`
- **Port**: `5432`
- **Data**: Persisted in `postgres_data` volume

### 🔗 Strapi Backend
- **Build**: From `../dashboard-backend/Dockerfile`
- **Container**: `dashboard-backend`
- **Port**: `1337`
- **API Endpoint**: `/api`
- **Admin Panel**: `/admin`
- **Uploads**: Persisted in `backend_uploads` volume

### 🎨 Vue.js Frontend
- **Build**: From `../dashboard-client/Dockerfile`
- **Container**: `dashboard-frontend`
- **Port**: `80` (mapped to `3000` on host)
- **Server**: Nginx (production build)

## 🔄 Common Commands

### Start Services
```bash
# Development (database only)
docker compose -f docker-compose.dev.yml up -d

# Full stack
docker compose -f docker-compose.yml up -d --build

# Production
docker compose -f docker-compose.prod.yml up -d --build
```

### Stop Services
```bash
# Stop all services
docker compose -f docker-compose.yml down

# Stop and remove volumes (⚠️ deletes data)
docker compose -f docker-compose.yml down -v
```

### View Logs
```bash
# All services
docker compose -f docker-compose.yml logs -f

# Specific service
docker compose -f docker-compose.yml logs -f backend
docker compose -f docker-compose.yml logs -f frontend
docker compose -f docker-compose.yml logs -f postgres
```

### Rebuild Services
```bash
# Rebuild all
docker compose -f docker-compose.yml up -d --build

# Rebuild specific service
docker compose -f docker-compose.yml up -d --build backend
```

## 🔒 Environment Variables (Production)

For production deployment, create a `.env` file in the project root:

```bash
# Database
POSTGRES_DB=dashboard_prod
POSTGRES_USER=dashboard_user
POSTGRES_PASSWORD=your_secure_password

# Strapi
APP_KEYS=key1,key2,key3,key4
API_TOKEN_SALT=your_salt
ADMIN_JWT_SECRET=your_secret
TRANSFER_TOKEN_SALT=your_salt
JWT_SECRET=your_jwt_secret
ENCRYPTION_KEY=your_encryption_key

# Frontend
FRONTEND_API_URL=https://yourdomain.com
```

## 🚨 Troubleshooting

### Port Conflicts
```bash
# Check what's using ports
lsof -i :3000
lsof -i :1337
lsof -i :5432

# Change ports in docker-compose.yml if needed
```

### Database Connection Issues
```bash
# Check if database is healthy
docker compose ps
docker compose logs postgres

# Reset database
docker compose down -v
docker compose up -d
```

### Build Failures
```bash
# Clean rebuild
docker compose down
docker system prune -a
docker compose up -d --build
```

### Container Status
```bash
# Check running containers
docker ps

# Check all containers (including stopped)
docker ps -a

# Inspect specific container
docker inspect dashboard-backend
```

## 📦 Volumes

- **`postgres_data`**: Database files (persistent)
- **`backend_uploads`**: User uploaded files (persistent)

### Backup Data
```bash
# Backup database
docker exec dashboard-postgres pg_dump -U dashboard dashboard > backup.sql

# Backup uploads
docker cp dashboard-backend:/app/public/uploads ./uploads-backup
```

### Restore Data
```bash
# Restore database
docker exec -i dashboard-postgres psql -U dashboard dashboard < backup.sql

# Restore uploads
docker cp ./uploads-backup dashboard-backend:/app/public/uploads
```

## 🌐 Deployment Platforms

### DigitalOcean App Platform
1. Connect GitHub repository
2. Use `docker-compose.prod.yml`
3. Set environment variables in dashboard
4. Deploy

### AWS ECS
1. Push images to ECR
2. Create task definitions
3. Deploy to ECS cluster

### Railway
1. Connect GitHub repository
2. Railway auto-detects Docker Compose
3. Set environment variables
4. Deploy

### Manual VPS Deployment
```bash
# SSH into your server
git clone https://github.com/yourusername/dashboard-integration.git
cd dashboard-integration
cp .env.example .env
# Edit .env with production values
docker compose -f dashboard-stack/docker-compose.prod.yml up -d --build
```

## 🔧 Development Tips

### Hot Reload Development
Use `docker-compose.dev.yml` for faster development with hot reload:
```bash
# Terminal 1: Start database
docker compose -f docker-compose.dev.yml up -d

# Terminal 2: Backend with hot reload
cd ../dashboard-backend && yarn develop

# Terminal 3: Frontend with hot reload
cd ../dashboard-client && yarn dev
```

### Database GUI
Connect to PostgreSQL using tools like:
- **pgAdmin**: http://localhost:5432
- **DBeaver**: localhost:5432
- **TablePlus**: localhost:5432

Connection details:
- Host: `localhost`
- Port: `5432`
- Database: `dashboard`
- Username: `dashboard`
- Password: `123456` (development)

## 📞 Support

If you encounter issues:
1. Check the logs: `docker compose logs -f`
2. Verify all containers are running: `docker ps`
3. Check port availability: `lsof -i :PORT`
4. Try a clean rebuild: `docker compose down && docker compose up -d --build`

## 📄 License

This project is licensed under the MIT License.
