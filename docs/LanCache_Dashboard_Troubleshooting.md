# Lancache-Dashboard Troubleshooting Guide

This guide documents the common hurdles and configurations required to get `lancache-dashboard` running smoothly in a Dockerized homelab, specifically when dealing with volume mapping, log ingestion, and firewall configurations.

## Overview
The `lancache-dashboard` is an excellent tool for monitoring your cache traffic, but it relies on precise file pathing and network visibility to function correctly. This guide outlines the fixes for common issues encountered during setup.

---

## 1. Environment Configuration

### Common `docker-compose.yml` Template
Use the following structure as a baseline. Ensure you replace `/path/to/your/...` with the actual absolute paths on your host machine.

```yaml
version: '3.8'
services:
  lancache-dashboard:
    image: lancache/lancache-dashboard:latest
    container_name: lancache-dashboard
    ports:
      - "3000:3000"
    volumes:
      # Map host log directory to container log directory
      - /path/to/your/logs:/data/logs
      # Isolate the database directory to prevent SQLITE_CANTOPEN errors
      - /path/to/your/db:/data/db
    environment:
      - TZ=America/New_York
    restart: unless-stopped

```

## 2. Troubleshooting & Common Gotchas
SQLITE_CANTOPEN Errors
* The Issue: The container lacks the permissions to write to the SQLite database file.

* The Fix: Ensure your volume mapping points to a dedicated directory for the database. Verify that the user running the Docker daemon has ownership and read/write permissions for that directory on the host.

YAML Syntax & Formatting
* The Issue: Incorrect indentation or syntax can cause services to fail silently or crash on start.

* The Fix: Always use a YAML linter before deploying. Ensure that service definitions are correctly nested under the services key.

Log Ingestion Pathing
* The Issue: Dashboard not displaying data even though services are running.

* The Fix: Confirm that the host directory mapped to /data/logs is the exact location where your Lancache service is actively writing its logs.

UniFi Firewall & Network Connectivity
If you are running a UniFi network, the dashboard port (default 3000) may be blocked by inter-VLAN firewall rules.

* The Fix: In the UniFi Controller, navigate to Settings > Security > Firewall & Security.

* Action: Ensure your "Allow" rule for port 3000 is placed above any default "Drop" or "Block" rules. Rule ordering is critical; if the block rule is higher, the traffic will never reach the container.

## 3. Contributing & Feedback
If you encountered a different error or have a refinement for this setup, feel free to open a Pull Request or create an Issue on this repository.

This guide was compiled based on common deployment experiences in a Proxmox/Docker-based homelab environment.
