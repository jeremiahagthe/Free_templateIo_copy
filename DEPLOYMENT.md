# 🚀 Deployment Guide

Complete guide to deploy your Carousel Generator to Netlify.

## Prerequisites

- GitHub account
- Netlify account (free tier)
- Node.js 18+ installed locally (for testing)

## Method 1: Deploy via GitHub (Recommended)

### Step 1: Push to GitHub

```bash
# Initialize git repository (if not already done)
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit: Carousel Generator"

# Create repo on GitHub and push
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
git branch -M main
git push -u origin main
```

### Step 2: Connect to Netlify

1. Go to [Netlify](https://app.netlify.com/)
2. Click **"Add new site"** → **"Import an existing project"**
3. Choose **GitHub** and authorize Netlify
4. Select your repository
5. Configure build settings:
   - **Build command:** Leave empty (not needed for serverless functions)
   - **Publish directory:** `public`
   - **Functions directory:** `netlify/functions`
6. Click **"Deploy site"**

### Step 3: Verify Deployment

Your site will be live at: `https://random-name-12345.netlify.app`

Test the API:
```bash
curl -X POST https://YOUR-SITE.netlify.app/.netlify/functions/carousel \
  -H "Content-Type: application/json" \
  -d '{
    "backgrounds": ["https://picsum.photos/1080/1080"],
    "slides": [{"title": "Test", "subtitle": "Testing deployment"}]
  }'
```

### Step 4: Custom Domain (Optional)

1. In Netlify dashboard → **Domain settings**
2. Click **"Add custom domain"**
3. Follow DNS configuration instructions

---

## Method 2: Deploy via Netlify CLI

### Step 1: Install Netlify CLI

```bash
npm install -g netlify-cli
```

### Step 2: Login to Netlify

```bash
netlify login
```

### Step 3: Deploy

```bash
# Initialize Netlify project
netlify init

# Deploy to production
netlify deploy --prod
```

### Step 4: Follow prompts

- **Create a new site or use existing:** Choose "Create a new site"
- **Team:** Select your team
- **Site name:** Choose a unique name (or leave blank for random)
- **Publish directory:** `public`

---

## Method 3: One-Click Deploy

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/jeremiahagthe/Free_templateIo_copy)

1. Click the button above
2. Authorize GitHub
3. Choose repository name
4. Click "Deploy site"

---

## Post-Deployment Configuration

### 1. Test the Web UI

Visit: `https://YOUR-SITE.netlify.app`

- Add 2 background URLs
- Add 2 slides
- Click "Generate Carousel"
- Verify images are generated

### 2. Test the API Directly

Visit: `https://YOUR-SITE.netlify.app/test.html`

- Click "Test 1: Basic Carousel"
- Verify response in results section
- Check generated images appear

### 3. Configure Environment Variables (Optional)

If you need Google Drive integration:

1. Go to Netlify dashboard → **Site settings** → **Environment variables**
2. Add variables from `.env.example`:
   ```
   NODE_ENV=production
   GOOGLE_CLIENT_ID=your-client-id
   GOOGLE_CLIENT_SECRET=your-secret
   ```
3. Redeploy the site

### 4. Enable Functions Logs

1. Go to **Functions** tab in Netlify dashboard
2. Click on `carousel` function
3. View real-time logs for debugging

---

## Performance Optimization

### 1. Enable Edge Functions (Optional)

For faster response times globally:

1. Upgrade to Netlify Pro ($19/month)
2. Functions automatically deploy to edge locations

### 2. Increase Function Timeout

Default: 10 seconds (configured in `netlify.toml`)

For larger carousels, increase timeout:

```toml
[functions.carousel]
  timeout = 20
```

### 3. Monitor Usage

**Netlify Dashboard → Analytics:**
- Function invocations
- Bandwidth usage
- Build minutes

**Free Tier Limits:**
- 125K function requests/month
- 100 hours function runtime/month
- 100GB bandwidth/month

**Estimated capacity:**
- ~100-300 carousels per day (depending on size)
- ~3,000-9,000 carousels per month

---

## Troubleshooting

### Build Fails

**Check build logs:**
1. Netlify dashboard → **Deploys**
2. Click failed deploy
3. View logs

**Common issues:**
- Missing `package.json` dependencies
- Node version mismatch (requires 18+)
- Incorrect file paths

**Solution:**
```bash
# Verify locally first
npm install
netlify dev

# Test function locally
curl -X POST http://localhost:8888/.netlify/functions/carousel \
  -H "Content-Type: application/json" \
  -d '{"backgrounds":["https://picsum.photos/1080/1080"],"slides":[{"title":"Test","subtitle":"Local"}]}'
```

### Function Timeout

**Error:** "Task timed out after 10 seconds"

**Solutions:**
1. Reduce number of slides
2. Use smaller image dimensions
3. Increase timeout in `netlify.toml`

### Rate Limit Issues

**Error:** "Rate limit exceeded"

**Solutions:**
1. Wait 1 minute between requests
2. Implement queuing in your n8n workflow
3. Deploy multiple instances

### Images Not Generating

**Check function logs:**
1. Netlify dashboard → **Functions** → `carousel`
2. View recent logs
3. Look for error messages

**Common issues:**
- Invalid image URLs
- Image download timeout
- Sharp processing error

### CORS Errors

If calling from a different domain:

**Verify CORS headers in `netlify.toml`:**
```toml
[[headers]]
  for = "/*"
  [headers.values]
    Access-Control-Allow-Origin = "*"
```

---

## Updating the Deployment

### Via GitHub (Automatic)

```bash
# Make changes locally
# Test changes
npm run dev

# Commit and push
git add .
git commit -m "feat: Add new feature"
git push

# Netlify auto-deploys on push to main
```

### Via CLI (Manual)

```bash
# Deploy to production
netlify deploy --prod

# Deploy to preview (test before production)
netlify deploy
```

---

## Monitoring & Maintenance

### 1. Set Up Alerts

**Netlify Dashboard → Site settings → Notifications:**
- Deploy failed
- Deploy succeeded
- Form submissions (if enabled)

### 2. Monitor Error Rate

**Check function logs weekly:**
1. Look for repeated errors
2. Check rate limit hits
3. Monitor generation time

### 3. Update Dependencies

**Monthly maintenance:**
```bash
# Check for updates
npm outdated

# Update dependencies
npm update

# Test locally
npm run dev

# Deploy
git add package.json package-lock.json
git commit -m "chore: Update dependencies"
git push
```

---

## Scaling Beyond Free Tier

### When to Upgrade

**Consider Netlify Pro ($19/month) if you need:**
- More than 125K requests/month
- Faster function execution (more CPU/memory)
- Team collaboration features
- Priority support

### Alternative: Multiple Free Instances

Deploy to multiple Netlify accounts:
- carousel-gen-1.netlify.app
- carousel-gen-2.netlify.app
- carousel-gen-3.netlify.app

Load balance in n8n:
```javascript
const sites = [
  'https://carousel-gen-1.netlify.app',
  'https://carousel-gen-2.netlify.app',
  'https://carousel-gen-3.netlify.app'
];
const randomSite = sites[Math.floor(Math.random() * sites.length)];
// Use randomSite in HTTP request
```

---

## Security Best Practices

### 1. Environment Variables

**Never commit sensitive data:**
- `.env` is in `.gitignore`
- Use Netlify environment variables for production
- Rotate tokens regularly

### 2. Rate Limiting

**Current implementation:**
- 10 requests/minute per IP
- Configured in `carousel.js`

**Increase for authenticated users:**
```javascript
// In carousel.js
const RATE_LIMIT = req.headers.authorization ? 50 : 10;
```

### 3. Input Validation

**Already implemented:**
- URL validation
- Dimension limits (200-4000px)
- Text sanitization (XML escaping)

### 4. HTTPS Only

Netlify automatically enforces HTTPS for all requests.

---

## Backup & Recovery

### 1. Git Repository

Your code is backed up on GitHub automatically.

### 2. Netlify Snapshots

Netlify keeps deployment history:
- View past deploys
- Rollback to previous version
- Download deployment artifacts

### 3. Regular Exports

**Monthly backup:**
1. Clone repository
2. Export environment variables from Netlify dashboard
3. Store securely (encrypted storage or password manager)

---

## Support

**Documentation:**
- [Netlify Functions Docs](https://docs.netlify.com/functions/overview/)
- [Sharp Documentation](https://sharp.pixelplumbing.com/)

**Community:**
- [Netlify Community Forums](https://answers.netlify.com/)
- [GitHub Issues](https://github.com/jeremiahagthe/Free_templateIo_copy/issues)

**Emergency Rollback:**
```bash
# Revert to previous commit
git revert HEAD
git push

# Or rollback in Netlify dashboard
# Deploys → Select previous deploy → "Publish deploy"
```

---

**Ready to deploy? Start with Method 1 (GitHub) for the best experience!** 🚀
