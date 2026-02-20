# 🚀 Mobile Webapp Deployment Guide

## Quick Start - Deploy to Render

### Step 1: Prepare Your Files

Your mobile webapp includes:
```
mobile-webapp/
├── app.py
├── market_data.py
├── requirements.txt
├── render.yaml
├── static/
│   ├── style_mobile.css
│   └── stock_logos.json
├── templates/
│   └── chat_mobile.html
└── modules/
    └── modules.py
```

### Step 2: Deploy to Render

#### Option A: Via GitHub (Recommended)

1. **Create a GitHub Repository**
   ```bash
   cd mobile-webapp
   git init
   git add .
   git commit -m "Initial commit - Mobile Finance App"
   ```

2. **Push to GitHub**
   ```bash
   git remote add origin https://github.com/yourusername/finassist-mobile.git
   git branch -M main
   git push -u origin main
   ```

3. **Connect to Render**
   - Go to [Render Dashboard](https://dashboard.render.com)
   - Click "New +" → "Web Service"
   - Connect your GitHub repository
   - Select the mobile-webapp repository

4. **Configure Build Settings**
   ```
   Name: finassist-mobile
   Environment: Python 3
   Build Command: pip install -r requirements.txt
   Start Command: gunicorn app:app
   ```

5. **Deploy**
   - Click "Create Web Service"
   - Wait for deployment (usually 2-3 minutes)
   - Your app will be live at `https://finassist-mobile.onrender.com`

#### Option B: Direct Git Deploy

1. **From Render Dashboard**
   - Click "New +" → "Web Service"
   - Select "Public Git repository"
   - Enter your repo URL

2. **Configure as above** and deploy

### Step 3: Configure Custom Domain (Optional)

1. **In Render Dashboard**
   - Go to your service
   - Click "Settings" → "Custom Domain"
   - Add your domain: `m.yoursite.com`

2. **Update DNS**
   - Add CNAME record:
     ```
     Type: CNAME
     Name: m
     Value: finassist-mobile.onrender.com
     ```

3. **Enable HTTPS**
   - Render automatically provisions SSL certificates
   - Your site will be accessible at `https://m.yoursite.com`

## Environment Variables

No environment variables are required for basic deployment. The app works out of the box.

### Optional: API Keys (if you add premium features)

If you want to add premium data sources:

```bash
# News API (for enhanced news)
NEWS_API_KEY=your_key_here

# Currency API (for forex)
CURRENCY_API_KEY=your_key_here
```

Set these in Render:
- Go to service → Settings → Environment
- Add each variable

## Performance Optimization

### 1. Enable Caching

Add to your Render configuration:
```yaml
# render.yaml
services:
  - type: web
    name: finassist-mobile
    env: python
    buildCommand: pip install -r requirements.txt
    startCommand: gunicorn app:app --workers 2 --threads 4
```

### 2. Use CDN for Static Assets

If serving many users, consider:
- Cloudflare (free tier)
- AWS CloudFront
- Render's built-in CDN

### 3. Database (if needed)

For storing user preferences or watchlists:
```bash
# Add PostgreSQL
render database create finassist-mobile-db
```

## Monitoring

### 1. Render Dashboard
- View logs in real-time
- Monitor CPU/Memory usage
- Check response times

### 2. Set Up Alerts
- Go to Settings → Notifications
- Add email for downtime alerts
- Configure Slack webhook (optional)

### 3. Custom Monitoring

Add to requirements.txt:
```
sentry-sdk[flask]
```

Add to app.py:
```python
import sentry_sdk
from sentry_sdk.integrations.flask import FlaskIntegration

sentry_sdk.init(
    dsn="your-sentry-dsn",
    integrations=[FlaskIntegration()],
)
```

## Scaling

### Free Tier
- 750 hours/month free
- Spins down after 15 min of inactivity
- Good for personal use / testing

### Paid Tiers
- Starter ($7/month): Always on, 0.5GB RAM
- Standard ($25/month): 2GB RAM, better performance
- Pro ($85/month): 4GB RAM, auto-scaling

Upgrade in Settings → Instance Type

## Troubleshooting

### Issue: App won't start

**Check:**
1. Requirements.txt has all dependencies
2. Gunicorn is in requirements.txt
3. Start command is correct: `gunicorn app:app`

**View Logs:**
```bash
# In Render dashboard, click "Logs"
# Or use Render CLI:
render logs finassist-mobile
```

### Issue: 502 Bad Gateway

**Causes:**
- App crashed
- Taking too long to start
- Port binding issue

**Fix:**
```python
# In app.py, ensure:
if __name__ == "__main__":
    app.run(debug=False)  # Not debug=True
```

### Issue: Slow performance

**Optimize:**
1. Add more workers:
   ```
   gunicorn app:app --workers 4
   ```

2. Enable response compression:
   ```python
   from flask_compress import Compress
   Compress(app)
   ```

3. Cache market data:
   ```python
   from functools import lru_cache
   
   @lru_cache(maxsize=128)
   def get_market_data():
       # Your code
   ```

### Issue: Memory usage high

**Solutions:**
1. Reduce worker count
2. Implement data pagination
3. Clear cache periodically
4. Upgrade to larger instance

## Testing Before Production

### 1. Local Testing
```bash
# Test locally
python app.py

# Test with gunicorn
gunicorn app:app --bind 0.0.0.0:5000
```

### 2. Mobile Testing
```bash
# Get your local IP
ifconfig | grep inet

# Access from phone
http://192.168.1.x:5000
```

### 3. Load Testing
```bash
# Install apache bench
apt-get install apache2-utils

# Test 1000 requests, 10 concurrent
ab -n 1000 -c 10 http://localhost:5000/
```

## Maintenance

### Regular Updates

1. **Update Dependencies**
   ```bash
   pip list --outdated
   pip install --upgrade package-name
   pip freeze > requirements.txt
   git commit -am "Update dependencies"
   git push
   ```

2. **Monitor Logs**
   - Check daily for errors
   - Look for unusual patterns
   - Monitor API rate limits

3. **Backup Configuration**
   ```bash
   # Export Render config
   render config export > render-backup.yaml
   ```

### Security

1. **Update Python Version**
   - Keep Python updated
   - Test before deploying

2. **Scan for Vulnerabilities**
   ```bash
   pip install safety
   safety check
   ```

3. **Rate Limiting**
   ```python
   from flask_limiter import Limiter
   
   limiter = Limiter(app, key_func=get_remote_address)
   
   @app.route("/api/stocks")
   @limiter.limit("10 per minute")
   def stocks():
       pass
   ```

## Going Live Checklist

- [ ] Test all features on mobile device
- [ ] Verify all calculator outputs
- [ ] Check market data updates
- [ ] Test error handling
- [ ] Verify responsive design
- [ ] Test on slow 3G connection
- [ ] Check accessibility
- [ ] Set up monitoring
- [ ] Configure alerts
- [ ] Document API endpoints
- [ ] Create user guide
- [ ] Set up analytics (optional)

## Support

### Render Support
- [Render Docs](https://render.com/docs)
- [Render Discord](https://discord.gg/render)
- Email: support@render.com

### App Issues
- Check logs first
- Review this guide
- Test locally
- Create GitHub issue

## Cost Estimate

### Free Deployment
- Render Free: $0/month
- Custom domain: $10-15/year (optional)
- Total: ~$1/month with domain

### Production Deployment
- Render Starter: $7/month
- Custom domain: $10-15/year
- CDN: $0 (Cloudflare free)
- Total: ~$8/month

### High Traffic
- Render Standard: $25/month
- Database: $7/month (if needed)
- CDN: $5-10/month
- Total: ~$40/month

---

**Need Help?** Create an issue or contact support!
