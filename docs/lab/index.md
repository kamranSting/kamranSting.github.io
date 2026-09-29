# Home Lab Overview

## Topology

| Component | Role | Status |
|---|---|---|
| VirtualBox | Hypervisor | Running |
| pfSense | Firewall / router between lab segments | Running |
| Kali Linux | Attacker box for generating test activity | Running |
| Windows Server (AD DS) | Domain controller | Planned |
| Windows client + Sysmon | Endpoint telemetry | Planned |
| CrowdSec | Behaviour-based blocking | Planned |
| Security Onion | NSM / SIEM | Planned |

!!! warning "Publishing rule"
    Write-ups never include real public IPs, passwords, API keys or anything from outside the lab.

## Write-ups

- [pfSense network segmentation](pfsense-segmentation.md)
