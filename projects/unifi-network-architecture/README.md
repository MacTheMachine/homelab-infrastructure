# UniFi Network Architecture

## Overview
This project documents the design and implementation of my home network, leveraging UniFi enterprise-grade hardware to achieve high availability and robust security.

## Network Topology
- **Gateway:** UniFi Cloud Gateway Max
- **Switching:** Unifi Flex 2.5G 5 port switch
- **VLAN Segmentation:**
  - `VLAN 1` : Management (Infrastructure)   
  - `VLAN 10`: Home Network (Trusted)
  - `VLAN 20`: Work Network (Isolated)
  - `VLAN 30`: IoT Devices (Untrusted)

## Key Configurations
- **Port Profiles:** Custom port tagging used to isolate laboratory traffic from standard home traffic.
- **Routing:** Inter-VLAN routing configured for secure communication between specific lab segments.

## Goals & Results
- **Goal:** Achieve logical separation between production and experimental network environments.
- **Result:** Successfully contained laboratory traffic within its own VLAN, reducing the impact of potential misconfigurations on the main home network.
