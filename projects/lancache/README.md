# LanCache Implementation

## 1. Objective
The objective was to hava a centralized, high-speed local cache for game services like Steam, Battle.net, and Epic for the home network. 
This service would allow for game updates and patches to be time-shifted to a time when the home network is least used and use the internal 2.5 Gbps 
connectivity of the LAN network between them. This would eliminate the network latency spike and bottleneck of multiple systems reaching out and downloading
the same information through the same Internet connection and conserve bandwidth and improving network performance.

## 2. Technical Stack
* **Platform:** Proxmox VM (Ubuntu 24.04 LTS)
* **Software:** [LanCache.net](https://lancache.net/) (Docker-based)
* **Game Content Cache Domain Scripting** [GitHub uklans/cache-domains](https://github.com/uklans/cache-domains) (Python script)
* **Lightweight HTTP Server** Hosts the Game Content Cache Domain file that is updated to the Technitium DNS container (Python script)
* **Game Cache Pre-Fill Scripting** [GitHub tpill90](https://github.com/tpill90/steam-lancache-prefill) (Docker-based)
* **Resource Allocation:** 4 vCPU, 8GB RAM, 500GB ZFS Storage

## 3. Implementation Steps (The "How")
1. Provisioned VM and performed base OS hardening.
2. Configured static IP and DNS entries to redirect traffic to cache.
3. Deployed containers using Docker Compose.
4. Validated cache hits via `sniproxy` and logs via Splunk

## 4. Key Challenges & Resolution
## Challenge: DNS "Pre-fill" Loopback/Bottlenecks

**The Problem:** Your automation script (`push_cache_domains.py`) was failing due to API throttling and schema validation errors when attempting to batch-add thousands of CDN domains to Technitium DNS via its API. The cache-prefill container was getting trapped in a DNS resolution loop, trying to use the local Technitium DNS server to look up external CDN endpoints it needed to download assets.

**The Resolution:**

* **DNS Interception Optimization:** Moved away from the API-based record creation. Now, I compile domains into a consolidated `technitium_lancache_rules.txt` file and perform a **native configuration file import** into Technitium. This bypasses API throttling entirely and supports wildcard matching, significantly reducing database overhead.
* **Prefill Isolation:** Configured the `lancache-prefill` container to entirely bypass local DNS controls. By forcing the container to use external upstream DNS providers (e.g., Cloudflare/Google) for its initial lookups, it can now reach the public internet cleanly to warm the cache without triggering local DNS resolution loops.

## Challenge: Windows Delivery Optimization (DO) 403 Forbidden Errors

**The Problem:** LanCache successfully cached Steam, Battle.net, and Epic Games traffic, Windows Update clients consistently triggered `403 Forbidden` errors. This occurred because Microsoft’s Delivery Optimization (DO) service uses time-sensitive security tokens that desynchronize when intercepted by a transparent proxy.

**The Resolution:**

* **Using Windows Update built-in functionality:** Instead of forcing Windows Updates through the standard monolithic game cache, used the built-in Windows 11 feature of serving Windows Updates on the LAN locally.


## 5. Maintenance Checklist
* Using Splunk to review LanCache VM log details on a weekly basis
* Check and update LanCache monolithic and pre-fill Docker containers on a monthly basis
* Confirm LanCache VM disk usage and performance through the Proxmox host
