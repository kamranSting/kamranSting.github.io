# pfSense lab network: isolation and a DNS fault

**Built:** mid-2026  
**SY0-701 domains:** 3 (Security Architecture), 4 (Security Operations)  
**Tools:** pfSense CE 2.8.1, VirtualBox, Kali Linux

## Goal

I wanted a lab where I could run attack tools and deliberately vulnerable machines without any of that traffic touching my home network. The lab VMs sit on a VirtualBox Internal Network behind pfSense, and their only route to the internet is through pfSense's WAN.

Everything inside the lab is on one LAN subnet for now. Splitting it up is on my list (see [Next steps](#next-steps)).

## Setup

| Item | Setting |
|---|---|
| pfSense VM | 1024 MB RAM, 2 vCPUs, 20 GB disk, OS type FreeBSD (64-bit), EFI off |
| pfSense WAN (em0) | VirtualBox NAT adapter, DHCP client, got `10.0.2.15/24` |
| pfSense LAN (em1) | VirtualBox Internal Network `intnet`, static `192.168.1.1/24` |
| DHCP on LAN | pfSense DHCP server, pool `192.168.1.100` to `192.168.1.200` |
| Kali | Adapter moved from NAT to Internal Network `intnet`, leased `192.168.1.101` |
| Web GUI | HTTPS (I declined the option to revert to HTTP) |

An Internal Network in VirtualBox is a virtual switch with no bridge to the host's network, so anything on `intnet` can only reach the outside world through pfSense. Later I added Metasploitable2, an Ubuntu VM running Splunk and a Windows 11 endpoint to the same `intnet` LAN. The Splunk VM also has a host-only adapter so I can reach its web interface from my PC.

## What I did

1. Installed pfSense CE from the AMD64 ISO, taking the default ZFS/GPT layout and leaving WAN on DHCP.
2. Removed the ISO (see [What went wrong](#what-went-wrong)) and booted into the installed system. WAN picked up `10.0.2.15/24`.
3. Added a second adapter to the VM as Internal Network `intnet`.
4. From the console, used option 1 to assign interfaces (WAN = em0, LAN = em1) and option 2 to give LAN a static `192.168.1.1/24` with no upstream gateway.
5. Turned on the DHCP server for LAN with a pool of `.100` to `.200`.
6. Moved Kali onto `intnet` and requested a lease.

```text
WAN (wan) -> em0 -> v4/DHCP4: 10.0.2.15/24
LAN (lan) -> em1 -> v4: 192.168.1.1/24
```

```text
eth0: offered 192.168.1.101 from 192.168.1.1
eth0: leased 192.168.1.101 for 7200 seconds
eth0: adding default route via 192.168.1.1
```

The lease output shows pfSense handing out the address and setting itself as Kali's default gateway.

The firewall runs on pfSense's default rules. LAN has the default rule allowing LAN to any destination, and WAN blocks all unsolicited inbound traffic, along with private and bogon source addresses. I haven't written custom rules yet.

## Testing

From Kali:

- `ip a` confirmed the interface was `eth0`.
- `sudo dhcpcd eth0` got a lease and a default route from pfSense.
- `ping 8.8.8.8` worked with 0% packet loss.
- `ping google.com` failed, which led to the DNS problem below.

## What went wrong

### DNS failed while the internet was reachable

`ping google.com` returned "Temporary failure in name resolution". Pinging `8.8.8.8` directly worked with no loss, so routing through pfSense, NAT and the internet connection were all fine and the fault had to be DNS. I set a nameserver on Kali by hand:

```text
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

After that, `ping google.com` resolved and replied in about 20 ms. It's only a workaround, since DHCP or a reboot can overwrite `/etc/resolv.conf`. The proper fix is to make sure pfSense hands out a DNS server in its DHCP offer.

### Smaller problems

- The VM kept booting back into the installer because the ISO was still attached and booted before the disk. I powered off, removed the ISO under Settings > Storage, and it booted the installed system. I now remove the ISO after every VM install.
- VirtualBox didn't recognise pfSense, so I set the OS type to FreeBSD (64-bit) by hand.
- Kali's `.vbox` file didn't show up in Import Appliance, which only lists `.ova` and `.ovf` files. Double-clicking the `.vbox` file opened it directly.
- This Kali image doesn't include `dhclient`. `sudo dhcpcd eth0` did the same job.

## What I learned

- To tell a DNS problem apart from a connectivity problem, ping a raw IP and then a hostname. If the IP works and the name doesn't, look at DNS before touching routing or the firewall.
- A VirtualBox Internal Network keeps the lab off my home network, with pfSense as the only way out. The machines inside can still all reach each other, so the lab is isolated but not segmented.
- pfSense's WAN default is implicit deny: nothing comes in unless a rule allows it.
- I watched DHCP hand out an address and a default route, and could read each step in the `dhcpcd` output.
- pfSense serves its web GUI over HTTPS by default, and I saw no reason to change that.

## Next steps

- Put the Active Directory lab on its own Internal Network behind a new pfSense interface (OPT1), with rules controlling what can cross between the two. At the moment Metasploitable2 sits on the same flat LAN as everything else.
- Write my own firewall rules, block something on purpose, and find it in the pfSense firewall logs.
- Hand out DNS through pfSense's DHCP settings instead of editing `/etc/resolv.conf` on each VM.
- Fix NTP on pfSense so the lab VMs' clocks stop drifting.
- Try Suricata or Snort on pfSense.
