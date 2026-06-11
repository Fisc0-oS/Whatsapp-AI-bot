# Contributing to WhatsApp AI Bot

Thank you for your interest in contributing! 🎉

## How to Contribute

### Reporting Bugs
- Use the GitHub Issues tab
- Include your OS, hardware specs, Docker version
- Paste relevant logs from `docker compose logs`

### Suggesting Features
- Open a GitHub Issue with the `enhancement` label
- Describe the use case clearly

### Pull Requests
1. Fork the repository
2. Create a branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Test thoroughly
5. Submit a Pull Request with a clear description

## Development Setup

```bash
git clone https://github.com/YOUR_USERNAME/whatsapp-ai-bot.git
cd whatsapp-ai-bot
cp .env.example .env
docker compose up -d
```

## Code Style

- JavaScript (n8n Code nodes): clear variable names, comments for complex logic
- Keep Code nodes under 50 lines where possible
- Document any new environment variables in `.env.example`

## Areas Where Help is Needed

- 🌍 Additional language support (FR, ES, IT, PL)
- 🎤 Voice message transcription (Whisper integration)
- 📊 Analytics dashboard
- 🧪 Testing & documentation improvements
