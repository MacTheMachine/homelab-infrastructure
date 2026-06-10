# Splunk Log Aggregation Project

## Overview
This project focuses on centralizing logs from my home lab infrastructure to gain visibility into system health and security events using Splunk.

## The Challenge
Initially, log data was siloed across Proxmox hosts, UniFi gateway logs, and local containers, making it difficult to detect anomalies or troubleshoot performance issues efficiently.

## Implementation
- **Data Collection:** Configured Universal Forwarders on Proxmox nodes.
- **Log Ingestion:** Implemented log routing from UniFi hardware to the Splunk indexer.
- **Dashboarding:** Created custom dashboards to visualize network traffic patterns and system resource utilization.

## Outcomes
- Reduced troubleshooting time by providing a "single pane of glass" for all infrastructure events.
- Established baseline metrics for network traffic to identify potential security threats.

## Future Improvements
- Implement automated alerting for failed SSH login attempts.
- Integrate automated log rotation and storage optimization.
