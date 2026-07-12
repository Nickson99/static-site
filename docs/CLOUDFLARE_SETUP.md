# Cloudflare Setup Guide

## Overview
This guide provides step-by-step instructions to set up Cloudflare for maximum security protection of your static-site.

## Prerequisites
- Active Cloudflare account (free or paid)
- Your domain registered
- Access to domain registrar to change nameservers

---

## Step 1: Add Site to Cloudflare

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Click "Add a Site"
3. Enter your domain name
4. Select your plan (Free plan recommended for getting started)
5. Click "Continue"

---

## Step 2: Update Nameservers

Cloudflare will provide two nameservers. Update these at your domain registrar:

**Cloudflare Nameservers (Example):**
```
ethan.ns.cloudflare.com
fay.ns.cloudflare.com
```

**How to Update:**
1. Go to your domain registrar (GoDaddy, Namecheap, etc.)
2. Find "Nameservers" or "DNS" settings
3. Replace existing nameservers with Cloudflare's
4. Save changes
5. Wait 24-48 hours for DNS propagation

---

## Step 3: Configure DNS Records

In Cloudflare Dashboard > DNS:

### Add DNS Records

| Type | Name | Content | TTL | Proxy |
|------|------|---------|-----|-------|
| CNAME | @ | yourhost.com | Auto | Proxied 🟠 |
| CNAME | www | yourhost.com | Auto | Proxied 🟠 |
| MX | @ | mail.example.com | Auto | DNS only |
| TXT | @ | (SPF record) | Auto | DNS only |

**Note:** 🟠 Proxied = Cloudflare protection enabled

---

## Step 4: SSL/TLS Configuration

### Settings Path
Cloudflare Dashboard > SSL/TLS

### Configure SSL/TLS Mode

1. **Go to Overview Tab**
2. **Select "Full (Strict)"** (Recommended)
   - Encrypts traffic end-to-end
   - Requires valid SSL on origin
   - Most secure option

```
SSL/TLS Encryption Mode: Full (Strict)
Minimum TLS Version: 1.2
```

### Enable HSTS

1. **Go to Edge Certificates Tab**
2. **Enable "HSTS (HTTP Strict-Transport-Security)"**
3. **Configure:**
   - Max Age: 31536000 (1 year)
   - Include Subdomains: ON
   - Preload: ON

### Auto HTTPS Rewrites
1. **Go to Edge Certificates**
2. **Enable "Automatic HTTPS Rewrites"**
3. This converts `http://` links to `https://`

---

## Step 5: Security Headers Configuration

### Method 1: Using Transform Rules

1. **Go to Rules > Transform Rules**
2. **Create New Rule for HTTP Response Headers**
3. **Add Headers:**

```
Header Name: X-Content-Type-Options
Header Value: nosniff

Header Name: X-Frame-Options
Header Value: DENY

Header Name: X-XSS-Protection
Header Value: 1; mode=block

Header Name: Referrer-Policy
Header Value: strict-origin-when-cross-origin

Header Name: Permissions-Policy
Header Value: geolocation=(), microphone=(), camera=()

Header Name: Content-Security-Policy
Header Value: default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net; img-src 'self' data: https:; font-src 'self' https://fonts.googleapis.com; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
```

---

## Step 6: Web Application Firewall (WAF)

### Enable WAF

1. **Go to Security > WAF**
2. **Select "Manage Rules"**

### Configure Managed Ruleset

1. **Cloudflare Managed Ruleset**
   - Status: ON
   - Action: Challenge

2. **OWASP ModSecurity Core Rule Set**
   - Status: ON
   - Action: Challenge

### Create Custom WAF Rules

1. **Go to Security > WAF > Custom Rules**
2. **Create New Rule:**

```
Rule 1: Block SQL Injection
Expression: (http.request.uri.query contains "' OR '" ) or (http.request.body contains "' OR '")
Action: Block
```

```
Rule 2: Block XSS Attempts
Expression: (http.request.uri.query contains "<script") or (http.request.body contains "<script")
Action: Block
```

```
Rule 3: Protect Admin Panel
Expression: http.request.uri.path contains "/admin" and cf.threat_score > 20
Action: Challenge
```

---

## Step 7: Bot Management

### Enable Bot Management

1. **Go to Security > Bots**
2. **Enable "Super Bot Fight Mode"** (Free tier feature)

### Configure Bot Settings

```
Definitely Automated: Challenge
Likely Automated: Challenge
Verified Bots: Allow
Static Resource Protection: ON
```

### Cloudflare Intelligence

1. **Go to Security > Bots > Bot Analytics**
2. Monitor bot traffic patterns
3. Adjust rules based on insights

---

## Step 8: Rate Limiting

### Create Rate Limiting Rules

1. **Go to Security > Rate limiting**
2. **Create New Rule:**

#### Rule 1: API Rate Limit
```
Match: http.request.uri.path eq "/api"
Threshold: 100 requests per 10 seconds
Action: Challenge
Mitigation Timeout: 1 hour
```

#### Rule 2: Login Attempt Limit
```
Match: http.request.uri.path eq "/login" and http.request.method eq "POST"
Threshold: 5 requests per 5 minutes
Action: Block
Mitigation Timeout: 15 minutes
```

#### Rule 3: Contact Form Limit
```
Match: http.request.uri.path eq "/contact" and http.request.method eq "POST"
Threshold: 2 requests per 10 minutes
Action: Challenge
Mitigation Timeout: 1 hour
```

---

## Step 9: Page Rules for Caching

### Configure Page Rules

1. **Go to Rules > Page Rules**
2. **Create New Rule:**

#### Rule 1: API (No Cache)
```
URL Pattern: example.com/api/*
Settings:
  - Cache Level: Bypass
  - Security Level: High
  - Browser Cache TTL: Respect existing headers
```

#### Rule 2: Static Assets (Full Cache)
```
URL Pattern: example.com/static/*
Settings:
  - Cache Level: Cache Everything
  - Browser Cache TTL: 1 year
  - Edge Cache TTL: 1 month
```

#### Rule 3: HTML (Standard Cache)
```
URL Pattern: example.com/*.html
Settings:
  - Cache Level: Standard
  - Browser Cache TTL: 1 hour
  - Edge Cache TTL: 30 minutes
```

---

## Step 10: DDoS Protection

### Configure DDoS Settings

1. **Go to Security > DDoS**
2. **Set Sensitivity Level: High**
3. **Settings:**
   - Sensitivity Level: High
   - Advanced DDoS: Enabled (Enterprise)
   - HTTP Flood Protection: Enabled
   - TCP Protection: Enabled
   - UDP Protection: Enabled

### Monitoring
- Real-time attack metrics
- Automatic mitigation active
- Review analytics dashboard

---

## Step 11: Firewall Rules

### Create Firewall Rules

1. **Go to Security > WAF > Firewall rules**
2. **Create New Rule:**

```
Rule 1: Block High Threat Scores
Expression: cf.threat_score >= 80
Action: Block
Priority: 1

Rule 2: Challenge Medium Threats
Expression: (cf.threat_score >= 50 and cf.threat_score < 80)
Action: Challenge
Priority: 2

Rule 3: Allow Legitimate Traffic
Expression: cf.threat_score < 50
Action: Allow
Priority: 3
```

---

## Step 12: Caching Configuration

### Cache Settings

1. **Go to Caching > Configuration**
2. **Set Cache Level:**
   - Cache Level: Standard
   - Browser Cache Expiration: 30 minutes

### Cache Rules

1. **Go to Rules > Cache Rules**
2. **Create Rules for Different Content Types:**

```
HTML Files:
- Match: http.request.uri.path ends with ".html"
- Cache Duration: 1 hour

CSS/JS Files:
- Match: http.request.uri.path ends with ".css" or ".js"
- Cache Duration: 30 days

Images:
- Match: http.response.content_type eq "image/*"
- Cache Duration: 90 days

API Endpoints:
- Match: http.request.uri.path contains "/api"
- Cache Duration: Do Not Cache
```

---

## Step 13: Analytics & Monitoring

### Enable Analytics

1. **Go to Analytics & Logs > Overview**
2. **View Key Metrics:**
   - Web Traffic
   - Cache Performance
   - DDoS Attacks
   - Bot Traffic
   - SSL/TLS Performance

### Set Up Alerts

1. **Go to Notifications**
2. **Create Alert:**
   - Alert Type: DDoS Attack Alert
   - Channel: Email
   - Recipients: your@email.com

3. **Create Alert:**
   - Alert Type: High Error Rate
   - Threshold: 5% error rate
   - Channel: Email

4. **Create Alert:**
   - Alert Type: SSL Certificate Expiring
   - Days Before: 30
   - Channel: Email

---

## Step 14: Workers & Scripting (Optional)

### Cloudflare Workers

For advanced security, use Cloudflare Workers:

1. **Go to Workers > Manage Workers**
2. **Create New Worker for:**
   - Additional rate limiting
   - Custom request filtering
   - Bot detection
   - Request logging

### Example Worker for Security

```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  // Add security headers
  const response = await fetch(request)
  
  const newResponse = new Response(response.body, response)
  newResponse.headers.set('X-Custom-Security', 'Enabled')
  newResponse.headers.set('X-Request-Id', crypto.randomUUID())
  
  return newResponse
}
```

---

## Verification Checklist

After setup, verify everything works:

- [ ] Domain resolves correctly
- [ ] HTTPS enabled (green lock in browser)
- [ ] Security headers present (use securityheaders.io)
- [ ] CSP working (check browser console)
- [ ] HSTS enabled (check response headers)
- [ ] Static content cached (fast load times)
- [ ] API bypassing cache
- [ ] WAF active (test with malicious query)
- [ ] Rate limiting working
- [ ] Bot rules active
- [ ] Analytics dashboard populated

---

## Testing Security

### Verify Security Headers

Visit [Security Headers](https://securityheaders.com/) and enter your domain.

Expected Results:
- Grade: A or higher
- All major headers present
- CSP properly configured

### Test SSL/TLS

Visit [SSL Labs](https://www.ssllabs.com/ssltest/) and test your domain.

Expected Results:
- Grade: A or higher
- TLS 1.2 minimum
- No weak ciphers

### Check WAF

Test with intentional attack:
```
https://example.com/?search=' OR '1'='1
```

Should be blocked or challenged.

---

## Troubleshooting

### Issue: DNS not updating
**Solution:**
- Wait 24-48 hours
- Clear browser DNS cache
- Try different DNS: 1.1.1.1 or 8.8.8.8

### Issue: SSL certificate error
**Solution:**
- Wait for certificate issuance (24 hours)
- Check SSL/TLS mode is Full (strict)
- Verify origin has valid certificate

### Issue: High false positive rate
**Solution:**
- Lower WAF sensitivity
- Whitelist legitimate IPs/patterns
- Review WAF logs for patterns

### Issue: Performance issues
**Solution:**
- Adjust cache levels
- Enable Brotli compression
- Use image optimization
- Review analytics for bottlenecks

---

## Maintenance

### Weekly
- [ ] Review security analytics
- [ ] Check WAF rule activity
- [ ] Monitor DDoS metrics
- [ ] Review bot traffic

### Monthly
- [ ] Update WAF rules
- [ ] Review rate limiting effectiveness
- [ ] Check certificate expiration
- [ ] Update DNS records if needed

### Quarterly
- [ ] Full security audit
- [ ] Penetration testing
- [ ] Review all settings
- [ ] Update security policies

---

## Resources

- [Cloudflare Documentation](https://developers.cloudflare.com/)
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/)
- [Security Headers Guide](https://securityheaders.com/)
- [SSL Labs](https://www.ssllabs.com/ssltest/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

---

**Setup Date:** [Enter Date]
**Last Updated:** 2024
**Security Level:** Maximum
**Status:** ✅ Active

