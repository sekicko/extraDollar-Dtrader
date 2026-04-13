# Vercel Deployment Guide for Derivatives Trading Platform

## 📋 Pre-Deployment Checklist

Before deploying to Vercel, complete these steps locally:

### 1. Build Locally (Verify Everything Works)

```bash
# Install dependencies
npm run bootstrap

# Build all packages
npm run build:all

# Check for build errors
ls -la packages/core/dist/
```

Expected output:

- `packages/core/dist/` folder with `index.html` and JS/CSS bundles
- No build errors in console

### 2. Verify Build Configuration

Your project is already configured for Vercel with:

- **`vercel.json`** - Standard Vercel config (routes only)
- **`vercel.dr.json`** - Custom build config (what you'll use)

**Choose ONE based on your need:**

#### Option A: `vercel.json` (Recommended - Simpler)

```json
{
    "routes": [{ "handle": "filesystem" }, { "src": "/(.*)", "dest": "/index.html" }]
}
```

✅ Vercel auto-builds from source  
✅ Simpler setup  
✅ Recommended for most projects

#### Option B: `vercel.dr.json` (Pre-built)

```json
{
  "outputDirectory": "packages/core/dist",
  "buildCommand": "echo ✅ Skipping build (pre-built)",
  "installCommand": "echo ✅ Skipping install",
  "routes": [...]
}
```

✅ Uses pre-built files  
✅ Faster deployment  
✅ For CI/CD pipelines

## 🚀 Step-by-Step Vercel Deployment

### Step 1: Push to GitHub

```bash
git add .
git commit -m "chore: ready for Vercel deployment"
git push origin master
```

### Step 2: Connect to Vercel

1. Go to **[vercel.com](https://vercel.com)**
2. Click **"Add New..."** → **"Project"**
3. Select **GitHub** and authorize
4. Find your repo: `dtrader-template`
5. Click **"Import"**

### Step 3: Configure Build Settings

In Vercel dashboard, set:

| Setting              | Value                |
| -------------------- | -------------------- |
| **Framework**        | `Other` (Monorepo)   |
| **Build Command**    | `npm run build:all`  |
| **Output Directory** | `packages/core/dist` |
| **Node Version**     | `20.x`               |
| **Install Command**  | `npm run bootstrap`  |

### Step 4: Configure Environment Variables

Click **"Environment Variables"** and add:

```
CROWDIN_URL=https://crowdin-api.example.com
R2_PROJECT_NAME=derivatives-trader
CROWDIN_BRANCH_NAME=main
NODE_ENV=production
```

**Optional - Add these if using translations:**

```
CROWDIN_API_KEY=your_key_here
R2_ACCESS_KEY_ID=your_key_here
R2_SECRET_ACCESS_KEY=your_key_here
```

### Step 5: Configure Custom Domain

1. Go to **Project Settings** → **Domains**
2. Click **"Add"**
3. Enter: `derivpulse.site`
4. Vercel will show **DNS configuration**
5. Update DNS records at your domain registrar:
    - CNAME: `derivpulse.site` → `cname.vercel.com`
    - (or follow Vercel's specific instructions)

### Step 6: Set up Redirect URLs

On **Deriv API Portal**:

1. Go to **Applications** → Your App (`32LTHOWJyXh0f3E6uTNFP`)
2. Add **Redirect URI**: `https://derivpulse.site/callback`
3. Add **Redirect URI**: `https://derivpulse.site` (base URL)
4. Save

### Step 7: Deploy!

Click **"Deploy"** button in Vercel

Wait for build to complete (~5-10 minutes for first build)

---

## 🔧 Vercel Project Settings (Detailed)

### General Settings

```
Project Name: dtrader-template
Framework: Other
Git Repository: Danielandeche/dtrader-template
Production Branch: master
```

### Build & Development

```
Build Command:    npm run build:all
Output Directory: packages/core/dist
Install Command:  npm run bootstrap
Node.js Version:  20.x
```

### Environment Variables (Production)

```
CROWDIN_URL=https://crowdin-api.example.com
R2_PROJECT_NAME=derivatives-trader
CROWDIN_BRANCH_NAME=main
NODE_ENV=production
```

### Custom Domain

```
Domain: derivpulse.site
SSL/TLS: Automatic (Vercel manages)
```

---

## 📊 Monitoring & Debugging on Vercel

### Check Build Logs

1. Click **"Deployments"** tab
2. Select latest deployment
3. View **"Build Logs"** for errors

### Preview & Production URLs

- **Preview URL**: Auto-generated for each PR
- **Production URL**: `https://derivpulse.site`

### Analytics

- **Vercel Analytics** - Performance metrics
- **Real-time logs** - Function calls, errors

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Build Fails - "Cannot find module '@deriv/trader'"

**Solution:**

```bash
# Ensure all packages build locally first
npm run build:all

# Check dependencies
npm run bootstrap
```

### Issue 2: Translation CDN 404 Error

**Fix in Vercel:**

```
Set: CROWDIN_URL=https://your-crowdin-url
```

Or update `.env`:

```
CROWDIN_URL=https://crowdin-api.example.com
```

### Issue 3: API Connection Fails (WebSocket Errors)

**Check:**

1. Vercel has correct **App ID**: `32LTHOWJyXh0f3E6uTNFP`
2. Deriv API portal has **Redirect URL**: `https://derivpulse.site`
3. `brand.config.json` has correct settings

### Issue 4: CSR (Client-Side Rendering) 404s on Page Refresh

**Already fixed in `vercel.json`:**

```json
{
    "routes": [{ "handle": "filesystem" }, { "src": "/(.*)", "dest": "/index.html" }]
}
```

This ensures all routes fallback to `/index.html` for React Router to handle.

---

## 📈 Post-Deployment Checklist

After deployment, verify:

- [ ] App loads at `https://derivpulse.site`
- [ ] No console errors (F12 → Console tab)
- [ ] Can connect to Deriv API (WebSocket connects)
- [ ] Can log in with Deriv account
- [ ] Trading form loads
- [ ] Can fetch price quotes
- [ ] Live updates work

---

## 🔐 Security Best Practices

1. **Never commit `.env` files** with secrets
2. **Use Vercel Environment Variables** for sensitive data
3. **Enable 2FA** on your Deriv API account
4. **Restrict API permissions** to what's needed
5. **Monitor Vercel logs** for unauthorized access
6. **Use HTTPS only** (Vercel auto-enables)

---

## 📝 Recommended Workflow

### Development

```bash
npm run bootstrap
npm run serve core  # Local testing
```

### Before Each Deploy

```bash
npm run build:all   # Build locally
npm run test:jest   # Run tests
git push origin     # Push to GitHub
```

### Vercel Auto-Deploys

- Automatic deployment on every `master` branch push
- Can be configured in Vercel dashboard

---

## 💡 Advanced Configuration (Optional)

### Custom Vercel Build Function

Create `vercel.json` with functions:

```json
{
    "buildCommand": "npm run build:all",
    "outputDirectory": "packages/core/dist",
    "functions": {
        "api/**/*.js": {
            "memory": 512,
            "maxDuration": 60
        }
    }
}
```

### Using Environment-Specific Configs

For staging vs production:

```bash
# .env.production
NODE_ENV=production
REACT_APP_API_URL=wss://api.deriv.com

# .env.staging
NODE_ENV=development
REACT_APP_API_URL=wss://staging-api.deriv.com
```

Then Vercel uses appropriate env based on branch.

---

## 📞 Support & Resources

- **Vercel Docs:** https://vercel.com/docs
- **Deriv API Docs:** https://api.deriv.com
- **React Deployment:** https://react.dev/learn/start-a-new-react-project#deployment

---

**Happy Deploying! 🚀**
