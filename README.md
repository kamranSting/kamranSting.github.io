# Blue Team Portfolio

My home lab and SOC practice, written up as I build it.

Live site: https://kamranSting.github.io

I am moving into cyber security from regulated legal and immigration work. I hold CompTIA Security+ and AZ-900. This site shows what I have built and tested myself.

## What is on the site

- A segmented lab network. pfSense runs a separate segment for Active Directory. Each firewall rule has a written reason, and I tested the block rule and saw it in the firewall log.
- Detection write-ups. I guessed passwords from Kali against my own Windows 11 VM, then found the attack in Splunk with the Windows logon failure events.
- Splunk Boss of the SOC v1. My method and reasoning for the questions I have worked through.
- Tickets. Each finding is written up the way a SOC analyst would log it.
- Study notes. Security+ SY0-701 notes and my TryHackMe log (SOC Level 1 in progress).

## Lab tools

VirtualBox, pfSense, Splunk, Windows 11, Windows Server 2022, Kali Linux.

## Build it locally

    pip install -r requirements.txt
    mkdocs serve

Then open http://127.0.0.1:8000
