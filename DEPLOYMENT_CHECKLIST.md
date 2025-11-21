# DigitalOcean Deployment Checklist

Use this checklist to deploy EcoBuddy to DigitalOcean step-by-step.

## ☐ Pre-Deployment (15 minutes)

### Account Setup
- [ ] Create DigitalOcean account at https://www.digitalocean.com
- [ ] Add payment method to account
- [ ] Connect GitHub account to DigitalOcean

### Local Preparation
- [ ] Test locally with Docker:
  ```bash
  docker compose up
  ```
- [ ] Generate JWT secret:
  ```bash
  openssl rand -base64 32
  ```
- [ ] Save JWT secret securely (you'll need it later)

---

## ☐ Database Setup (10 minutes)

- [ ] In DigitalOcean, click **Create** → **Databases**
- [ ] Select **PostgreSQL 15**
- [ ] Choose datacenter region (closest to users)
- [ ] Select **Basic Plan** ($15/mo): 1 GB RAM / 10 GB Disk
- [ ] Name: `ecobuddy-db`
- [ ] Click **Create Database Cluster**
- [ ] Wait 3-5 minutes for provisioning

### Configure Database
- [ ] Go to **Users & Databases** tab
- [ ] Click **Add Database**
- [ ] Database name: `energyteen`
- [ ] Click **Save**

### Save Connection Details
- [ ] Copy connection string from **Connection Details**
- [ ] Format: `postgresql://doadmin:PASSWORD@HOST:PORT/energyteen?sslmode=require`
- [ ] Save this - you'll need it for backend

---

## ☐ Backend Deployment (15 minutes)

### Create App
- [ ] Click **Apps** in DigitalOcean
- [ ] Click **Create App**
- [ ] Choose **GitHub** as source
- [ ] Select repository: `zting052/ecobuddyclone`
- [ ] Branch: `main` (or your branch)
- [ ] Source Directory: `/backend`
- [ ] Enable **Autodeploy** ✅
- [ ] Click **Next**

### Configure Backend
- [ ] Component name: `ecobuddy-backend`
- [ ] Type: **Web Service**
- [ ] Dockerfile: `backend/Dockerfile`
- [ ] HTTP Port: `3001`
- [ ] Instance Size: **Basic - $5/mo** (512 MB RAM)
- [ ] Click **Next**

### Set Environment Variables
Click **Edit** on backend component and add:

- [ ] `DATABASE_URL` = (paste connection string from database)
- [ ] `JWT_SECRET` = (paste generated secret)
- [ ] `NODE_ENV` = `production`
- [ ] `PORT` = `3001`
- [ ] `FRONTEND_URL` = (leave empty for now - will add later)

### Deploy Backend
- [ ] Click **Next** → **Create Resources**
- [ ] Wait 5-10 minutes for deployment
- [ ] Check **Runtime Logs** for errors
- [ ] Copy backend URL (e.g., `https://ecobuddy-backend-xxxxx.ondigitalocean.app`)

### Test Backend
- [ ] Visit: `https://your-backend-url.ondigitalocean.app/api/health`
- [ ] Should return:
  ```json
  {"status":"ok","message":"Energy Teen API is running"}
  ```

---

## ☐ Frontend Deployment (15 minutes)

### Add Frontend Component
- [ ] In same App, click **Create Component** → **Service**
- [ ] Source: Same repository
- [ ] Source Directory: `/` (root)
- [ ] Enable **Autodeploy** ✅

### Configure Frontend
- [ ] Component name: `ecobuddy-frontend`
- [ ] Type: **Web Service**
- [ ] Dockerfile: `Dockerfile` (root)
- [ ] HTTP Port: `80`
- [ ] Instance Size: **Basic - $3/mo**

### Set Build Arguments
- [ ] Click **Edit Build Arguments**
- [ ] Add: `VITE_API_URL` = `https://your-backend-url.ondigitalocean.app/api`
  - ⚠️ Use the backend URL you copied earlier
  - ⚠️ Must include `/api` at the end

### Deploy Frontend
- [ ] Click **Save**
- [ ] Wait 5-10 minutes for deployment
- [ ] Copy frontend URL (e.g., `https://ecobuddy-frontend-xxxxx.ondigitalocean.app`)

---

## ☐ Connect Frontend & Backend (5 minutes)

### Update Backend CORS
- [ ] Go to backend component → **Settings**
- [ ] Edit environment variables
- [ ] Update `FRONTEND_URL` = (paste frontend URL)
- [ ] Click **Save** (backend will restart)

### Verify Connection
- [ ] Visit frontend URL
- [ ] Try to sign up with test account
- [ ] Verify login works
- [ ] Check data persists after logout/login

---

## ☐ Final Verification (5 minutes)

### Test All Features
- [ ] Sign up / Login works
- [ ] Dashboard loads
- [ ] Daily tasks can be completed
- [ ] Analytics page shows data
- [ ] Leaderboard displays
- [ ] EcoBuddy mascot appears
- [ ] Browser notifications prompt appears

### Check Logs
- [ ] Backend logs show no errors
- [ ] Frontend logs show no errors
- [ ] Database connection successful
- [ ] Migrations completed

### Security Check
- [ ] HTTPS enabled (automatic)
- [ ] JWT_SECRET is random and secure
- [ ] DATABASE_URL not exposed
- [ ] FRONTEND_URL set correctly

---

## ☐ Optional: Custom Domain (15 minutes)

### Setup Domain
- [ ] In App Platform, click **Domains**
- [ ] Click **Add Domain**
- [ ] Enter domain: `app.yourdomain.com`
- [ ] Add DNS CNAME record:
  ```
  CNAME  app  your-frontend.ondigitalocean.app
  ```
- [ ] Wait for SSL certificate (5-15 minutes)

### Update Backend
- [ ] Update `FRONTEND_URL` to your custom domain
- [ ] Update frontend `VITE_API_URL` if using API subdomain

---

## ☐ Monitoring Setup (5 minutes)

### Enable Alerts
- [ ] Go to **Monitoring** → **Alerts**
- [ ] Create alert for CPU > 80%
- [ ] Create alert for Memory > 80%
- [ ] Add email notification

### Check Metrics
- [ ] View App **Insights** tab
- [ ] Check CPU usage
- [ ] Check Memory usage
- [ ] Monitor request count

---

## 📝 Important URLs to Save

After deployment, save these URLs:

```
Frontend URL: https://__________________.ondigitalocean.app
Backend URL:  https://__________________.ondigitalocean.app
Health Check: https://__________________.ondigitalocean.app/api/health

Database Host: __________________
Database Port: __________________
Database Name: energyteen
Database User: doadmin
```

---

## 🆘 Troubleshooting

If something doesn't work:

1. **Check Runtime Logs** in DigitalOcean App Platform
2. **Review [DIGITALOCEAN_DEPLOYMENT.md](DIGITALOCEAN_DEPLOYMENT.md)** troubleshooting section
3. **Verify Environment Variables** are set correctly
4. **Test backend health** endpoint
5. **Check browser console** for frontend errors

---

## ✅ Deployment Complete!

Your EcoBuddy app is now live on DigitalOcean!

**Next Steps:**
- Share your app URL
- Monitor usage and costs
- Set up regular backups
- Consider custom domain
- Invite users to test

**Monthly Cost:** ~$23
- Database: $15
- Backend: $5
- Frontend: $3

---

**Need Help?** See [DIGITALOCEAN_DEPLOYMENT.md](DIGITALOCEAN_DEPLOYMENT.md) for detailed instructions.
