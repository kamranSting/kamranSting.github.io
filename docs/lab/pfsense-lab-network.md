# Building an isolated home lab with pfSense and Kali

**Built:** mid-2026  
**SY0-701 domains:** 3 (Security Architecture), 4 (Security Operations)  
**Tools:** pfSense CE 2.8.1, VirtualBox, Kali Linux

This is the first stage of my home lab: a pfSense firewall in front of an internal VirtualBox network, with Kali as the first machine on it. I built it on my Windows PC while studying for CompTIA A+, Network+ and Security+ and working through TryHackMe.

I've tried to write it up the way I'd explain it in an interview. Where I relied on pfSense's defaults or haven't built something yet, I say so.

## Scope

The lab is one LAN segment behind pfSense. pfSense handles routing and DHCP for it, and I manage it over HTTPS. Kali is on the network and can reach the internet. The design is about isolation, keeping lab traffic away from my real home network. It isn't a multi-segment network.

There's one WAN and one LAN. I haven't written any firewall rules of my own, there's no second subnet or DMZ, and I haven't tested traffic between segments or reviewed the firewall logs. Those are all in [Next steps](#next-steps).

## Goal

I wanted somewhere to practise attacking, and later defending: running scans, starting deliberately vulnerable machines and eventually monitoring the traffic, without any of it landing on my home network.

The part of the design that does this is VirtualBox's Internal Network. An internal network is a virtual switch with no connection to the host's network card or my home LAN. The lab VMs can talk to each other and to pfSense, and the only way out is through pfSense's WAN interface.

That boundary works in one direction for now. Nothing on my home network can start a connection into the lab, because pfSense's WAN side blocks unsolicited inbound traffic. Going the other way, the default LAN rule allows traffic to any destination, and VirtualBox NAT sits behind the WAN, so a lab VM could probably still reach devices on my home network. Blocking that is on the list below.

## Architecture

```text
[Internet]
     |
[pfSense WAN  em0]  DHCP client -> 10.0.2.15/24   (VirtualBox NAT)
     |
[pfSense LAN  em1]  static 192.168.1.1/24, DHCP server (.100-.200)
     |
  intnet  (VirtualBox Internal Network)
     |
[Kali Linux]  192.168.1.101 (DHCP)
```

| Interface | VirtualBox adapter | Addressing | Role |
|---|---|---|---|
| WAN (em0) | NAT | DHCP client, `10.0.2.15/24` | Route out to the internet through VirtualBox NAT |
| LAN (em1) | Internal Network `intnet` | Static `192.168.1.1/24`, DHCP pool `.100` to `.200` | Gateway and DHCP for the lab |

A VM joins the lab by setting its own adapter to the same internal network name, `intnet`. pfSense's one LAN adapter serves the whole virtual switch, so I don't need a new pfSense interface for each machine. Kali was the first. Metasploitable2, an Ubuntu VM running Splunk and a Windows 11 endpoint have joined the same LAN since.

`10.0.2.15/24` is VirtualBox's default NAT range, so it's a private address. Nothing here shows a real public IP, my home router's address or a MAC address.

## pfSense VM

| Setting | Value |
|---|---|
| Version | CE 2.8.1 (the current stable release when I installed it) |
| RAM | 1024 MB |
| vCPUs | 2 |
| Disk | 20 GB |
| Guest OS type | FreeBSD (64-bit), set by hand because VirtualBox didn't detect it |
| Firmware | EFI off |

## Build steps

1. Installed pfSense CE from the AMD64 ISO. WAN came up as `em0` on DHCP. I kept the default ZFS/GPT layout and chose Install CE instead of the paid Plus path ("Retry Validation").
2. Got stuck in a boot loop on the first reboot and fixed it (see [Other problems](#other-problems)).
3. Added a second VirtualBox adapter on Internal Network `intnet`, which became the LAN.
4. In the pfSense console, used Assign Interfaces to set WAN = `em0` and LAN = `em1`.
5. Gave LAN a static `192.168.1.1/24` with no upstream gateway. Only WAN needs one.
6. Turned on the DHCP server for LAN with a pool of `192.168.1.100` to `192.168.1.200`.
7. Kept the web GUI on HTTPS when the console offered to switch it to HTTP.
8. Moved Kali onto `intnet`, brought its interface up with DHCP, and then ran into the DNS problem below.

## Firewall rules

I'm running pfSense's default rules. I didn't write them, but I can explain them:

- On LAN, the default "allow LAN to any" rule lets the lab make outbound connections.
- On WAN, all unsolicited inbound traffic is blocked, along with private and bogon source addresses. Nothing from the WAN side reaches the lab unless a rule allows it.

The WAN side is implicit deny in practice: anything not explicitly allowed gets dropped. I haven't written per-interface rules or looked through the firewall logs yet, so I'm not claiming least-privilege rule design or log analysis here. Writing rules and then reading the logs is first on my list.

## DNS troubleshooting

This was the part of the build I learned the most from, mostly because of how I narrowed it down.

Once Kali had a DHCP lease from pfSense, `ping google.com` failed:

```text
ping: google.com: Temporary failure in name resolution
```

Before blaming the firewall or the internet connection, I pinged a raw IP to separate name resolution from connectivity:

```text
$ ping 8.8.8.8
64 bytes from 8.8.8.8 ... 0% packet loss     # routing, NAT and internet fine
$ ping google.com
ping: google.com: Temporary failure in name resolution   # only DNS fails
```

The IP worked, so routing, NAT and the internet connection were fine and the problem had to be DNS. I pointed Kali at a nameserver by hand:

```text
$ echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
$ ping google.com    # resolves now, ~20 ms replies, 0% loss
```

I noted at the time that this is a workaround. `/etc/resolv.conf` is often managed automatically and can be overwritten by the next DHCP lease or a reboot. The lasting fix is to make sure pfSense includes a DNS server in its DHCP offer (Services > DHCP Server) so no one has to patch the file on each client.

What I'll reuse is the method. Test one layer at a time: if the IP works and the name doesn't, it's DNS, and there's no need to start pulling apart the firewall.

## Other problems

- **Boot loop after install.** The VM kept rebooting into the installer because the ISO was still attached and booted before the disk. I powered off, removed the ISO under Settings > Storage, and it booted the installed system. I now check this after every VM install.
- **No `dhclient` on Kali.** `sudo dhclient eth0` gave "command not found" on this image. `sudo dhcpcd eth0` worked and pulled the lease (`offered 192.168.1.101 from 192.168.1.1`, default route via `192.168.1.1`), which also confirmed pfSense was acting as DHCP server and gateway.
- **Kali's `.vbox` file didn't show in Import Appliance.** The importer only lists `.ova` and `.ovf` files. Double-clicking the `.vbox` file opened it, and switching the file filter to "All files" would have worked too.
- **VirtualBox didn't recognise pfSense.** I set the guest type to FreeBSD (64-bit) myself.

## What I learned, and the Security+ links

- **Isolation.** The Internal Network has no direct path to my home LAN, and the only way out is through pfSense. It's a small, concrete case of separating networks by trust level, although the lab is one flat segment inside.
- **Implicit deny.** pfSense's WAN side drops any inbound traffic that no rule allows.
- **DHCP, DNS and the default gateway.** I watched pfSense lease an address and set the default route, and I separated a DNS fault from a routing fault.
- **Secure defaults.** I kept HTTPS on the management interface.

I haven't claimed least privilege or logging, because I didn't configure either here. I'd rather defend a small claim than overstate a big one.

## Limitations

- One flat LAN, with no second segment or DMZ.
- Default firewall rules only.
- Lab VMs can probably reach my home network through the default LAN rule.
- No testing between segments and no firewall log review yet.
- No NTP in the lab, so clocks drift across VMs, which caused problems in later stages.

## Next steps

- Add a second Internal Network behind a new pfSense interface (OPT1) and write rules for what can cross between the two segments. After that I'll be able to answer firewall rule questions from experience.
- Add a LAN rule that blocks the lab from reaching my home network's address range, then confirm it from Kali.
- Write explicit LAN and OPT1 rules, then check the firewall logs to see what they block.
- Hand out DNS through pfSense's DHCP settings instead of editing `/etc/resolv.conf`.
- Try Suricata or Snort on pfSense.
- Point every VM at pfSense for time to stop the clock drift.
- Write up the later stages (Metasploitable2, the Windows 11 endpoint with Sysmon, and Splunk) as separate pages.

---

*Environment: Oracle VirtualBox on Windows, i7-11800H with 32 GB RAM. All addresses shown are private lab ranges. I used Claude as a build partner and to talk through troubleshooting. I carried out and checked every step myself and can explain why I did each one.*
