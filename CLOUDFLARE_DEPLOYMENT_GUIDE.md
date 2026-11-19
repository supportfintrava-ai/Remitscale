# Cloudflare Pages Production Deployment Guide
**Project:** SEO EMPIRE VAULT  
**Brand:** RemetiSky  
**Production Domain:** [https://remetisky.online](https://remetisky.online)  
**Payment Gateway:** Razorpay Payment Page (`https://rzp.io/rzp/tfKV8tKO`)  

---

## Quick Reference Summary

| Configuration Setting | Required Production Value |
| :--- | :--- |
| **Framework Preset** | `Vite` |
| **Build Command** | `npm run build` |
| **Build Output Directory** | `dist` |
| **Root Directory** | `/` |
| **Node.js Version** | `20` (or `20.18.0`+) |
| **Custom Domain** | `remetisky.online` & `www.remetisky.online` |
| **SPA Routing** | `_redirects` in `dist` (`/* /index.html 200`) |
| **Security & Caching** | Configured in `_headers` in `dist` |
| **Post-Payment Success URL** | `https://remetisky.online/access` |

---

## Step-by-Step Deployment Instructions

### Step 1: Push Local Code to GitHub
Ensure all latest code is committed and pushed to your GitHub repository on branch `main`:
```bash
git add .
git commit -m "chore: prepare production deployment for remetisky.online on Cloudflare Pages"
git push origin main
```

---

### Step 2: Create a Cloudflare Pages Project
1. Log in to your [Cloudflare Dashboard](https://dash.cloudflare.com/).
2. In the left navigation sidebar, select **Workers & Pages** -> **Overview**.
3. Click **Create Application**.
4. Switch to the **Pages** tab and click **Connect to Git**.
5. Authorize GitHub and select your repository (`Backlinks` or your repo name).
6. Click **Begin setup**.

---

### Step 3: Configure Build & Environment Settings
In the **Set up builds and deployments** screen, enter the following exact settings:

- **Project name:** `remetisky` (or `seo-empire-vault`)
- **Production branch:** `main`
- **Framework preset:** Select `Vite`
- **Build command:** `npm run build`
- **Build output directory:** `dist`
- **Root directory:** Leave empty (defaults to `/`)

#### Environment Variables (Advanced):
Click **Environment variables (advanced)** -> **Add variable**:

| Variable Name | Value | Purpose |
| :--- | :--- | :--- |
| `NODE_VERSION` | `20` | Enforces modern Node.js runtime during Cloudflare build |
| `VITE_RAZORPAY_PAYMENT_URL` | `https://rzp.io/rzp/tfKV8tKO` | Razorpay checkout endpoint |

Click **Save and Deploy**. Cloudflare Pages will execute `npm run build` and publish your site to `<project-name>.pages.dev` in ~1 minute.

---

### Step 4: Configure Your Custom Domain (`remetisky.online`)

1. Once the initial build finishes, go to your Pages project in Cloudflare.
2. Click on the **Custom domains** tab.
3. Click **Set up a custom domain**.
4. Enter `remetisky.online` and click **Continue**.
5. **DNS Record Setup:**
   - **If your domain is already on Cloudflare DNS:** Cloudflare will automatically configure the CNAME / apex record for you. Click **Activate domain**.
   - **If your domain is registered elsewhere (e.g. Namecheap, GoDaddy, Hostinger):**
     Add a `CNAME` record in your registrar's DNS management:
     - **Type:** `CNAME`
     - **Name / Host:** `@` (or `remetisky.online`)
     - **Target / Value:** `<your-project-name>.pages.dev`
     - **TTL:** Automatic or 300
6. Repeat the steps to add `www.remetisky.online` pointing to `<your-project-name>.pages.dev` or configure a 301 redirect from `www` to apex.

---

### Step 5: Configure SSL/TLS & HTTPS Rules
1. In Cloudflare, navigate to **SSL/TLS** -> **Overview**.
2. Set SSL/TLS encryption mode to **Full (strict)**.
3. Navigate to **SSL/TLS** -> **Edge Certificates**:
   - Turn **Always Use HTTPS** to **ON**.
   - Turn **Automatic HTTPS Rewrites** to **ON**.
   - Set **Minimum TLS Version** to `TLS 1.2`.

---

### Step 6: Configure Razorpay Post-Payment Redirection
To ensure customers who complete payment are instantly returned to their private access vault:

1. Log in to your [Razorpay Merchant Dashboard](https://dashboard.razorpay.com/).
2. In the left menu, select **Payment Pages**.
3. Locate your payment page: `https://rzp.io/rzp/tfKV8tKO` (or page title `SEO EMPIRE VAULT`).
4. Click **Edit** (pencil icon).
5. Open **Page Settings** (or **Post Payment Settings**).
6. Under **Action after successful payment**:
   - Select **Redirect to your website**.
   - Set **Redirect URL** to:
     ```
     https://remetisky.online/access
     ```
   - Set redirect timer (recommended: `Immediately` or `3 seconds`).
7. Click **Save and Update**.

---

### Step 7: Verification & Quality Assurance Checklist

After deployment and DNS propagation, verify each of the following endpoints:

- [ ] **Homepage:** Navigate to `https://remetisky.online/`. Verify that all components, 3D elements, typography, and images render crisply.
- [ ] **Payment CTAs:** Click any `Smart Only Access` button (in Navbar, Hero, Pricing, Beta Launch, or Exit Modal). Verify it opens `https://rzp.io/rzp/tfKV8tKO`.
- [ ] **Direct Access Route:** Navigate directly to `https://remetisky.online/access`.
  - Without payment tokens: shows authentication request / purchase gateway.
  - With token (`https://remetisky.online/access?payment=success` or `?razorpay_payment_id=pay_test123`): displays the full delivery dashboard with Notion workspace link, Google Sheet link, and downloadable asset formats (CSV, XLSX, Python, ZIP).
- [ ] **SPA Routing Fallback:** Refresh while on `https://remetisky.online/access`. Verify it does NOT return a 404 error (handled by `_redirects`).
- [ ] **Security Headers:** Inspect response headers in DevTools Network tab. Verify `X-Frame-Options: SAMEORIGIN` and `X-Content-Type-Options: nosniff`.
- [ ] **Sitemap & Robots:**
  - Visit `https://remetisky.online/sitemap.xml` (valid XML).
  - Visit `https://remetisky.online/robots.txt` (valid directives).
