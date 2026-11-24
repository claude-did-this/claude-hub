# 🚀 Quick Start Guide

> [!WARNING]
> **This project is archived and no longer maintained.** This guide is kept for reference only and may no longer work. See the [README](./README.md#-project-status-archived) for known issues and the maintained alternative, the [Claude Code GitHub Action](https://github.com/anthropics/claude-code-action).

Get Claude responding to your GitHub issues in minutes using Cloudflare Tunnel.

## Prerequisites
- GitHub account
- Docker installed
- Claude.ai account with Max plan (5x or 20x)
- Cloudflare account (free tier works)

## Step 1: Create a GitHub Bot Account

1. Sign out of GitHub and create a new account for your bot (e.g., `YourProjectBot`)
2. Sign in to your bot account and create a [Personal Access Token](https://github.com/settings/tokens) with `repo` and `write` permissions
3. Add the bot account as a collaborator to your repositories from your main account

## Step 2: Clone and Configure

```bash
# Clone the repository
git clone https://github.com/claude-did-this/claude-hub.git
cd claude-hub

# Copy the quickstart environment file
cp .env.quickstart .env

# Edit .env with your values
nano .env
```

Required values:
- `GITHUB_TOKEN`: Your bot account's GitHub Personal Access Token
- `GITHUB_WEBHOOK_SECRET`: Generate with `openssl rand -hex 32`
- `BOT_USERNAME`: Your bot's GitHub username (e.g., `@YourProjectBot`)
- `BOT_EMAIL`: Your bot's email
- `AUTHORIZED_USERS`: Comma-separated GitHub usernames who can use the bot

## Step 3: Authenticate Claude

```bash
# Run the interactive setup
./scripts/setup/setup-claude-interactive.sh
```

This will:
1. Open your browser for Claude.ai authentication
2. Save your credentials securely
3. Confirm everything is working

## Step 4: Start the Service

```bash
# Start the webhook service
docker compose up -d

# Check it's running
docker compose logs -f webhook
```

## Step 5: Install Cloudflare Tunnel

### Option A: Ubuntu/Debian
```bash
# Add cloudflare gpg key
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

# Add this repo to your apt repositories
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared focal main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

# Install cloudflared
sudo apt-get update && sudo apt-get install cloudflared
```

### Option B: Direct Download
```bash
# Download the latest cloudflared binary
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

### Option C: Using snap
```bash
sudo snap install cloudflared
```

## Step 6: Create Tunnel

```bash
# Create a tunnel to your local service
cloudflared tunnel --url http://localhost:3002
```

Copy the generated URL (like `https://abc123.trycloudflare.com`)

## Step 7: Configure GitHub Webhook

1. Go to your repository → Settings → Webhooks
2. Click "Add webhook"
3. **Payload URL**: Your Cloudflare URL + `/api/webhooks/github`
   - Example: `https://abc123.trycloudflare.com/api/webhooks/github`
4. **Content type**: `application/json`
5. **Secret**: Same value as `GITHUB_WEBHOOK_SECRET` in your .env
6. **Events**: Select "Let me select individual events"
   - Check: Issues, Issue comments, Pull requests, Pull request reviews

## 🎉 You're Done!

Test it in your own repository by creating an issue and mentioning your bot:

```
@YourProjectBot Can you help me understand this codebase?
```

**Note:** Your bot will only respond in repositories where you've configured the webhook and to users listed in `AUTHORIZED_USERS`.

## Next Steps

- **Production Deployment**: Set up a permanent Cloudflare Tunnel with `cloudflared service install`
- **Advanced Features**: Check `.env.example` for PR auto-review, auto-tagging, and more
- **Multiple Repos**: Add the same webhook to any repo where you want bot assistance

## Troubleshooting

**Bot not responding?**
- Check logs: `docker compose logs webhook`
- Verify webhook delivery in GitHub → Settings → Webhooks → Recent Deliveries
- Ensure the commenting user is in `AUTHORIZED_USERS`

**Authentication issues?**
- Re-run: `./scripts/setup/setup-claude-interactive.sh`
- Ensure you have an active Claude.ai Max plan (5x or 20x)

**Need help?** This project is archived and no longer supported. See the [README](./README.md#-project-status-archived) for known issues, or use the official [Claude Code GitHub Action](https://github.com/anthropics/claude-code-action) instead.