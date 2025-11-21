# EcoBuddy DigitalOcean Deployment Architecture

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    DigitalOcean Cloud                   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │           App Platform                            │  │
│  │                                                    │  │
│  │  ┌─────────────────┐      ┌──────────────────┐   │  │
│  │  │   Frontend      │      │    Backend API   │   │  │
│  │  │   (Static Site) │      │   (Web Service)  │   │  │
│  │  │                 │      │                  │   │  │
│  │  │  • React App    │◄────►│  • Express.js    │   │  │
│  │  │  • Vite Build   │      │  • JWT Auth      │   │  │
│  │  │  • Nginx        │      │  • REST API      │   │  │
│  │  │  • Port 80      │      │  • Port 3001     │   │  │
│  │  │                 │      │                  │   │  │
│  │  │  $3/month       │      │  $5/month        │   │  │
│  │  └─────────────────┘      └──────────────────┘   │  │
│  │          │                         │              │  │
│  │          │                         │              │  │
│  │          │                         ▼              │  │
│  │          │                ┌──────────────────┐   │  │
│  │          │                │   PostgreSQL     │   │  │
│  │          │                │   Managed DB     │   │  │
│  │          │                │                  │   │  │
│  │          │                │  • Database:     │   │  │
│  │          │                │    energyteen    │   │  │
│  │          │                │  • SSL Required  │   │  │
│  │          │                │  • Auto Backups  │   │  │
│  │          │                │                  │   │  │
│  │          │                │  $15/month       │   │  │
│  │          │                └──────────────────┘   │  │
│  │          │                                        │  │
│  └──────────┼────────────────────────────────────────┘  │
│             │                                            │
│             ▼                                            │
│  ┌─────────────────────┐                                │
│  │   Users' Browsers   │                                │
│  │   • HTTPS (SSL)     │                                │
│  │   • Custom Domain   │                                │
│  └─────────────────────┘                                │
│                                                          │
└─────────────────────────────────────────────────────────┘

Total Monthly Cost: ~$23/month
```

---

## 🔄 Request Flow

```
1. User → HTTPS → DigitalOcean Frontend (Nginx)
                   ↓
2. React App renders in browser
                   ↓
3. API calls → HTTPS → DigitalOcean Backend (Express)
                        ↓
4. Backend → SSL → PostgreSQL Managed Database
                   ↓
5. Response ← Backend ← Database
             ↓
6. User ← Frontend ← API Response
```

---

## 🐳 Local Development Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Your Computer                         │
│                                                          │
│  Docker Compose Stack                                    │
│                                                          │
│  ┌─────────────────┐      ┌──────────────────┐          │
│  │   Frontend      │      │    Backend API   │          │
│  │   Container     │      │    Container     │          │
│  │                 │      │                  │          │
│  │  • React + Vite │◄────►│  • Express.js    │          │
│  │  • Nginx        │      │  • Node.js 18    │          │
│  │  • Port 80      │      │  • Port 3001     │          │
│  │                 │      │                  │          │
│  └─────────────────┘      └──────┬───────────┘          │
│          │                       │                       │
│          │                       ▼                       │
│          │              ┌──────────────────┐             │
│          │              │   PostgreSQL     │             │
│          │              │   Container      │             │
│          │              │                  │             │
│          │              │  • Port 5432     │             │
│          │              │  • Volume        │             │
│          │              │  • Persistent    │             │
│          │              │                  │             │
│          │              └──────────────────┘             │
│          │                                               │
│          ▼                                               │
│  ┌─────────────────┐                                     │
│  │  localhost:80   │                                     │
│  │  (Your Browser) │                                     │
│  └─────────────────┘                                     │
│                                                          │
└─────────────────────────────────────────────────────────┘

Command: docker compose up
```

---

## 📦 Component Details

### Frontend Component
- **Technology**: React 18 + Vite 5
- **Web Server**: Nginx Alpine
- **Build**: Multi-stage Docker build
- **Port**: 80 (HTTP) / 443 (HTTPS with DO)
- **Size**: ~50 MB (optimized)
- **Files**:
  - `Dockerfile` - Build and serve configuration
  - `nginx.conf` - Web server configuration
  - `.dockerignore` - Build optimization

**Key Features:**
- SPA routing support
- Gzip compression
- Security headers
- Static asset caching
- Health check endpoint

---

### Backend Component
- **Technology**: Node.js 18 + Express.js
- **Database**: PostgreSQL with pg driver
- **Auth**: JWT tokens
- **Port**: 3001
- **Size**: ~150 MB
- **Files**:
  - `backend/Dockerfile` - Container configuration
  - `backend/.dockerignore` - Build optimization

**Key Features:**
- RESTful API
- Database connection pooling
- Automatic migrations
- Rate limiting
- CORS protection
- Health check endpoint

---

### Database Component
- **Technology**: PostgreSQL 15
- **Type**: Managed Database (DigitalOcean)
- **Connection**: SSL required
- **Backup**: Automatic daily backups
- **Size**: 10 GB storage (Basic plan)

**Tables:**
- users
- user_progress
- ecobuddy
- achievements
- user_achievements
- challenges
- user_challenges
- energy_usage

---

## 🔐 Security Architecture

```
┌──────────────────────────────────────────┐
│          Security Layers                 │
├──────────────────────────────────────────┤
│                                          │
│  1. HTTPS/TLS (Automatic with DO)       │
│     └─ SSL certificates auto-renewed    │
│                                          │
│  2. CORS Protection                      │
│     └─ Frontend URL whitelist           │
│                                          │
│  3. JWT Authentication                   │
│     └─ Secure token-based auth          │
│                                          │
│  4. Database SSL                         │
│     └─ Encrypted connections required   │
│                                          │
│  5. Environment Variables                │
│     └─ Secrets not in code               │
│                                          │
│  6. Rate Limiting                        │
│     └─ API request throttling            │
│                                          │
│  7. Security Headers                     │
│     └─ X-Frame-Options, CSP, etc.        │
│                                          │
│  8. Password Hashing                     │
│     └─ bcrypt with salt                  │
│                                          │
└──────────────────────────────────────────┘
```

---

## 🔄 CI/CD Flow

```
┌─────────────────────────────────────────────────┐
│  Development Flow                               │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. Code Changes                                │
│     └─ Push to GitHub                           │
│                                                 │
│  2. Auto Deploy (if enabled)                    │
│     ├─ DigitalOcean detects push                │
│     ├─ Builds Docker images                     │
│     ├─ Runs health checks                       │
│     └─ Deploys new version                      │
│                                                 │
│  3. Database Migrations                         │
│     └─ Auto-run on backend startup              │
│                                                 │
│  4. Health Checks                               │
│     ├─ Frontend: /health                        │
│     └─ Backend: /api/health                     │
│                                                 │
│  5. Monitoring                                  │
│     ├─ CPU/Memory metrics                       │
│     ├─ Request logs                             │
│     └─ Error tracking                           │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 📊 Scaling Strategy

### Horizontal Scaling
```
Current: 1 instance each
         ↓
Scale: Multiple instances with load balancing
       ↓
Cost: $5/month per additional backend instance
      $3/month per additional frontend instance
```

### Vertical Scaling
```
Current: Basic tier (512 MB RAM)
         ↓
Upgrade: Professional tier (1-8 GB RAM)
         ↓
Cost: $12-$120/month per instance
```

### Database Scaling
```
Current: Basic (1 GB RAM, 10 GB disk)
         ↓
Upgrade: Higher tier (2-16 GB RAM, up to 512 GB disk)
         ↓
Cost: $25-$750/month
```

---

## 🌍 Multi-Region Architecture (Optional)

```
┌─────────────────────────────────────────────────┐
│  Global Deployment                              │
├─────────────────────────────────────────────────┤
│                                                 │
│  Region 1: US East (New York)                   │
│  ├─ Frontend + Backend                          │
│  └─ Database Primary                            │
│                                                 │
│  Region 2: EU (Amsterdam)                       │
│  ├─ Frontend + Backend                          │
│  └─ Database Read Replica                       │
│                                                 │
│  Region 3: Asia (Singapore)                     │
│  ├─ Frontend + Backend                          │
│  └─ Database Read Replica                       │
│                                                 │
│  Load Balancer                                  │
│  └─ Route to nearest region                     │
│                                                 │
└─────────────────────────────────────────────────┘

Additional Cost: ~$50-100/month per region
```

---

## 🔍 Monitoring & Logging

```
┌──────────────────────────────────────────┐
│  DigitalOcean App Platform Insights     │
├──────────────────────────────────────────┤
│                                          │
│  Metrics:                                │
│  • CPU Usage                             │
│  • Memory Usage                          │
│  • Request Count                         │
│  • Response Time                         │
│  • Error Rate                            │
│                                          │
│  Logs:                                   │
│  • Runtime Logs (stdout/stderr)          │
│  • Access Logs (requests)                │
│  • Error Logs (failures)                 │
│                                          │
│  Alerts:                                 │
│  • Email notifications                   │
│  • Slack integration                     │
│  • Custom thresholds                     │
│                                          │
└──────────────────────────────────────────┘
```

---

## 💰 Cost Breakdown by Component

```
Component         Plan          RAM    Cost/Month
─────────────────────────────────────────────────
Database          Basic         1 GB   $15.00
Backend API       Basic         512 MB $5.00
Frontend          Static Site   -      $3.00
SSL Certificates  Auto          -      $0.00 (free)
Bandwidth         Included      -      $0.00
─────────────────────────────────────────────────
TOTAL                                  $23.00/month

Annual Cost: ~$276/year
```

### Included Free:
- SSL/TLS certificates
- 100 GB bandwidth/month
- DDoS protection
- Automatic backups (7 days)
- Monitoring & alerts
- Auto-scaling capabilities

---

## 🚀 Deployment Timeline

```
┌────────────────────────────────────────────────┐
│  Deployment Steps & Time                       │
├────────────────────────────────────────────────┤
│                                                │
│  Pre-Deployment (15 min)                       │
│  ├─ Create DigitalOcean account (5 min)        │
│  ├─ Add payment method (5 min)                 │
│  └─ Generate JWT secret (5 min)                │
│                                                │
│  Database Setup (10 min)                       │
│  ├─ Create database cluster (5 min)            │
│  └─ Configure database (5 min)                 │
│                                                │
│  Backend Deployment (15 min)                   │
│  ├─ Create app (5 min)                         │
│  ├─ Configure environment (5 min)              │
│  └─ Wait for build & deploy (5 min)            │
│                                                │
│  Frontend Deployment (15 min)                  │
│  ├─ Add component (5 min)                      │
│  ├─ Configure build args (5 min)               │
│  └─ Wait for build & deploy (5 min)            │
│                                                │
│  Testing & Verification (10 min)               │
│  ├─ Test API health (2 min)                    │
│  ├─ Test frontend (3 min)                      │
│  └─ Verify features (5 min)                    │
│                                                │
├────────────────────────────────────────────────┤
│  TOTAL TIME: 30-45 minutes                     │
└────────────────────────────────────────────────┘
```

---

## 📚 Architecture Documentation

All architecture details are documented in:
- [DEPLOYMENT_GUIDE_INDEX.md](DEPLOYMENT_GUIDE_INDEX.md) - Complete guide index
- [DIGITALOCEAN_DEPLOYMENT.md](DIGITALOCEAN_DEPLOYMENT.md) - Deployment instructions
- [DOCKER_QUICKSTART.md](DOCKER_QUICKSTART.md) - Local architecture

---

*EcoBuddy v1.5.0 - DigitalOcean Deployment Architecture*  
*Last Updated: November 21, 2024*
