# 🤖 WhatsApp AI Bot — Local & Private

<div align="center">

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Docker](https://img.shields.io/badge/docker-compose-2496ED?logo=docker)
![n8n](https://img.shields.io/badge/n8n-self--hosted-EA4B71?logo=n8n)
![Ollama](https://img.shields.io/badge/Ollama-local%20LLM-black)
![WAHA](https://img.shields.io/badge/WAHA-WhatsApp%20API-25D366?logo=whatsapp)
![Privacy](https://img.shields.io/badge/privacy-100%25%20local-green)
![Cost](https://img.shields.io/badge/cost-%240%2Fmonth-brightgreen)

**A fully local, private WhatsApp AI assistant — no cloud APIs, no token costs, no data leaks.**

Built with WAHA + n8n + Ollama + Obsidian. Runs entirely on your homelab hardware.

[Features](#-features) · [Architecture](#-architecture) · [Quick Start](#-quick-start) · [Demo](#-demo) · [Roadmap](#-roadmap)

</div>

---

## ✨ Features

- 💬 **Natural conversation** in English & German (auto-detects language)
- 🧠 **Local LLM** via Ollama — gemma2:2b or llama3.2:3b, runs on GPU
- 📋 **Smart intent detection** — reservations, menu, hours, cancellations
- 📓 **Obsidian integration** — reservations saved as markdown notes
- 🔒 **100% private** — zero data leaves your network
- 💸 **Zero cost** — no OpenAI, no API keys, no subscriptions
- ⚡ **Fast** — 60+ tokens/sec on GTX 1080 with CUDA
- 🍽️ **Full menu with nutrition info** — calories, protein, sugar, diet type
- 🗓️ **Table reservations** — collect details, confirm, save to Obsidian
- 📱 **Mobile-optimized** — short messages, emojis, WhatsApp formatting

---

## 🏗️ Architecture

```
WhatsApp User
      │
      ▼
  WAHA Gateway          ← WhatsApp Web bridge (Docker)
      │  webhook
      ▼
   n8n Engine           ← Automation & workflow logic (Docker)
      │
      ├── Intent Router (reservation / menu / cancel / general)
      │
      ├── Context Injection (menu, hours, FAQ from flat file)
      │
      ├── Chat History (sliding window, last 6 turns)
      │
      ▼
  Ollama + GPU          ← Local LLM inference (gemma2:2b)
      │
      ├── Response formatter
      │
      ├── Obsidian REST API  ← Save reservations as .md notes
      │
      ▼
  WAHA → WhatsApp User
```

### Stack

| Component | Technology | Why |
|-----------|-----------|-----|
| WhatsApp Gateway | WAHA Core | Free, self-hosted WhatsApp API |
| Automation | n8n (self-hosted) | Visual workflow, no code needed |
| LLM Runtime | Ollama | Local GPU inference, CUDA support |
| Model | gemma2:2b / llama3.2:3b | Best speed/quality for 8GB VRAM |
| Notes | Obsidian + Local REST API | Human-readable reservation storage |
| Deployment | Docker Compose | One-command setup |

---

## 💻 Hardware Requirements

| Component | Minimum | Recommended (this build) |
|-----------|---------|--------------------------|
| CPU | Any x86-64 | Intel i7-4790K |
| RAM | 8GB | 16GB |
| GPU | None (CPU mode) | NVIDIA GTX 1080 8GB |
| Storage | 20GB | 50GB SSD |
| OS | Ubuntu 20.04+ | Ubuntu Server 24.04 |

> **CPU-only mode** works but responses take 15–30s instead of 1–2s.

---

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- NVIDIA GPU + drivers (optional but recommended)
- Obsidian desktop app with [Local REST API plugin](https://github.com/coddingtonbear/obsidian-local-rest-api)

### 1. Clone the repo

```bash
git clone https://github.com/YOUR_USERNAME/whatsapp-ai-bot.git
cd whatsapp-ai-bot
```

### 2. Configure environment

```bash
cp .env.example .env
nano .env  # Edit your settings
```

### 3. Start containers

```bash
docker compose up -d
```

### 4. Pull the LLM model

```bash
# On Linux (Docker)
docker exec ollama ollama pull gemma2:2b

# On Windows (Ollama Desktop)
ollama pull gemma2:2b
```

### 5. Connect WhatsApp

```bash
# Start WAHA session
curl -X POST -H "X-Api-Key: YOUR_API_KEY" \
  http://localhost:3000/api/sessions/default/start

# Get QR code
open http://localhost:3000/dashboard
```

Scan the QR code with WhatsApp on your phone.

### 6. Import n8n workflow

1. Open n8n at `http://localhost:5678`
2. Click **Import** → select `workflow.json`
3. Configure credentials
4. **Activate** the workflow

### 7. Set webhook

```bash
curl -X PUT \
  -H "X-Api-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"config":{"webhooks":[{"url":"http://n8n:5678/webhook/waha-hook","events":["message"]}]}}' \
  http://localhost:3000/api/sessions/default
```

**Done!** Send a message to your WhatsApp number. 🎉

---

## 📱 Demo

### Menu Query
```
User: What coffee do you have?

Bot: ☕ We offer several types of coffee:
▪️ Espresso (30ml) — 2.50 EUR | 5 kcal | Vegan
▪️ Cappuccino (180ml) — 3.50 EUR | 90 kcal | 🥚 Vegetarian
▪️ Latte (300ml) — 4.00 EUR | 150 kcal | 🥚 Vegetarian
▪️ Cold Brew (300ml) — 4.50 EUR | 15 kcal | 🌱 Vegan
```

### Reservation Flow
```
User: I want to book a table for 4 on Saturday at 19:00, name Mueller

Bot: ✅ Reservation confirmed!

- 📅 Date: Saturday June 14
- 🕐 Time: 19:00
- 👥 Guests: 4
- 👤 Name: Mueller

We look forward to seeing you! ☕
```

### German Language (auto-detected)
```
User: Was sind eure Öffnungszeiten?

Bot: 🕐 Wir haben folgende Öffnungszeiten:
▪️ Mo–Fr: 07:00 – 20:00
▪️ Samstag: 08:00 – 21:00
▪️ Sonntag: 09:00 – 18:00

Bis bald! ☕
```

---

## ⚙️ Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `WAHA_API_KEY` | WAHA authentication key | `changeme` |
| `N8N_SECURE_COOKIE` | Disable for HTTP access | `false` |
| `OLLAMA_HOST` | Ollama bind address | `0.0.0.0:11434` |
| `OBSIDIAN_API_KEY` | Obsidian REST API key | — |
| `OBSIDIAN_HOST` | Obsidian host IP | `192.168.1.x` |

### Model Selection

Edit the model in n8n Code nodes:

```javascript
model: 'gemma2:2b'      // Recommended: 61 t/s, smart
model: 'llama3.2:3b'    // Fast: 71 t/s
model: 'gemma2:9b'      // Smartest: 22 t/s (needs 6GB+ VRAM)
```

### Customizing the Menu / Context

Edit the `context` string in the **Load Context & Detect Intent** node in n8n. No coding required — just update the text.

---

## 📁 Project Structure

```
whatsapp-ai-bot/
├── docker-compose.yml       # All services
├── .env.example             # Environment template
├── workflow.json            # n8n workflow (import this)
├── docs/
│   ├── SETUP.md             # Detailed setup guide
│   ├── OBSIDIAN.md          # Obsidian integration
│   ├── GPU_SETUP.md         # NVIDIA GPU configuration
│   └── TROUBLESHOOTING.md   # Common issues
├── .github/
│   └── ISSUE_TEMPLATE/      # Bug & feature templates
└── README.md
```

---

## 🗺️ Roadmap

- [x] WhatsApp message handling
- [x] Local LLM with GPU acceleration
- [x] Bilingual support (EN/DE)
- [x] Table reservations → Obsidian
- [x] Intent detection (menu, reservations, cancel)
- [x] Nutrition info in menu
- [ ] Voice message transcription (Whisper)
- [ ] Image recognition (menu photos)
- [ ] Multi-language support (FR, ES, IT)
- [ ] Admin dashboard
- [ ] Analytics & conversation logs
- [ ] Telegram bot variant

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) first.

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit: `git commit -m 'Add amazing feature'`
4. Push: `git push origin feature/amazing-feature`
5. Open a Pull Request

---

## ⚠️ Disclaimer

This project uses WAHA Core (free tier). WhatsApp automation must comply with [WhatsApp Terms of Service](https://www.whatsapp.com/legal/terms-of-service). Use responsibly for legitimate business purposes.

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

Built with ❤️ for the homelab community

⭐ Star this repo if it helped you!

</div>
