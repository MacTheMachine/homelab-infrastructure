# Proxmox Virtualization Environment

## Overview
This project documents my Proxmox VE implementation, which serves as the backbone for my home lab infrastructure. By leveraging both Virtual Machines (VMs) and Lightweight Linux Containers (LXC), I optimize resource allocation on a mini PC while maintaining high availability for core services.

## Architecture
- **Hypervisor:** Proxmox VE (Debian-based)
- **Deployment Strategy:** 
  - **VMs:** Reserved for resource-heavy or OS-specific applications (e.g., Splunk Enterprise, and LanCache).
  - **LXC:** Utilized for lightweight, efficient containerized services (e.g., Technitium DNS).

## Core Services

| Service | Type | Purpose |
| :--- | :--- | :--- |
| Chrony | PVE Host | NTP server for network-wide time synchronization |
| Technitium DNS | LXC | Local DNS resolution and block list services |
| LanCache | VM | Off-peak caching for game patches and updates |
| Splunk Enterprise | VM | Centralized log aggregation and security analysis |

## Resource Management
- **Networking:** Proxmox bridges configured to map VM/LXC traffic to specific network VLANs.

## Key Learnings
- Determing the pros and cons of deploying LXC containers versus VMs within Proxomx.
- Configuring bridge-based network segmentation to align with physical UniFi VLANs.
- Created firewall pinholes for allowing only Core Services ports to connect to other Unifi VLANs within network
