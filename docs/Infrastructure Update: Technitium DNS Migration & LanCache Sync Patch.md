## Infrastructure Update: Technitium DNS Migration & LanCache Sync Patch

**Date:** July 7, 2026  
**Target Environment:** LANCache VM & Technitium DNS LXC  

### Overview
Following the migration and update of the Technitium DNS LXC instance, its static IP address changed from `192.168.1.251` to `192.168.1.35`. This change disrupted the automated Python pipeline responsible for injecting LanCache intercept zones into the DNS server via the Technitium API, causing client traffic to bypass the local cache.

---

### Issues Identified & Resolved

#### 1. Hardcoded API Endpoints
The automation scripts (`sync_zone.py`, `dns_zones_cleanup.py`, and `test_technitium_api.py`) had the legacy `.251` IP address hardcoded. 
* **Fix:** Updated the `TECHNITIUM_URL` and `TECHNITIUM_IP` variables across all files to point to the new LXC target: `http://192.168.1.35:5380`.

#### 2. False-Alarm Empty Zone Array Exit
On a fresh or wiped DNS instance, fetching existing zones returns an empty list (`[]`). The original script logic evaluated an empty list as a connection failure (`if not existing_zones:`) and aborted execution.
* **Fix:** Patched `sync_zone.py` to differentiate between an explicit connection error (`None`) and an empty zone array (`[]`), allowing initial provisions to execute seamlessly.

#### 3. Dnsmasq Rule String Parsing Error
The upstream `technitium_lancache_rules.txt` file is structured in a dnsmasq block format (`address=/domain/ip`). The core sync engine originally expected space-separated parts (`ip domain`), causing it to silently skip all incoming lines due to index mismatches.
* **Fix:** Refactored the core parsing loop inside `sync_zone.py` to natively handle both standard space-separated files and dnsmasq-style forward-slash formats cleanly.

---

### Core Script Patch (Parsing Logic)

The string parsing loop inside `sync_zone.py` was updated to ensure backward compatibility while adding robust handling for dnsmasq configuration strings:

```python
success_count = 0
for line in lines:
    line = line.strip()
    # Skip empty lines or comments
    if not line or line.startswith("#"):
        continue

    # Parse dnsmasq format: address=/domain/ip
    if line.startswith("address="):
        parts = line.split('/')
        if len(parts) >= 3:
            domain = parts[1]
        else:
            continue
    else:
        # Fallback to standard space-separated format just in case
        parts = line.split()
        if len(parts) == 2:
            _, domain = parts
        else:
            continue

    # Provisioning engine calls remain unchanged below...

```

---

### Verification & Performance Metrics

After executing the patched script, **25 LanCache intercept zones** were successfully mapped and pushed to the new Technitium API endpoint.

A post-change `nslookup` on a client machine confirms localized caching interception is fully functional:

```cmd
C:\Users\ryanm> nslookup lancache.steamcontent.com 192.168.1.35
Server:  UnKnown
Address:  192.168.1.35

Non-authoritative answer:
Name:    lancache.steamcontent.com
Address: 192.168.1.132

```

During live game download testing, container statistics verified full local throughput:

* **`lancache-monolithic` CPU:** ~46.84%
* **Network I/O:** 3.03 GB In / 3.17 GB Out (Active Cache Hits/Writes)

```

```
