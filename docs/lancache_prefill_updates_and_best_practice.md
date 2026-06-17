# LanCache Modular Prefill Architecture
This repository documents the architecture and configuration for a modular, containerized LanCache prefill system. This setup provides bandwidth-efficient, automated game updates while avoiding common DNS routing pitfalls.

## 1. Architecture Overview
This solution uses independent Docker containers for each gaming platform (Steam, Battle.net, Epic). This modularity allows for:

Independence: Issues with one launcher do not affect others.

Granular Scheduling: Staggered cron jobs avoid network congestion.

DNS Isolation: Dedicated DNS routing prevents infinite loops between the cache and external CDNs.

## 2. Deployment Configuration
The following docker-compose.yml structure defines the services:
```
YAML
services:
  # The central cache engine
  lancache:
    image: lancachenet/monolithic:latest
    container_name: lancache-monolithic
    restart: unless-stopped
    ports:
      - "TechnitiumDNS-IP-Address-Here:80:80"
      - "TechnitiumDNS-IP-Address-Here:443:443"
    environment:
      - CACHE_DISK_SIZE=680g
      - LANCACHE_IP="LanCache IP Address Here:
      - UPSTREAM_DNS=1.1.1.1;8.8.8.8
      - CACHE_INDEX_SIZE=250m
      - CACHE_DOMAIN_MICROSOFT=false
    volumes:
      - /home/mac/lancache/data:/data/cache
      - /home/mac/lancache/logs:/data/logs

  # Steam Prefill Service
  prefill-steam:
    image: ich777/lancache-prefill:latest
    dns:
      - 192.168.1.251
    environment:
      - ENABLE_STEAM=true
      - ENABLE_BN=false
      - ENABLE_EPIC=false
      - PREFILL_PARAMS_STEAM=--recent --force
      - CRON_SCHED_GLOBAL=0 7 * * *
    volumes:
      - /home/yourhomenetwork/lancache/prefill-steam:/config
      
  # Battle.net Prefill Service
  prefill-battlenet:
    image: ich777/lancache-prefill:latest
    dns:
      - 192.168.1.251
    environment:
      - ENABLE_STEAM=false
      - ENABLE_BN=true
      - ENABLE_EPIC=false
      - PREFILL_PARAMS_BN=--recent --force
      - CRON_SCHED_GLOBAL=30 7 * * *
    volumes:
      - /home/yourhomenetwork/lancache/prefill-bn:/config

  # Epic Prefill Service
  prefill-epic:
    image: ich777/lancache-prefill:latest
    dns:
      - 192.168.1.251
    environment:
      - ENABLE_STEAM=false
      - ENABLE_BN=false
      - ENABLE_EPIC=true
      - PREFILL_PARAMS_EPIC=--recent --force
      - CRON_SCHED_GLOBAL=0 8 * * *
    volumes:
      - /home/yourhomenetwork/lancache/prefill-epic:/config

  # Dashboard for monitoring traffic
  lancache-dashboard:
    image: lancache-dashboard:local
    container_name: lancache-dashboard
    ports:
      - "3000:3000"
    volumes:
      - /home/yourhomenetwork/lancache/logs:/logs:ro
      - ./dashboard-db:/app/data:rw
    restart: unless-stopped
```

## 3. Operational Best Practices
Avoid DNS Loops: Ensure prefill containers point specifically to your internal DNS (e.g., Technitium) which handles domain hijacking, while Technitium itself forwards non-game traffic to public upstream DNS.

Use --force: To bypass the container's built-in safety check that aborts when detecting public IP addresses, include --force in your PREFILL_PARAMS_... environment variables.

Persistence: Always map the /config directory to a persistent host volume to avoid losing login credentials on container restarts.

## 4. Verification & Troubleshooting
Manual Execution
To trigger a prefill job manually, use:
```
Bash
docker exec -it lancache-prefill-[service] /lancacheprefill/[Service]Prefill/[Service]Prefill prefill
```

Verification Commands
DNS Resolution Check: Verify the container is resolving to the local cache IP.
```
Bash
docker exec -it lancache-prefill-steam-1 nslookup lancache.steamcontent.com
Log Monitoring: Watch the cache in real-time for MISS/HIT traffic.

```
Bash
docker exec -it lancache-monolithic tail -f /data/logs/access.log
```
