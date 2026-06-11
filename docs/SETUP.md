# Detailed Setup Guide

## Step 1 — Prerequisites

### Install Docker
```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

### Install Ollama (Windows)
Download from https://ollama.com/download/windows

After install, expose to network:
```cmd
setx OLLAMA_HOST "0.0.0.0:11434" /M
```
Restart Ollama, then add firewall rule:
```cmd
netsh advfirewall firewall add rule name="Ollama" dir=in action=allow protocol=TCP localport=11434
```

## Step 2 — Clone & Configure

```bash
git clone https://github.com/YOUR_USERNAME/whatsapp-ai-bot.git
cd whatsapp-ai-bot
cp .env.example .env
```

Edit `.env` with your actual values:
- Set `WAHA_API_KEY` to a strong password
- Set `OLLAMA_HOST_IP` to your Windows machine IP
- Set `OBSIDIAN_API_KEY` from Obsidian Local REST API plugin

## Step 3 — Pull LLM Model

On your Ollama machine:
```bash
ollama pull gemma2:2b    # Recommended (61 t/s on GTX 1080)
# or
ollama pull llama3.2:3b  # Faster (71 t/s) but less smart
# or  
ollama pull gemma2:9b    # Smartest (22 t/s, needs 6GB+ VRAM)
```

## Step 4 — Start Services

```bash
docker compose up -d
docker compose logs -f  # Watch logs
```

## Step 5 — Connect WhatsApp

```bash
# Start session
curl -X POST \
  -H "X-Api-Key: YOUR_API_KEY" \
  http://localhost:3000/api/sessions/default/start

# Check status (wait for SCAN_QR_CODE)
curl -H "X-Api-Key: YOUR_API_KEY" \
  http://localhost:3000/api/sessions/default

# Open dashboard to scan QR
open http://localhost:3000/dashboard
```

Login: `admin` / your API key

## Step 6 — Import n8n Workflow

1. Open `http://localhost:5678`
2. Create account
3. Click `...` menu → Import from file
4. Select `workflow.json`
5. Fix any node connections if needed
6. Click **Activate**

## Step 7 — Configure Webhook

```bash
curl -X PUT \
  -H "X-Api-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "config": {
      "webhooks": [{
        "url": "http://172.17.0.1:5678/webhook/waha-hook",
        "events": ["message"]
      }]
    }
  }' \
  http://localhost:3000/api/sessions/default
```

> Note: `172.17.0.1` is the Docker bridge IP. Verify with `docker network inspect bridge`.

## Step 8 — Setup Obsidian

1. Install Obsidian from https://obsidian.md
2. Create a vault (e.g. `CafeDemo`)
3. Settings → Community Plugins → Browse → "Local REST API with MCP"
4. Install & Enable
5. Note your API key and port (27124)
6. Allow network access:
```cmd
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=27124 connectaddress=127.0.0.1 connectport=27124
netsh advfirewall firewall add rule name="Obsidian REST API" dir=in action=allow protocol=TCP localport=27124
```
7. Create `Reservations` folder in vault

## Verification

Test the bot by sending a WhatsApp message:
- `Hello` → greeting
- `What's on the menu?` → full menu
- `What are your opening hours?` → hours
- `I want to book a table` → reservation flow

Check n8n Executions tab for any errors.
