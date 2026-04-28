# 🛡️ Internal Network AI Security Agent

An AI-powered network security scanner that runs on a **Raspberry Pi 3** using **n8n + Docker**. It automatically discovers devices on your local network, performs deep port scans with nmap, and uses **Claude AI** to generate detailed security reports with CVE lookups and threat intelligence.

---

## 🔍 What It Does

Trigger a single webhook and this agent will:

1. **Discover** all live devices on your subnet
2. **Deep scan** each device for open ports and services
3. **Analyze** findings using Claude AI (Anthropic)
4. **Lookup CVEs** for every detected service version
5. **Generate** a full HTML security report
6. **Email** the report directly to you

---

## 🏗️ Architecture

```
Webhook (GET /nmap-scan?target=192.168.1.0/24)
  │
  ▼
Subnet Discovery ── nmap -sn (find live hosts)
  │
  ▼
Extract Live IPs ── JS parses nmap output → IP array
  │
  ▼
Loop Over IPs ── iterates through each device
  │
  ├──► Deep Scan ── nmap -sV --open per device
  │
  ├──► Parse Scan Output ── formats for Claude
  │
  └──► Security Analyzer (Claude AI Agent)
         ├── CVE Lookup (SerpAPI)
         └── Anthropic Chat Model
  │
  ▼
Collect All Reports ── aggregates per-device reports
  │
  ▼
Build HTML Report ── formats into styled HTML
  │
  ▼
Send Email ── delivers report to your inbox
  │
  ▼
Respond to Webhook ── returns results to caller
```

---

## 📋 Sample Findings

Example output from a typical home network scan:

| Severity | Finding |
|----------|---------|
| 🔴 CRITICAL | Insecure remote access protocol on common port |
| 🟠 HIGH | Outdated SSH server with known CVEs |
| 🟠 HIGH | UPnP services exposed unnecessarily |
| 🟠 HIGH | Outdated kernel version detected |
| 🟡 MEDIUM | SMB service with known vulnerability class |
| 🟡 MEDIUM | DNS service with known CVEs |

---

## ⚙️ Requirements

- **Hardware:** Raspberry Pi 3 (or any ARM/x86 Linux machine)
- **OS:** Raspberry Pi OS Lite
- **Software:**
  - Docker & Docker Compose
  - n8n (running in Docker)
  - nmap (installed on Pi)
- **API Keys:**
  - [Anthropic API Key](https://console.anthropic.com/) (Claude)
  - [SerpAPI Key](https://serpapi.com/) (CVE lookups)
- **Network:** Pi connected to your local network (Ethernet recommended for full device discovery)

---

## 🚀 Setup

### 1. Import the Workflow

1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Select `internal-network-agent.json`

### 2. Configure Credentials

After importing, you'll need to set up these credentials in n8n:

| Node | Credential Type | What to Add |
|------|----------------|-------------|
| Anthropic Chat Model | Anthropic API | Your API key |
| CVE Look up | SerpAPI | Your API key |
| Subnet Discovery / Deep Scan | SSH | Pi username, password/key, host IP |
| Send an Email | SMTP / Gmail | Your email credentials |

### 3. Update Your Network Range

In the webhook URL, change the target to match your subnet:

```bash
# Default
curl "http://<PI_IP>:5678/webhook/nmap-scan?target=192.168.1.0/24"

# Adjust if your network uses a different range
curl "http://<PI_IP>:5678/webhook/nmap-scan?target=10.0.0.0/24"
```

### 4. Test It

```bash
# Single device scan
curl "http://<PI_IP>:5678/webhook/nmap-scan?target=192.168.1.1"

# Full network scan
curl "http://<PI_IP>:5678/webhook/nmap-scan?target=192.168.1.0/24"
```

> **Note:** For test mode in n8n editor, use `webhook-test` instead of `webhook` in the URL, and click "Execute Workflow" first.

---

## 📊 What the Report Includes

- All open ports and running services per device
- Risk level per finding (Critical / High / Medium / Low)
- CVE matches for detected service versions
- Remediation recommendations
- Overall risk score (1-10) per device

---

## ⚠️ Known Limitations

- **Simple Memory** is incompatible with webhook-triggered workflows in n8n — removed from this version
- **Pi 3 performance:** Full /24 subnet scans with deep scanning take several minutes. Be patient!
- **Claude iteration limits:** If many services are found, Claude may hit the max iteration cap. Increase it in the Security Analyzer node settings if needed
- **WiFi vs Ethernet:** Pi on WiFi may miss some devices. Use Ethernet for more reliable discovery

---

## 📁 Project Structure

```
├── internal-network-agent.json   # n8n workflow (import this)
├── README.md                     # this file
```

---

## 🛠️ Built With

- [n8n](https://n8n.io/) — Workflow automation
- [nmap](https://nmap.org/) — Network scanning
- [Claude AI](https://anthropic.com/) — Security analysis
- [SerpAPI](https://serpapi.com/) — CVE lookups
- Docker on Raspberry Pi 3

---

## 📜 License

MIT — use it, modify it, scan your network, stay safe. 🔐
