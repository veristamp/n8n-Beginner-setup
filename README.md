# n8n Beginner Tutorial – Season 1

**Windows + macOS (All‑in‑One Docker Compose)**
**Zero external ngrok install • Everything runs in Docker**

> This repo ships with:
>
> * `docker-compose.yml` (n8n + PostgreSQL + ngrok, single command)
> * `example.env` (template you’ll copy to `.env`)
> * `workflows/lead-capture.json` (importable n8n workflow)

---

## Table of Contents

* [Quick Start (TL;DR)](#quick-start-tldr)
* [Setup: ngrok domain & token](#setup-ngrok-domain--token)
* [Setup: Google Cloud OAuth (Sheets + Gmail)](#setup-google-cloud-oauth-sheets--gmail)
* [Telegram Bot Setup](#telegram-bot-setup)
* [Import the example workflow](#import-the-example-workflow)
* [Beautiful HTML form (optional)](#beautiful-html-form-optional)
* [Troubleshooting](#troubleshooting)
* [FAQ](#faq)

---
## Prerequisites

Before starting, make sure these tools are installed:

* **Docker Desktop**

  * Windows: [Install Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
  * macOS: [Install Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/)
* **Git**: [https://git-scm.com/](https://git-scm.com/)
* **Visual Studio Code**: [https://code.visualstudio.com/](https://code.visualstudio.com/)

> Make sure Docker Desktop is running before you start.

---
## Quick Start (TL;DR)

1. **Clone this repo**

   ```bash
   git clone https://github.com/veristamp/n8n-Beginner-setup.git
   cd n8n-Beginner-setup
   ```

2. **Copy example environment file and edit**

   ```bash
   cp example.env .env
   ```

   Then fill it in after creating your ngrok account (next step).

3. **Get your ngrok domain and token** (see below) and add them to `.env`:

   ```env
   NGROK_AUTHTOKEN=your_authtoken_here
   NGROK_DOMAIN=my-cool-n8n.ngrok.io
   ```

4. **Start n8n + Postgres + ngrok**

   ```bash
   docker compose up -d
   ```

5. Wait 10–15 seconds, then open `https://your-ngrok-domain` in your browser and create your n8n admin user.

6. When n8n is up, continue to **Google OAuth setup**.

---

## Setup: ngrok domain & token

1. Go to [https://dashboard.ngrok.com](https://dashboard.ngrok.com) and sign in.
2. Navigate to **Cloud Edge → Domains → Reserve a domain** (free tier).

   * Example: `my-cool-n8n.ngrok.io`
3. Copy your **Authtoken** from [Your Authtoken page](https://dashboard.ngrok.com/get-started/your-authtoken).
4. Paste both values into `.env`:

   ```env
   NGROK_AUTHTOKEN=your_authtoken_here
   NGROK_DOMAIN=my-cool-n8n.ngrok.io
   ```
5. Save the file.

---

## Setup: Google Cloud OAuth (Sheets + Gmail)

1. Go to [https://console.cloud.google.com](https://console.cloud.google.com).
2. Create a new project called `n8n-season1`.
3. Enable these APIs:

   * Google Sheets API
   * Gmail API
4. Under **OAuth consent screen**, select **External**, fill in your info, and save.
5. Go to **Credentials → Create Credentials → OAuth Client ID** → Choose **Web Application**.
6. Add the following **Authorized redirect URI**:

   ```
   https://YOUR_NGROK_DOMAIN/oauth2/callback
   ```
7. Download your OAuth JSON credentials.
8. In n8n → **Credentials**, select:

   * **Google Sheets OAuth2 API** → Paste **Client ID** and **Client Secret** from JSON.
   * **Gmail OAuth2 API** → Same process.
9. Click **Connect**, sign in with Google, and allow access.

> The callback URL is automatically handled by n8n — no manual editing required.
> For deeper details, check the official n8n guide: [Google OAuth Generic Docs](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-generic/)

---

## Telegram Bot Setup

1. Open Telegram and search for `@BotFather`.
2. Run the command `/newbot`.
3. Give it a name (e.g., **n8n Alert Bot**) and a username ending in `bot` (e.g., `n8n_alert_bot`).
4. Copy the **API Token** shown by BotFather.
5. In n8n → **Credentials → Telegram Bot API**, paste your token and click **Connect**.

Now your n8n workflows can send messages to Telegram!

---

## Import the example workflow

1. In n8n, go to **Workflows → Import from File**.
2. Choose `workflows/lead-capture.json` from this repo.
3. Update the nodes to use your Google Sheets, Gmail, and Telegram credentials.
4. Click **Execute** or toggle **Active** to run it automatically.

**Workflow overview:**

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

1. Copy the code below into a file called `form.html`.
2. Replace `YOUR_NGROK_DOMAIN` with your actual domain.
3. Double-click it to open locally, or host it anywhere.

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

Now you have a simple, working lead capture form!

---

## Troubleshooting

**n8n shows 502/Bad Gateway**
Wait 30–60 seconds after `docker compose up -d`. Then run:

```bash
docker compose logs -f n8n
docker compose logs -f ngrok
```

**Google OAuth redirect_uri_mismatch**
Ensure the Google Console **Authorized redirect URI** exactly matches `https://YOUR_NGROK_DOMAIN/oauth2/callback`.

**PostgreSQL unhealthy**
If DB fails, reset volumes (this deletes data):

```bash
docker compose down -v && docker compose up -d
```

**Telegram not sending messages**
Check the token and make sure the bot has permission to message you.

---

## FAQ

**Q: Can I change the ngrok domain later?**
A: Yes, update `.env` with new domain and token, then `docker compose up -d`.

**Q: Is this production ready?**
A: It’s perfect for learning and demos. For production, add backups and a static reverse proxy.

**Q: How do I upgrade n8n?**
A: Update image tag in `docker-compose.yml`, then run:

```bash
docker compose pull && docker compose up -d
```

---

### Credits

Made with ❤️ for students learning automation. Enjoy building!
