# LanCache + Technitium DNS Native Integration

This document details the configuration used to seamlessly route game CDN traffic (Steam, Battle.net, Xbox, etc.) to a local LanCache server using Technitium DNS.

By leveraging Technitium's native Block List engine rather than API calls or legacy forwarding apps, we achieve a highly stable, zero-overhead intercept layer.

## 🏗️ Architecture Overview

* **Primary DNS Server:** Technitium DNS (`192.168.1.251`)
* **LanCache Server:** Ubuntu VM / Docker Monolithic (`192.168.1.132`)
* **Mechanism:** The LanCache server generates a flat-file of domain rules and hosts them locally via HTTP. Technitium periodically fetches this file and applies it to its native blocking engine to route traffic to the cache.

## ⚠️ The Problem with Legacy API Methods

Previously, automation scripts attempted to use the Technitium `/api/zones/records/add` endpoint to inject thousands of game domains individually.

* **The API Bomb:** Rapid-fire API calls can cause the Technitium web service to throttle, generate massive flat-file error logs, and eventually crash the Web UI with `Internal Server Error 500`.
* **Formatting Clashes:** Legacy apps like "Advanced Forwarding" expect strict AdGuard syntax and will crash (`invalid character [61]`) when encountering standard Dnsmasq format (`address=/domain/ip`).

## 🛠️ The Solution: Native Blocklist Overrides

Technitium's native Blocking engine handles Dnsmasq-formatted wildcard strings natively. By feeding it a single URL containing all rules, we eliminate API overhead entirely.

### Step 1: Generate the Ruleset

On the LanCache server (`192.168.1.132`), use a Python script to compile the `uklans/cache-domains` repository into a single flat text file formatted for Dnsmasq.

Example output format (`technitium_lancache_rules.txt`):

```text
address=/steamcontent.com/192.168.1.132
address=/assets2.xboxlive.com/192.168.1.132
address=/blizzard.com/192.168.1.132

```

### Step 2: Serve the File Locally over HTTP

Technitium needs a network path to ingest the text file. Spin up a lightweight web server in the directory containing your output file.

Using a background Python HTTP server (Port 8000):

```bash
# Navigate to the directory containing technitium_lancache_rules.txt
cd /path/to/your/dns-automation

# Start a simple HTTP server in the background
nohup python3 -m http.server 8000 &

```

*Note: You can verify this is working by visiting `http://192.168.1.132:8000/technitium_lancache_rules.txt` in a web browser.*

### Step 3: Configure Technitium DNS

Instruct Technitium to pull this file and use it to hijack the routing.

1. Open the Technitium Web UI (`http://192.168.1.251:5380`).
2. Navigate to **Settings > Block Lists**.
3. Scroll down to **Allow / Block List URLs** and click **Add**.
4. Configure the entry:
* **URL:** `http://192.168.1.132:8000/technitium_lancache_rules.txt`
* **List Type:** `Block List / Overrides`


5. Click **Save** and then click **Update Now** to force an immediate pull.

## ✅ Verification

Flush the DNS cache on your client machine and run a lookup against a known CDN domain:

```cmd
C:\> ipconfig /flushdns
C:\> nslookup assets2.xboxlive.com

```

**Expected Result:**

```text
Server:  technitium.local
Address:  192.168.1.251

Name:    assets2.xboxlive.com
Address:  192.168.1.132

```

If the IP resolves to your LanCache server (`.132`) instead of the public internet, the override is successfully engaged.

---
