# DigitalOcean Deployment Guide for EcoBuddy

This guide provides step-by-step instructions to deploy the EcoBuddy application on DigitalOcean using App Platform and Managed PostgreSQL Database.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Cost Estimate](#cost-estimate)
- [Deployment Steps](#deployment-steps)
  - [1. Setup PostgreSQL Database](#1-setup-postgresql-database)
  - [2. Deploy Backend API](#2-deploy-backend-api)
  - [3. Deploy Frontend](#3-deploy-frontend)
- [Environment Variables](#environment-variables)
- [Testing Deployment](#testing-deployment)
- [Troubleshooting](#troubleshooting)
- [Maintenance](#maintenance)

---

## Overview

The EcoBuddy application consists of three components:
1. **PostgreSQL Database** - Managed Database
2. **Backend API** - Node.js/Express application
3. **Frontend** - React/Vite static site

We'll deploy these using DigitalOcean's App Platform and Managed Database services.

---

## Prerequisites

Before you begin, ensure you have:
- [ ] DigitalOcean account (sign up at https://www.digitalocean.com)
- [ ] GitHub account with access to this repository
- [ ] Payment method added to DigitalOcean
- [ ] Basic understanding of environment variables

---

## Cost Estimate

### Monthly Costs (USD):

| Service | Tier | Cost/Month |
|---------|------|------------|
| **PostgreSQL Database** | Basic (1 GB RAM, 10 GB disk) | $15 |
| **Backend API** | Basic (512 MB RAM, 1 vCPU) | $5 |
| **Frontend** | Static Site | $3 |
| **Total** | | **~$23/month** |

**Notes:**
- Free trial credits may be available for new accounts
- Costs scale based on usage and performance needs
- Database backups included in managed database price
- SSL certificates are free with DigitalOcean App Platform

---

## Deployment Steps

### 1. Setup PostgreSQL Database

#### Step 1.1: Create Managed Database

1. Log in to your DigitalOcean account
2. Click **Create** → **Databases**
3. Configure the database:
   - **Database Engine**: PostgreSQL
   - **Version**: 15 (or latest stable)
   - **Datacenter Region**: Choose closest to your users (e.g., NYC, SFO, AMS)
   - **Database Configuration**: 
     - **Basic Plan**: 1 GB RAM / 1 vCPU / 10 GB Disk ($15/mo)
   - **Database Name**: `energyteen`
   - **Choose a name**: `ecobuddy-db` (or your preferred name)

4. Click **Create Database Cluster**

#### Step 1.2: Configure Database

Wait 3-5 minutes for the database to provision. Once ready:

1. Click on your database cluster
2. Go to **Users & Databases** tab
3. The default database `defaultdb` exists, but we'll use `energyteen`
4. Click **Add Database**
   - **Database Name**: `energyteen`
   - Click **Save**

5. Note down the connection details from the **Connection Details** section:
   - Host
   - Port
   - Username (usually `doadmin`)
   - Password
   - Database name (`energyteen`)

6. **Connection String Format**:
   ```
   postgresql://username:password@host:port/database?sslmode=require
   ```

#### Step 1.3: Configure Trusted Sources

1. In your database cluster settings, go to **Settings** tab
2. Under **Trusted Sources**, add:
   - Your IP address (for local testing)
   - Later, you'll add the App Platform app (DigitalOcean Apps are auto-trusted)

---

### 2. Deploy Backend API

#### Step 2.1: Create Backend App

1. Go to **Apps** in DigitalOcean
2. Click **Create App**
3. Choose source:
   - **GitHub**: Connect your GitHub account
   - Select this repository: `zting052/ecobuddyclone`
   - **Branch**: `main` (or your deployment branch)
   - **Source Directory**: `/backend`
   - **Autodeploy**: ✅ Enable (deploys on git push)

4. Click **Next**

#### Step 2.2: Configure Backend Resources

1. DigitalOcean will detect the Dockerfile
2. Configure the component:
   - **Name**: `ecobuddy-backend`
   - **Type**: Web Service
   - **Dockerfile Path**: `backend/Dockerfile`
   - **HTTP Port**: `3001`
   - **Instance Size**: Basic - $5/mo (512 MB RAM / 1 vCPU)
   - **Instance Count**: 1

3. Click **Next**

#### Step 2.3: Set Environment Variables

Click **Edit** next to the backend component and add environment variables:

| Key | Value | Notes |
|-----|-------|-------|
| `DATABASE_URL` | `postgresql://doadmin:PASSWORD@HOST:PORT/energyteen?sslmode=require` | Get from database connection details |
| `JWT_SECRET` | Generate random string | Use: `openssl rand -base64 32` |
| `NODE_ENV` | `production` | Production mode |
| `PORT` | `3001` | Backend port |
| `FRONTEND_URL` | Leave empty for now | Will add after frontend deploy |

**To generate JWT_SECRET:**
```bash
# On Mac/Linux/WSL
openssl rand -base64 32

# Or use any random 32+ character string
```

**Important**: 
- Replace `PASSWORD`, `HOST`, `PORT` with actual values from your database
- The `?sslmode=require` is required for DigitalOcean Managed Databases

4. Click **Save**

#### Step 2.4: Configure Database Connection (Alternative)

Instead of using `DATABASE_URL`, you can use DigitalOcean's database binding:

1. In the backend component settings
2. Scroll to **Resources**
3. Click **Attach Database**
4. Select your `ecobuddy-db` database
5. Choose `energyteen` database
6. This automatically sets `${DATABASE_URL}` environment variable

---

### 3. Deploy Frontend

#### Step 3.1: Create Frontend App

1. In the same App (or create new app), click **Create Component** → **Service**
2. Or start fresh: **Create App** → **GitHub**
3. Configure:
   - **Repository**: `zting052/ecobuddyclone`
   - **Branch**: `main`
   - **Source Directory**: `/` (root)
   - **Autodeploy**: ✅ Enable

#### Step 3.2: Configure Frontend Resources

1. Configure the component:
   - **Name**: `ecobuddy-frontend`
   - **Type**: Web Service
   - **Dockerfile Path**: `Dockerfile` (root)
   - **HTTP Port**: `80`
   - **Instance Size**: Basic - $3/mo (Static site)
   - **Instance Count**: 1

2. Set Build Arguments:
   - Click **Edit Build Arguments**
   - Add build argument:
     - **Key**: `VITE_API_URL`
     - **Value**: `https://your-backend-url.ondigitalocean.app/api`
     
   **Note**: You'll get the backend URL after backend deploys. Pattern is:
   - `https://ecobuddy-backend-xxxxx.ondigitalocean.app/api`
   - Or if using custom domain: `https://api.yourdomain.com/api`

#### Step 3.3: Configure Routes

1. In App settings, check **Routes**
2. Ensure frontend has route `/`
3. Backend should have route `/api` (or separate domain)

---

### 4. Link Frontend and Backend

After both components are deployed:

#### Step 4.1: Get Backend URL

1. Go to your App
2. Click on `ecobuddy-backend` component
3. Copy the URL (e.g., `https://ecobuddy-backend-xxxxx.ondigitalocean.app`)

#### Step 4.2: Update Frontend Build Argument

1. Go to App settings
2. Click `ecobuddy-frontend` component → **Settings**
3. Edit **Build Arguments**:
   - `VITE_API_URL`: `https://ecobuddy-backend-xxxxx.ondigitalocean.app/api`
4. **Save** and trigger a new deployment

#### Step 4.3: Update Backend CORS

1. Go to `ecobuddy-backend` component → **Settings**
2. Edit environment variables
3. Update `FRONTEND_URL`:
   - `https://your-frontend-url.ondigitalocean.app`
   - Or your custom domain
4. **Save** and deployment will restart

---

## Environment Variables

### Backend Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `DATABASE_URL` | ✅ | PostgreSQL connection string | `postgresql://user:pass@host:port/db?sslmode=require` |
| `JWT_SECRET` | ✅ | Secret key for JWT tokens | Random 32+ char string |
| `NODE_ENV` | ✅ | Environment mode | `production` |
| `PORT` | ✅ | Server port | `3001` |
| `FRONTEND_URL` | ✅ | Frontend URL for CORS | `https://app.yourdomain.com` |
| `PG_REQUIRE_SSL` | Optional | Force SSL connection | `true` |

### Frontend Build Arguments

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `VITE_API_URL` | ✅ | Backend API URL | `https://api.yourdomain.com/api` |

---

## Testing Deployment

### 1. Test Backend Health

```bash
curl https://your-backend-url.ondigitalocean.app/api/health
```

Expected response:
```json
{
  "status": "ok",
  "message": "Energy Teen API is running",
  "timestamp": "2024-11-21T..."
}
```

### 2. Test Database Connection

Backend logs should show:
```
✅ Database connection test successful
✅ Database migrations completed successfully
```

### 3. Test Frontend

1. Open `https://your-frontend-url.ondigitalocean.app`
2. Try to sign up with a new account
3. Verify login works
4. Check that data persists after logout/login

### 4. Check Logs

In DigitalOcean App Platform:
1. Go to your App
2. Click on component (backend or frontend)
3. Click **Runtime Logs** tab
4. Check for errors or issues

---

## Custom Domains (Optional)

### Setup Custom Domain for Frontend

1. In App settings, click **Domains**
2. Click **Add Domain**
3. Enter your domain (e.g., `app.yourdomain.com`)
4. Add CNAME record to your DNS:
   ```
   CNAME  app  your-app.ondigitalocean.app
   ```
5. Wait for SSL certificate provisioning (automatic, 5-15 minutes)

### Setup Custom Domain for Backend

1. Add another domain for backend (e.g., `api.yourdomain.com`)
2. Add CNAME record:
   ```
   CNAME  api  your-backend.ondigitalocean.app
   ```
3. Update `FRONTEND_URL` and `VITE_API_URL` accordingly

---

## Troubleshooting

### Database Connection Failed

**Error**: `Connection refused` or `Connection timeout`

**Solutions**:
1. Check database is running in DigitalOcean dashboard
2. Verify `DATABASE_URL` is correct
3. Ensure `?sslmode=require` is in connection string
4. Check **Trusted Sources** includes App Platform
5. Check database cluster status

### CORS Errors

**Error**: `Access-Control-Allow-Origin` errors in browser

**Solutions**:
1. Verify `FRONTEND_URL` is set in backend environment
2. Check frontend URL matches exactly (https/http, trailing slash)
3. For multiple frontends, separate with commas
4. Check backend logs for rejected origins

### Backend Won't Start

**Error**: App fails to deploy or crashes

**Solutions**:
1. Check **Runtime Logs** for errors
2. Verify all environment variables are set
3. Check Dockerfile is in correct location
4. Ensure `PORT` matches HTTP Port setting
5. Verify database migrations run successfully

### Frontend Shows Blank Page

**Error**: White screen or React errors

**Solutions**:
1. Check browser console for errors
2. Verify `VITE_API_URL` is correct
3. Check build logs for build failures
4. Ensure nginx.conf is properly configured
5. Verify React Router routes work

### Build Failures

**Error**: Deployment fails during build

**Solutions**:
1. Check build logs in DigitalOcean
2. Verify Dockerfile syntax
3. Check all dependencies are in package.json
4. Ensure build command succeeds locally
5. Check for out-of-memory issues (upgrade instance)

### Database Migration Fails

**Error**: Tables not created

**Solutions**:
1. Check backend logs for migration errors
2. Manually trigger migration:
   ```bash
   curl -X POST https://your-backend-url.ondigitalocean.app/api/migrate
   ```
3. Check database permissions
4. Verify database name is correct

---

## Maintenance

### Database Backups

DigitalOcean Managed Databases include automatic daily backups:
1. Go to your database cluster
2. Click **Backups** tab
3. Backups are retained for 7 days (Basic plan)
4. Can restore from any backup point

### Monitoring

1. **App Platform Metrics**:
   - Go to App → **Insights** tab
   - View CPU, Memory, Request metrics
   
2. **Database Metrics**:
   - Go to Database → **Metrics** tab
   - View connection count, CPU, disk usage

3. **Alerts**:
   - Set up alerts for high CPU, memory, or errors
   - Go to **Monitoring** → **Alerts**

### Scaling

#### Scale Backend:
1. Go to App → Backend component → **Settings**
2. Change instance size or count
3. Plans: $5 (512MB), $12 (1GB), $24 (2GB)

#### Scale Database:
1. Go to Database → **Settings** → **Resize**
2. Choose larger plan
3. Migration happens with minimal downtime

### Updates

#### Deploy Code Changes:
1. Push to GitHub (if autodeploy enabled)
2. Or manually trigger: App → **Actions** → **Force Rebuild**

#### Update Dependencies:
1. Update `package.json`
2. Commit and push
3. App rebuilds automatically

---

## Security Best Practices

### 1. Environment Variables
- ✅ Never commit `.env` files
- ✅ Use strong JWT_SECRET (32+ characters)
- ✅ Rotate secrets periodically

### 2. Database
- ✅ Use managed database with automatic backups
- ✅ Enable SSL connections (`?sslmode=require`)
- ✅ Limit trusted sources to necessary IPs/apps
- ✅ Use strong passwords

### 3. CORS
- ✅ Set explicit `FRONTEND_URL`
- ✅ Don't use wildcards (`*`) in production
- ✅ Keep list of allowed origins minimal

### 4. HTTPS
- ✅ Always use HTTPS (automatic with DigitalOcean)
- ✅ Enable HSTS headers
- ✅ Use secure cookies

---

## Cost Optimization

### Reduce Costs:

1. **Use Basic Tiers**: Start with smallest instances
2. **Single App**: Combine frontend/backend in one app (saves $3-5/mo)
3. **Development Database**: Use smaller database for dev/staging
4. **Suspend Unused Apps**: Pause apps when not in use
5. **Monitor Usage**: Check metrics to avoid over-provisioning

### Scale When Needed:

- Upgrade when CPU/memory consistently >80%
- Add instances when response time increases
- Upgrade database when connection limits hit

---

## Support

### DigitalOcean Resources:
- **Documentation**: https://docs.digitalocean.com/products/app-platform/
- **Community**: https://www.digitalocean.com/community
- **Support**: Available in DigitalOcean dashboard

### Application Issues:
- Check application logs in App Platform
- Review backend/frontend documentation
- Open issue in GitHub repository

---

## Quick Reference

### Useful Commands

```bash
# Test backend locally with Docker
docker-compose up backend

# Test full stack locally
docker-compose up

# Check backend health
curl https://your-backend.ondigitalocean.app/api/health

# Manual database migration
curl -X POST https://your-backend.ondigitalocean.app/api/migrate

# Generate JWT secret
openssl rand -base64 32
```

### Important URLs

After deployment, bookmark these:
- Frontend: `https://your-app.ondigitalocean.app`
- Backend: `https://your-backend.ondigitalocean.app`
- Backend Health: `https://your-backend.ondigitalocean.app/api/health`
- Database Dashboard: DigitalOcean → Databases → ecobuddy-db

---

## Next Steps

After successful deployment:

1. ✅ Test all features (signup, login, tasks, leaderboard)
2. ✅ Set up custom domain (optional)
3. ✅ Configure monitoring and alerts
4. ✅ Set up backups schedule
5. ✅ Document your specific URLs for team
6. ✅ Consider CDN for better performance (DigitalOcean Spaces)

---

**Deployment completed successfully?** 🎉

Your EcoBuddy app is now live on DigitalOcean! Share your URL and start tracking energy savings!

---

*Last updated: November 2024*
*EcoBuddy v1.5.0*
