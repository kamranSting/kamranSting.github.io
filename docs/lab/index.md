# Home Lab Overview

## Topology

The lab VMs share one VirtualBox Internal Network behind pfSense, separate from my home network. A second internal network for Active Directory has its own pfSense interface. Since 12 November my own firewall rules block the lab LAN from it, and every block is logged. Details are in the [November update](pfsense-lab-network.md#update-november-2026-a-second-segment-and-my-own-rules).

| Component | Role | Status |
|---|---|---|
| VirtualBox | Hypervisor | Running |
| pfSense CE | Firewall, router and DHCP for the lab LAN | Running |
| Kali Linux | Attacker box for generating test activity | Running |
| Metasploitable2 | Deliberately vulnerable target | Running |
| Windows 11 + Sysmon | Endpoint telemetry | Running |
| Ubuntu + Splunk | SIEM | Running |
| Windows Server 2022 | Will be the domain controller, on its own segment | Installed, takes its time from pfSense, not yet a domain controller |

I picked Splunk over Security Onion because Security Onion needs about 16 GB of RAM, which doesn't fit in 32 GB alongside an AD lab.

!!! warning "Publishing rule"
    Write-ups never include real public IPs, passwords, API keys or anything from outside the lab.

## Write-ups

- [Building an isolated home lab with pfSense and Kali](pfsense-lab-network.md)
- [Can I trust the timestamps in my lab SIEM?](timestamps-clock-problem.md)
- [SMB password guessing: run it, find it, detect it](smb-password-guessing.md)
- [Hunting in BOTS v1: from a scanner to a quieter second source](agentphp-second-ip.md)
