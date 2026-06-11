# 🚀 LawLexx Complete Deployment Guide
# Vercel + Railway + DNS Setup

## STEP 1: Get Your Vercel Token ⚙️

1. Go to https://vercel.com/account/tokens
2. Click "Create Token"
3. Name: "LawLexx Setup"
4. Expiration: 7 days
5. Copy the token
6. Run this command in PowerShell:

```powershell
$env:VERCEL_TOKEN = "your_token_here"
.\VERCEL_CONFIG.ps1
```

---

## STEP 2: Vercel Environment Variables ✅

The script above sets these automatically:

| Variable | Value |
|----------|-------|
| NEXT_PUBLIC_API_URL | https://api.legalitize.com |
| NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY | pk_live_51SMLWxGq2thkvEHI... |
| NEXT_PUBLIC_SENTRY_DSN | https://examplePublicKey@o0... |
| NEXT_PUBLIC_SENTRY_ENV | production |

**For:** Production, Preview, Development

---

## STEP 3: Railway Environment Variables 🔧

### Manual Steps (since Railway doesn't have easy API):

1. Go to https://railway.app → Your Project
2. Click "Backend" service
3. Go to "Variables" tab
4. Click "Add Variable"
5. Copy-paste from RAILWAY_ENV_VARS.env file (this repo)

### Variables Organized by Priority:

#### 🔴 CRITICAL (Required - app won't start without):
```
DATABASE_URL=postgresql+asyncpg://...
SECRET_KEY=a923a7e0d6fe52a1272f15eaf67681f5de1935b4677a066067b5368f05ef4cf7
GROQ_API_KEY=[get from https://console.groq.com]
```

#### 🟠 IMPORTANT (Features won't work):
```
COURTLISTENER_API_KEY=b3ae1e53785d7eeca5c4d7ceed968fd594bdd8f3
GOVINFO_API_KEY=ZidzVKpwkyLQdNP3Ux2IQwz6Y1Qjohrmg12P3fDc
CONGRESS_API_KEY=MRzbuYhFpFrPQtrYBeEKatE0iZHzLZgtX4G7Qu2B
REGULATIONS_GOV_API_KEY=sH948SIfWGFoTLqLJ9HrbioKWyXxuIvB6XrawrM2
OPENSTATES_API_KEY=183c4140-4ef1-4905-bd8d-f92b904dc297
USPTO_API_KEY=nwarjoslsuhvhvpwgnothzetqzkjsa
SEC_EDGAR_API_KEY=644d6afd4843458024155077785ccbd85d8f94d69bcde43a876be4edaa4a92fa
STRIPE_SECRET_KEY=sk_live_51SMLWxGq2thkvEHI...
RESEND_API_KEY=re_Tco3MSAv_JE47iAc9d29tno22zvYWu3MZ
SENTRY_DSN=https://examplePublicKey@o0...
```

#### 🟢 OPTIONAL (Nice to have):
```
LIVEBLOCKS_SECRET_KEY=sk_dev_placeholder
UPSTASH_REDIS_REST_URL=https://...
```

**After setting all variables:** Click "Deploy" button in Railway

---

## STEP 4: DNS Configuration 🌐

### Get Your Nameservers from Vercel:

1. Go to https://vercel.com/domains
2. Find `legalitize.com`
3. Click it
4. Copy the 4 nameservers

### Update Your Domain Registrar:

**If registered with Namecheap:**
1. Login to https://namecheap.com
2. Go to My Domains
3. Click Manage on `legalitize.com`
4. Go to Nameservers
5. Select "Custom DNS"
6. Paste Vercel's 4 nameservers

**If registered with GoDaddy:**
1. Login to https://godaddy.com
2. My Products → Domains
3. Click `legalitize.com`
4. Manage → Nameservers
5. Paste Vercel's 4 nameservers

**Other registrars:** Similar process - just find Nameservers section

### Add API Subdomain:

Once nameservers are updated, add CNAME record:

| Field | Value |
|-------|-------|
| Type | CNAME |
| Name | api |
| Value | [YOUR_RAILWAY_URL] |
| TTL | 3600 |

Example: If Railway URL is `legalitize-backend-prod.railway.app`:
```
api CNAME legalitize-backend-prod.railway.app
```

---

## STEP 5: Verify Everything Works ✨

Wait 24-48 hours for DNS propagation, then test:

### Test Frontend
```powershell
# Should load the landing page
curl -I https://www.legalitize.com

# Check environment vars are accessible
curl https://www.legalitize.com/__ENV
```

### Test Backend
```powershell
# Should return health check
curl https://api.legalitize.com/api/health

# Should return:
# {"status":"healthy","app":"Legalitize","version":"1.0.0"}
```

### Test Legal Research APIs
```powershell
# Search for a case
curl "https://api.legalitize.com/api/legal-research/cases?query=Smith%20v%20Jones"

# Should return real case data
```

### Test Admin Features
```powershell
# Login as admin
curl -X POST https://api.legalitize.com/api/auth/login `
  -H "Content-Type: application/json" `
  -d '{"username":"admin","password":"password"}'

# List templates
curl "https://api.legalitize.com/api/admin/templates" `
  -H "Authorization: Bearer [TOKEN]"
```

---

## STEP 6: Troubleshooting 🔍

### "DNS not resolving"
```powershell
# Check DNS status
nslookup legalitize.com
nslookup api.legalitize.com

# OR use this tool:
https://dnschecker.org/?q=legalitize.com

# Wait 24-48 hours if showing old nameservers
```

### "Backend returning 404"
1. Check if Railway deployment is "Running" (green status)
2. Check logs at https://railway.app → Backend → Logs
3. Look for startup errors (usually missing DATABASE_URL or GROQ_API_KEY)

### "Frontend can't reach backend"
1. Check NEXT_PUBLIC_API_URL is set correctly
2. Check browser console (F12) for CORS errors
3. Verify backend is running: https://api.legalitize.com/api/health

### "Errors in Sentry/Error tracking not working"
1. Verify SENTRY_DSN is set in both Vercel and Railway
2. Check Sentry project exists at https://sentry.io
3. Wait 5 minutes for first errors to appear

### "Stripe payment failing"
1. Verify STRIPE_SECRET_KEY is correct
2. Verify all 3 STRIPE_*_PRICE_ID variables are set
3. Test in Stripe dashboard (not production)

---

## Timeline 📅

| Action | Time |
|--------|------|
| Set Vercel vars (script) | 5 min |
| Set Railway vars (manual) | 20 min |
| Update DNS nameservers | 5 min |
| Vercel redeploy | 5 min |
| Railway redeploy | 5 min |
| DNS propagation | 24-48 hours ⏳ |
| **Total Setup** | **1 hour** |
| **Full Deployment** | **24-48 hours** |

---

## Final Verification Checklist ✅

After 24-48 hours:

- [ ] https://www.legalitize.com loads
- [ ] https://api.legalitize.com/api/health returns {"status":"healthy"}
- [ ] Frontend login works
- [ ] Pro Se portal accessible
- [ ] Legal research shows real case data
- [ ] Admin panel accessible
- [ ] Deadline tracker functional
- [ ] Errors appear in Sentry dashboard
- [ ] Payments work (test mode)
- [ ] Emails send (check spam folder)

---

## Support Resources

| Issue | Resource |
|-------|----------|
| Vercel Deployment | https://vercel.com/docs/deployments/overview |
| Railway Deployment | https://railway.app/docs |
| DNS Setup | https://dnschecker.org |
| Error Tracking | https://sentry.io/docs/ |
| API Documentation | https://api.legalitize.com/docs |

---

## Next Steps After Deployment

1. **Set up monitoring:**
   - Check Sentry for real errors
   - Monitor Railway logs daily
   - Set up Vercel analytics

2. **Collect legal research data:**
   - Test all 7 legal research APIs
   - Verify rate limits are working
   - Monitor API quota usage

3. **User acceptance testing:**
   - Test Pro Se portal as new user
   - Verify deadline reminders send
   - Test all payment tiers

4. **Go live:**
   - Update DNS to point to production
   - Announce launch
   - Monitor 24/7 for first week

---

**You've got this! 🎉**

*Questions? Check the documentation in C:\LawLexx\ folder*
