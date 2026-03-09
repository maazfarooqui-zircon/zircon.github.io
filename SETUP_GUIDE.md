# 🚀 Netlify Migration & Setup Guide
## Protected Stock Dashboard — yourdomain.com/stocks

---

## 📁 YOUR DEPLOYMENT PACKAGE (files in this zip)

```
your-repo/
├── index.html           ← Your EXISTING homepage (replace placeholder)
├── netlify.toml         ← Netlify config (DO NOT EDIT unless instructed)
└── stocks/
    └── index.html       ← Protected stock dashboard (Netlify Identity built in)
```

---

## STEP 1 — Create a Netlify Account (free)

1. Go to **https://netlify.com** and click **Sign Up**
2. Sign up with your **GitHub account** (recommended — links directly to your repo)
3. Confirm your email

---

## STEP 2 — Import Your GitHub Repo into Netlify

1. In Netlify dashboard → click **"Add new site"** → **"Import an existing project"**
2. Choose **GitHub** → authorise Netlify to access your repos
3. Select your repository from the list
4. Build settings:
   - **Build command:** *(leave blank)*
   - **Publish directory:** `.` *(a single dot — means root)*
5. Click **"Deploy site"**

✅ Netlify will give you a temporary URL like `random-name.netlify.app`

---

## STEP 3 — Connect Your Custom Domain

1. In Netlify → **Site settings → Domain management → Add a domain**
2. Type your custom domain (e.g. `yourdomain.com`) → click **Verify**
3. Netlify will show you **DNS records** to add. Two options:

### Option A — If your domain uses Cloudflare / your own DNS:
Add these records at your DNS provider:
```
Type    Name    Value
A       @       75.2.60.5
CNAME   www     your-netlify-name.netlify.app
```

### Option B — Transfer DNS to Netlify (easiest):
Follow Netlify's nameserver instructions. They handle everything including SSL.

4. Wait 5–30 minutes for DNS to propagate
5. Netlify auto-provisions a **free SSL certificate** (HTTPS) ✅

---

## STEP 4 — Enable Netlify Identity

1. In Netlify dashboard → **Site settings → Identity → Enable Identity**
2. Under **Registration preferences** → select **"Invite only"**
   *(This means ONLY people you invite can create accounts — nobody can self-register)*
3. Under **External providers** → you can optionally enable Google login
4. Click **Save**

---

## STEP 5 — Invite Your Users

1. Go to **Identity tab** in your Netlify dashboard
2. Click **"Invite users"**
3. Enter the email address of each person you want to give access
4. They receive an email with a **"Accept the invite"** link
5. They click it, set their own password, and they're in

### To remove a user:
- Identity tab → find the user → click the three dots → **Delete user**

### To see who has accessed the page:
- Identity tab shows all users and their last login time

---

## STEP 6 — Test It

1. Open an **incognito/private browser window**
2. Go to **https://yourdomain.com/stocks**
3. You should see the login screen
4. Enter the credentials of an invited user
5. The dashboard loads ✅

---

## 🔐 How the Security Works

```
User visits yourdomain.com/stocks
         ↓
Netlify serves the HTML file
         ↓
Browser loads Netlify Identity widget (netlify-identity-widget.js)
         ↓
JavaScript checks: is there a valid JWT token in localStorage?
    NO  → Shows "Sign In" button → Opens Netlify Identity modal
    YES → Renders the full stock dashboard immediately
         ↓
On login: Netlify Identity server validates credentials
         ↓
Issues a JWT token (expires in 1 hour by default)
         ↓
Dashboard becomes visible
```

**Important:** The HTML file itself IS accessible without login (GitHub Pages limitation).
Netlify Identity protects the *rendering* of the content, not the raw HTML file.
For a stock screener with public data, this level of protection is appropriate.
If you need file-level protection, use **Netlify Edge Functions** (see Advanced section below).

---

## 🛡️ Advanced: True File-Level Protection (Optional)

If you want the HTML file itself to be inaccessible without login, add this to `netlify.toml`:

```toml
[[edge_functions]]
  path = "/stocks/*"
  function = "auth-guard"
```

Then create `netlify/edge-functions/auth-guard.js`:
```javascript
import { Context } from "netlify:edge";

export default async (request, context) => {
  const token = request.headers.get("Cookie")?.match(/nf_jwt=([^;]+)/)?.[1];
  
  if (!token) {
    return Response.redirect(new URL("/#login", request.url));
  }
  
  return context.next();
};
```

This requires Netlify Pro ($19/month) but gives true server-side protection.

---

## 📊 Free Tier Limits (Netlify)

| Feature          | Free Limit        | Your Need    |
|-----------------|-------------------|--------------|
| Bandwidth        | 100 GB/month      | ~1 GB/month  |
| Build minutes    | 300 min/month     | ~0 (static)  |
| Identity users   | 1,000 users       | ✅ Plenty    |
| Custom domain    | ✅ Included       | ✅           |
| SSL certificate  | ✅ Auto           | ✅           |
| Sites            | Unlimited         | ✅           |

**You will not hit any free limits for this use case.**

---

## 🔄 Updating the Stock Page in Future

Since your repo is connected to Netlify:
1. Edit `stocks/index.html` in your GitHub repo
2. Commit and push
3. Netlify auto-deploys within ~30 seconds ✅

No manual uploads ever needed.

---

## ❓ Troubleshooting

**"Identity not configured" error:**
→ Make sure you enabled Identity in Netlify dashboard (Step 4)

**Login works but dashboard doesn't show:**
→ Clear browser cache and try again in incognito

**Custom domain not working:**
→ DNS propagation can take up to 48 hours. Check at https://dnschecker.org

**User didn't receive invite email:**
→ Check spam folder. Re-send from Identity tab → user → "Send magic link"

**Prices not loading:**
→ Yahoo Finance API has rate limits. Wait 60 seconds and click Refresh.

---

*Guide version 1.0 — Generated March 2026*
*For support: check https://docs.netlify.com/visitor-access/identity/*
