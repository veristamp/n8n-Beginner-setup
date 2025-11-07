# n8n Beginner Tutorial – Season 1

**Windows + macOS (All‑in‑One Docker Compose)**
**Zero external ngrok install • Everything runs in Docker • Production‑ready-ish**

> This repo ships with:
>
> * `docker-compose.yml` (n8n + PostgreSQL + ngrok, single command)
> * `example.env` (template you’ll copy to `.env`)
> * `workflows/lead-capture.json` (importable n8n workflow)

---

## Table of Contents

* [What you’ll build](#what-youll-build)
* [Prerequisites](#prerequisites)
* [Quick Start (TL;DR)](#quick-start-tldr)
* [Setup: ngrok domain & token](#setup-ngrok-domain--token)
* [Setup: Google Cloud OAuth (Sheets + Gmail)](#setup-google-cloud-oauth-sheets--gmail)
* [Run on Windows](#run-on-windows)
* [Run on macOS](#run-on-macos)
* [Import the example workflow](#import-the-example-workflow)
* [Beautiful HTML form (optional)](#beautiful-html-form-optional)
* [Environment reference](#environment-reference)
* [Troubleshooting](#troubleshooting)
* [FAQ](#faq)
* [Uninstall / Reset](#uninstall--reset)
* [License](#license)

---

## What you’ll build

A complete **lead‑capture system** running fully in Docker:

* Public **HTTPS domain** via ngrok (e.g., `https://my-cool-n8n.ngrok.io`)
* n8n + PostgreSQL
* Google OAuth credentials (Sheets + Gmail)
* Workflow: Receive Webhook → Clean → Append to Google Sheet → Telegram alert → Gmail welcome email → Thank‑you response

---

## Prerequisites

| Tool               | Windows 10/11              | macOS |
| ------------------ | -------------------------- | ----- |
| Docker Desktop     | ✅ Enable **WSL 2** backend | ✅     |
| Git (optional)     | ✅                          | ✅     |
| VS Code (optional) | ✅                          | ✅     |

* Docker Desktop: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
* Git: [https://git-scm.com/](https://git-scm.com/)
* VS Code: [https://code.visualstudio.com/](https://code.visualstudio.com/)

> Make sure Docker Desktop is running **before** you start.

---

## Quick Start (TL;DR)

1. **Reserve a free ngrok domain** and copy your **Authtoken**.
2. Copy `example.env` → `.env` 
```bash
cp example.env .env
```
and Set these Credentials in the .env file

```env
NGROK_AUTHTOKEN=your_authtoken_here
NGROK_DOMAIN=my-cool-n8n.ngrok.io
```

3. Start services:

```bash
# Windows (PowerShell) or macOS (zsh) or Linux (bash)
docker compose up -d
```

4. Open `https://my-cool-n8n.ngrok.io`(your Ngrok Domain), create the n8n admin user.
5. Do **Google OAuth** setup (Sheets + Gmail) and add credentials in n8n.
6. Import **workflows/lead-capture.json** and press **Execute**.

---

## Setup: ngrok domain & token

1. Go to [https://dashboard.ngrok.com](https://dashboard.ngrok.com) and sign in.
2. **Cloud Edge → Domains → Reserve a domain** (free tier).

   * Example: `my-cool-n8n.ngrok.io`
3. Copy your **Authtoken** from **Get Started → Your Authtoken**.
4. Create `.env` in the repo root using the provided template:

   * Copy `example.env` → `.env` and fill:

```env
NGROK_AUTHTOKEN=your_authtoken_here
NGROK_DOMAIN=my-cool-n8n.ngrok.io
```

> **Never commit** your real `.env`.

---

## Setup: Google Cloud OAuth (Sheets + Gmail)

1. Go to [https://console.cloud.google.com](https://console.cloud.google.com) → **New Project** → name it `n8n-season1`.
2. **APIs & Services → Enable APIs & Services**:

   * Enable **Google Sheets API** and **Gmail API**.
3. **OAuth consent screen** → User type **External** → fill basic info → Save.
4. **Credentials → Create Credentials → OAuth client ID** → **Web application**.
5. Add **Authorized redirect URI** (must match your ngrok domain):

```
https://YOUR_NGROK_DOMAIN/oauth2/callback
```

6. Download the **client credentials JSON**.

> 👉 For deeper details, see the official n8n guide: [n8n Google OAuth Generic Docs](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-generic/)

### Add the credentials in n8n

In n8n → **Credentials**:

* **Google Sheets OAuth2 API** → Enter the **Client ID** and **Client Secret** from your JSON.
* **Gmail OAuth2 API** → Same process: enter **Client ID** and **Client Secret**.

> The redirect/callback URL is automatically set by n8n. You **do not** need to modify or manually enter it. Simply click **Connect**, sign in with Google, and grant access through the browser.

If you see a `redirect_uri_mismatch` error, double‑check that your **Authorized redirect URI** in the Google Console exactly matches your n8n instance URL (`https://YOUR_NGROK_DOMAIN/oauth2/callback`).

---

## Run on Windows

> Tested on Windows 11/10 with Docker Desktop + WSL2.

```powershell
# 1) Clone and open
git clone <YOUR_REPO_URL> n8n-beginner-season1
cd n8n-beginner-season1

# 2) Create env from template
copy example.env .env
# then edit .env and set NGROK_AUTHTOKEN + NGROK_DOMAIN

# 3) Start everything
docker compose up -d

# 4) Check health
docker compose ps
# n8n and postgresql should be healthy within ~30–60s

# 5) Open your domain in the browser
start https://YOUR_NGROK_DOMAIN
```

> First visit: create the n8n admin user.

---

## Run on macOS

> Tested on macOS Sonoma/Ventura with Docker Desktop.

```bash
# 1) Clone and open
git clone <YOUR_REPO_URL> n8n-beginner-season1
cd n8n-beginner-season1

# 2) Create env from template
cp example.env .env
# then edit .env and set NGROK_AUTHTOKEN + NGROK_DOMAIN

# 3) Start everything
docker compose up -d

# 4) Check health
docker compose ps

# 5) Open your domain in the browser
open https://YOUR_NGROK_DOMAIN
```

> First visit: create the n8n admin user.

---

## Import the example workflow

1. Open **n8n** → **Workflows** → **Import from File**.
2. Select `workflows/lead-capture.json` from this repo.
3. Update the **Credentials** nodes to use your saved Google/Telegram/Gmail/Slack creds.
4. Click **Execute** (or activate the workflow).

**Expected flow**

```
[Webhook: HTML Form]
   → [Set: Clean Data]
   → [Google Sheets: Append Row]
   → [Telegram: "New lead!"]
   → [Gmail: Welcome Email]
   → [Respond: Thank You Page]
```

---

## Beautiful HTML form (optional)

Use any static host (or local file) with a form posting to your webhook URL:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Join n8n Waitlist</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body { font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif; background: linear-gradient(135deg, #667eea, #764ba2); height: 100vh; display: flex; align-items: center; justify-content: center; margin:0; }
    .card { background: #fff; padding: 40px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,.25); width: 90%; max-width: 420px; }
    input, button { width: 100%; padding: 12px; margin: 10px 0; border-radius: 8px; border: 1px solid #ddd; font-size: 16px; }
    button { background: #667eea; color: #fff; border: none; cursor: pointer; font-weight: 600; }
    h1 { text-align: center; color: #333; margin: 0 0 10px; }
  </style>
</head>
<body>
  <div class="card">
    <h1>Join n8n Waitlist</h1>
    <form action="https://YOUR_NGROK_DOMAIN/webhook/form" method="POST">
      <input type="text" name="name" placeholder="Your Name" required />
      <input type="email" name="email" placeholder="your@email.com" required />
      <button type="submit">Get Early Access</button>
    </form>
  </div>
</body>
</html>
```

---

## Environment reference

All used by `docker-compose.yml`:

```env
# Required
NGROK_AUTHTOKEN=...
NGROK_DOMAIN=...

# Optional (defaults are set in compose file)
# Timezone
TZ=Asia/Kolkata
GENERIC_TIMEZONE=Asia/Kolkata
```

The compose configuration (already provided) sets:

* `N8N_EDITOR_BASE_URL=https://${NGROK_DOMAIN}`
* `WEBHOOK_URL=https://${NGROK_DOMAIN}`
* `N8N_PROTOCOL=https`
* Postgres connection env for persistence

---

## Troubleshooting

**n8n shows 502/Bad Gateway**

* Wait 30–60 seconds after `up -d`. Check `docker compose ps` and logs:

  ```bash
  docker compose logs -f n8n
  docker compose logs -f ngrok
  ```
* Ensure `NGROK_DOMAIN` is actually reserved in your ngrok dashboard.

**Google OAuth errors**

* If you get `redirect_uri_mismatch`, verify that your **Authorized redirect URI** in Google matches your n8n URL exactly.
* Make sure you entered **Client ID** and **Client Secret** correctly.

**PostgreSQL unhealthy**

* Remove volumes and retry (this wipes data!):

  ```bash
  docker compose down -v
  docker compose up -d
  ```

**Port conflicts**

* If port `5678` is busy, edit the ports section in `docker-compose.yml` (left side only), e.g. `8678:5678`.

**ngrok domain already in use**

* Make sure your reserved domain exactly matches `.env`.

**Telegram/Slack not sending**

* Check tokens and required bot scopes/permissions. Reconnect credentials in n8n.

---

## FAQ

**Q: Can I change the domain later?**
A: Yes—update the reserved domain in ngrok and set `NGROK_DOMAIN` in `.env`, then `docker compose up -d`.

**Q: Is this safe for production?**
A: It’s a great starting point. For real production, add backups, monitoring, secrets management, and consider a dedicated reverse proxy and DB.

**Q: Where does n8n store data?**
A: In the Postgres volume (`postgresql_data`) and n8n app volume (`n8n_data`). Both are Docker named volumes.

**Q: How do I upgrade n8n?**
A: Change the image tag in `docker-compose.yml` (e.g., `n8nio/n8n:<version>`), then `docker compose pull && docker compose up -d`.

---

## Uninstall / Reset

**Stop and remove containers (keep volumes):**

```bash
docker compose down
```

**Remove everything (containers + volumes + data):**

```bash
docker compose down -v
```

---

## License

This tutorial and example workflow are provided for educational use. See `LICENSE` for details.

---

### Credits

Made with ❤️ for students learning automation. Enjoy building!
