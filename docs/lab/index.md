# Home Lab Overview

## Topology

All the lab VMs share one VirtualBox Internal Network behind pfSense, separate from my home network.

| Component | Role | Status |
|---|---|---|
| VirtualBox | Hypervisor | Running |
| pfSense CE | Firewall, router and DHCP for the lab LAN | Running |
| Kali Linux | Attacker box for generating test activity | Running |
| Metasploitable2 | Deliberately vulnerable target | Running |
| Windows 11 + Sysmon | Endpoint telemetry | Running |
| Ubuntu + Splunk | SIEM | Running |
| Windows Server (AD DS) | Domain controller on its own segment | Planned |

I picked Splunk over Security Onion because Security Onion needs about 16 GB of RAM, which doesn't fit in 32 GB alongside an AD lab.

!!! warning "Publishing rule"
    Write-ups never include real public IPs, passwords, API keys or anything from outside the lab.

## Write-ups

- [Building an isolated home lab with pfSense and Kali](pfsense-lab-network.md)
