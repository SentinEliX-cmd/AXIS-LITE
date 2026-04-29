# AXIS-LITE 🔺

> **AI Security Agent** — Hybrid offensive/defensive security workflows powered by Agent Zero + Ollama Cloud API (qwen3.5:397b). Runs via Docker on Windows 11.

![Agent Zero](https://img.shields.io/badge/Agent%20Zero-v1.10-blue)
![Ollama](https://img.shields.io/badge/Ollama-Cloud%20API-green)
![Model](https://img.shields.io/badge/Model-qwen3.5%3A397b-purple)
![Platform](https://img.shields.io/badge/Platform-Windows%2011-lightgrey)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

---

## What is AXIS-LITE?

AXIS-LITE is an AI security agent stack built on [Agent Zero](https://www.agent-zero.ai/). It uses the Ollama Cloud API with `qwen3.5:397b` to power a stateful penetration testing and security research agent, running via Docker on Windows 11.

Three operational profiles:

| Profile | Role |
|---|---|
| 🔴 **Hacker** | Stateful pentest — recon → enum → vuln analysis → report |
| 🟡 **Researcher** | OSINT, CTF investigation, threat intel |
| 🔵 **Defender** | SOC triage, log analysis, incident response |

---

## Stack

| Component | Details |
|---|---|
| Agent Framework | Agent Zero v1.10 (`agent0ai/agent-zero:latest`) |
| Chat Model | Ollama Cloud API (`https://api.ollama.com`) — `qwen3.5:cloud` |
| Embed Model | Local Ollama (`http://host.docker.internal:11434`) — `granite-embedding:278m` |
| Runtime | Docker Desktop (Windows 11) |
| Web UI | `http://localhost:8088` |
| Wordlists | Mounted from `../wordlists` → `/usr/share/wordlists` |

> No Modelfiles. Ollama serves models via free local API. Swap models in the Agent Zero UI.

---

## Features

- **Stateful Pentest Brain** — memory.json tracks phases (recon, service enum, dir enum, vuln analysis). No repeated scans.
- **Intent Router** — classifies requests before acting. General questions answered directly, no tool spam.
- **Loop Prevention** — Two-strikes rule, max one tool per cycle, file existence checks before re-running.
- **Execute-First Hacker Mode** — no requirements interviews, acts immediately on clear targets.
- **Tool Integration** — nmap, nikto, gobuster, hydra, searchsploit, netcat. All output saved to workdir.
- **Safety Gate** — requires user confirmation before scanning external targets.

---

## Project Structure

```
AXIS-LITE/
├── docker-compose.yml                        # Container setup
├── settings.json                             # Agent Zero v1.10 config
├── data/                                     # Mounted to /a0/usr inside container
│   └── memory/
│       ├── specifics.md                      # Stateful pentest brain + loop fix
│       └── behaviour.md                      # Tool call discipline rules
├── prompts/
│   └── agent.system.main.communication.md   # Hacker profile communication override
└── docs/
    └── windows-setup.md                      # Full Windows 11 setup guide
```

---

## Quick Start

### Prerequisites

- Windows 11 with [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Ollama](https://ollama.com) installed and running natively (not in Docker)
- Wordlists folder at `../wordlists` relative to this repo (or edit the volume path)

### 1. Clone

```bash
git clone https://github.com/SentinEliX-cmd/AXIS-LITE.git
cd AXIS-LITE
```

### 2. Pull a model via Ollama

```bash
ollama pull qwen2.5:7b
```

Any model works. Set it in the Agent Zero UI after launch.

### 3. Launch

```bash
docker compose up -d
```

### 4. Access

```
http://localhost:8088
```

Import `settings.json` via the Agent Zero UI settings panel to apply AXIS-LITE config.

---

## Memory System (Pentest Brain)

The stateful pentest brain lives in `data/memory/specifics.md`. It maintains a `memory.json` file in the workdir to track scan phases:

```json
{
  "target": "",
  "phases": {
    "recon": { "done": false, "ports": [] },
    "service_enum": { "done": false, "http_ports": [] },
    "dir_enum": { "done": false, "found_dirs": [] },
    "vuln_analysis": { "done": false, "findings": [] }
  }
}
```

The agent reads this before every action — completed phases are skipped, no duplicate scans.

---

## Tools Confirmed Working

| Tool | Usage |
|---|---|
| `nmap` | Port scan → `/a0/usr/workdir/nmap.txt` |
| `nikto` | Web vuln scan → `/a0/usr/workdir/nikto.txt` |
| `gobuster` | Dir/subdomain brute force → `/a0/usr/workdir/gobuster.txt` |
| `searchsploit` | CVE matching → `/a0/usr/workdir/searchsploit.txt` |
| `hydra` | Auth brute force (explicit instruction only) |
| `netcat` | Manual service interaction |

---

## Wordlists Setup

The compose file expects wordlists at `../wordlists` (one level above this repo):

```
parent-folder/
├── AXIS-LITE/          ← this repo
└── wordlists/          ← seclists, dirb, etc.
    └── common.txt
```

Or edit `docker-compose.yml` to point to your actual wordlists path.

---

## Severity Reference

| Level | Description |
|---|---|
| Critical | RCE, unauthenticated admin access |
| High | Auth bypass, sensitive data exposure |
| Medium | Missing headers, misconfigurations |
| Low | Outdated versions, verbose errors |
| Info | Open ports, service fingerprints |

---

## Full Setup Guide

See [`docs/windows-setup.md`](./docs/windows-setup.md)

---

## Author

**Eli** — Cybersecurity & Forensics | Nairobi, Kenya  
[GitHub](https://github.com/SentinEliX-cmd) • [Sentinel](https://github.com/SentinEliX-cmd/Sentinel)

---

> Built for real security work. Not demos.
