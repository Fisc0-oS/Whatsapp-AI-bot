# Troubleshooting

## Bot doesn't respond

1. Check WAHA session status:
```bash
curl -H "X-Api-Key: YOUR_KEY" http://localhost:3000/api/sessions/default
```
Should show `"status":"WORKING"`

2. Check n8n workflow is **Active** (green toggle)

3. Check webhook is set correctly:
```bash
curl -H "X-Api-Key: YOUR_KEY" http://localhost:3000/api/sessions/default | grep webhook
```

4. Check n8n logs:
```bash
docker compose logs n8n --tail=50
```

## Ollama not reachable from n8n

Test from Ubuntu:
```bash
curl http://YOUR_WINDOWS_IP:11434/api/tags
```

If no response:
- Check `OLLAMA_HOST=0.0.0.0:11434` is set (Windows: `setx OLLAMA_HOST "0.0.0.0:11434" /M`)
- Restart Ollama completely
- Check firewall: `netsh advfirewall firewall show rule name="Ollama"`
- Verify: `netstat -ano | findstr 11434` should show `0.0.0.0:11434`

## Bot responds multiple times

This happens when WAHA sends multiple webhooks for one message (e.g. when the bot's own sent messages trigger webhooks).

Fix: Ensure Message Filter node has these 3 conditions:
- `$json.body.event` equals `message`
- `$json.body.payload.fromMe` equals `false` (Boolean)
- `$json.body.payload.hasMedia` equals `false` (Boolean)

## Obsidian reservations not saving

1. Test Obsidian API from Ubuntu:
```bash
curl -k -H "Authorization: Bearer YOUR_KEY" https://YOUR_IP:27124/
```

2. Check portproxy is set:
```cmd
netsh interface portproxy show all
```
Should show `0.0.0.0:27124`

3. Check firewall rule exists for port 27124

## sender_jid is undefined / chatId empty

The `chatId` must come from `payload.id.split('_')[1]`.

Check Extract Fields node:
```
sender_jid = {{ $json.body.payload.id.split('_')[1] }}
```

## Model responds in wrong language

Increase temperature slightly (0.3) and ensure the LANGUAGE RULE is the first line of your system prompt.

## Slow responses (>30 seconds)

- GPU not being used: check `nvidia-smi` during inference
- CPU fallback: ensure `OLLAMA_NUM_GPU=1` or Ollama Desktop has GPU enabled
- Model too large: switch from gemma2:9b to gemma2:2b or llama3.2:3b
