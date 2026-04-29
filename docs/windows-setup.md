# AXIS-LITE — Windows 11 Setup Guide

Complete walkthrough. Tested on HP Victus, Windows 11, NVIDIA RTX 3050, 24GB RAM.

---

## Requirements

| Component | Minimum | Tested On |
|---|---|---|
| OS | Windows 11 | Windows 11 |
| RAM | 16GB | 24GB (8GB + 16GB DDR4 3200MHz) |
| GPU | Any NVIDIA | RTX 3050 |
| Docker | Desktop 4.x | Latest |
| Ollama | Latest | Latest |

---

## Step 1 — Install Ollama

1. Download from [ollama.com](https://ollama.com)
2. Run the installer
3. **Open a dedicated terminal and run:**
```bash
ollama serve
```
**Leave this terminal open.** Ollama must run natively on Windows — not inside Docker — to access your GPU directly. It listens on `http://localhost:11434`.

### Pull a model
```bash
ollama pull qwen2.5:7b
```
Any model supported by Ollama works. Set the active model inside the Agent Zero UI after launch.

---

## Step 2 — Install Docker Desktop

1. Download from [docker.com](https://www.docker.com/products/docker-desktop)
2. Install with WSL2 backend (default)
3. Start Docker Desktop
4. Verify:
```bash
docker --version
docker compose version
```

---

## Step 3 — Wordlists

The compose file mounts `../wordlists` into the container. Set this up before launching:

```
parent-folder/
├── AXIS-LITE/       ← this repo
└── wordlists/       ← put your lists here
    └── common.txt
```

Recommended: download [SecLists](https://github.com/danielmiessler/SecLists) into the wordlists folder.

Or edit `docker-compose.yml` to point to wherever your wordlists live:
```yaml
- /your/actual/path/wordlists:/usr/share/wordlists
```

---

## Step 4 — Clone AXIS-LITE

```bash
git clone https://github.com/SentinEliX-cmd/AXIS-LITE.git
cd AXIS-LITE
```

---

## Step 5 — Launch

```bash
docker compose up -d
```

First run pulls `agent0ai/agent-zero:latest` (~2GB). Subsequent launches are instant.

Check it's running:
```bash
docker compose ps
```

Access the UI:
```
http://localhost:8088
```

---

## Step 6 — Import Settings

1. Open `http://localhost:8088`
2. Go to **Settings** (gear icon)
3. Import `settings.json` from this repo
4. This sets Agent Zero to v1.10 config with correct workdir paths

---

## Step 7 — Set Your Model

In the Agent Zero UI → Settings → Models:
- Set **Chat Model** to your pulled Ollama model (e.g. `qwen2.5:7b`)
- Set provider to **Ollama**
- Base URL: `http://host.docker.internal:11434`

`host.docker.internal` resolves to your Windows host from inside Docker — this is how the container reaches your local Ollama instance.

---

## Step 8 — Load Memory Files

The `data/` folder is already mounted to `/a0/usr` in the container via the compose volume.

`data/memory/specifics.md` and `data/memory/behaviour.md` are picked up automatically by Agent Zero's memory system on startup.

Verify inside the container:
```bash
docker exec -it agent-zero ls /a0/usr/memory/
```

You should see `specifics.md` and `behaviour.md`.

---

## Step 9 — Verify Tools

In the AXIS-LITE chat:
```
check if nmap, nikto, gobuster, and searchsploit are available and print their versions
```

All four should respond with version info.

---

## Stopping AXIS-LITE

```bash
docker compose down
```

Stop Ollama by closing the `ollama serve` terminal or:
```bash
taskkill /F /IM ollama.exe
```

---

## Accessing Scan Output

All scan results save to `/a0/usr/workdir/` inside the container. To pull them to Windows:

```bash
docker cp agent-zero:/a0/usr/workdir ./workdir-output
```

---

## Troubleshooting

**Agent can't reach Ollama**
- Confirm `ollama serve` is running in a separate terminal
- Check OLLAMA_BASE_URL in docker-compose.yml is `http://host.docker.internal:11434`
- Allow Ollama through Windows Defender Firewall if prompted

**Memory files not loading**
- Check `docker compose ps` — volume mount must show `./data:/a0/usr`
- Run `docker exec -it agent-zero ls /a0/usr/memory/` to confirm files are present

**Agent looping**
- The loop fix in `specifics.md` handles this — if still looping, restart the container: `docker compose restart`

**Wordlists not found inside container**
- Confirm wordlists folder exists at `../wordlists` relative to the repo
- Or update the volume path in `docker-compose.yml`

---

*Tested: Windows 11 · HP Victus · RTX 3050 · 24GB RAM · Agent Zero v1.10*
