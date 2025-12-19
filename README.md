# SimpleX Private Infrastructure on Raspberry Pi
## Tor-Only SMP + XFTP Server with Multi-Instance Private Routing

A battle-tested, reproducible guide to deploy your own **high-security SimpleX messaging infrastructure** on a Raspberry Pi using Tor v3 hidden services.

> **Version:** 0.6 (19. December 2025)  
> **Tested on:** Raspberry Pi 4 (4GB), Raspberry Pi OS Lite 64-bit (Bookworm)  
> **SimpleX Version:** 6.4.5.1

---

## About This Project

This guide provides a **complete, self-hosted messaging infrastructure** designed for individuals and organizations requiring maximum privacy and metadata protection. Unlike traditional messaging solutions, this setup ensures:

- **Zero clearnet exposure** – All traffic routes exclusively through Tor v3 onion services
- **No user identifiers** – SimpleX uses no phone numbers, emails, or usernames
- **Full infrastructure control** – You own and operate every component
- **Private Message Routing** – Multi-hop routing prevents any single server from seeing sender AND recipient
- **Metadata resistance** – Traffic mixing across multiple servers defeats correlation analysis

### Who Is This For?

| Use Case | Why SimpleX + Tor? |
|----------|-------------------|
| **Journalists & Media** | Protect sources, secure editorial communications, defeat surveillance |
| **Whistleblowers** | Anonymous tip submission, secure document transfer |
| **Activists & NGOs** | Organize without exposing social graphs to adversaries |
| **Legal & Medical** | Client/patient confidentiality, privileged communications |
| **Families & Friends** | Private group communication without Big Tech surveillance |
| **Government & Authorities** | Secure internal communications, classified discussions |
| **Research & Academia** | Protect research subjects, secure collaboration |
| **Businesses** | Trade secrets, M&A discussions, competitive intelligence protection |

### Why Raspberry Pi?

This guide uses a Raspberry Pi as a **low-budget, high-security** reference platform:

| Advantage | Description |
|-----------|-------------|
| **Cost** | ~€50-80 total hardware investment |
| **Power** | 5-15W consumption, runs 24/7 for pennies |
| **Stealth** | Small form factor, easy to conceal or relocate |
| **Air-Gap Ready** | Can operate completely isolated from main networks |
| **Reproducible** | Identical setup across multiple locations |
| **Disposable** | SD card can be destroyed; device is replaceable |

> **Note:** This architecture scales to VPS, dedicated servers, or Kubernetes clusters. The Raspberry Pi version proves the concept at minimal cost before larger deployments.

### What You Will Build

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR RASPBERRY PI                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              10 × SMP SERVERS                       │   │
│   │         (Private Message Routing Pool)              │   │
│   │                                                     │   │
│   │   Each server = unique .onion address               │   │
│   │   Traffic distributed across all instances          │   │
│   │   No single point of metadata correlation           │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              1 × XFTP SERVER                        │   │
│   │         (Encrypted File Transfer)                   │   │
│   │                                                     │   │
│   │   20GB storage, files auto-expire                   │   │
│   │   Chunked, encrypted uploads/downloads              │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              TOR DAEMON                             │   │
│   │         (11 Hidden Services)                        │   │
│   │                                                     │   │
│   │   All services Tor-only, zero clearnet             │   │
│   │   v3 onion addresses (ed25519 crypto)              │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │   TOR NETWORK   │
                    └─────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ SimpleX Clients │
                    │  (iOS/Android/  │
                    │   Desktop)      │
                    └─────────────────┘
```

### Security Model

| Layer | Protection |
|-------|------------|
| **Transport** | Tor v3 hidden services (ed25519, no exit nodes) |
| **Application** | SimpleX double-ratchet E2EE + post-quantum resistance |
| **Metadata** | Private routing, traffic mixing, no user IDs |
| **Infrastructure** | Self-hosted, no third-party dependencies |
| **Network** | Firewall blocks all clearnet access to services |

---

## Table of Contents

### Core Setup (Required)
1. [Base setup](#1-base-setup)
2. [Install dependencies](#2-install-dependencies)
3. [Create SimpleX service user](#3-create-simplex-service-user)
4. [Install Tor](#4-install-tor)
5. [Configure Tor hidden services](#5-configure-tor-hidden-services)
6. [Build SimpleX from source](#6-build-simplex-from-source)
7. [Initialize SMP server](#7-initialize-smp-server)
8. [Initialize XFTP server](#8-initialize-xftp-server)
9. [Create systemd services](#9-create-systemd-services)
10. [Firewall (nftables)](#10-firewall-nftables)
11. [Verify everything](#11-verify-everything)
12. [Client setup](#12-client-setup)
13. [Troubleshooting](#13-troubleshooting)

### Optional Hardening (Recommended for High-Security)
- [Appendix A: Multi-SMP for Private Message Routing](#appendix-a-multi-smp-for-private-message-routing)
- [Appendix B: SSH over Tor](#appendix-b-ssh-over-tor) *(coming soon)*
- [Appendix C: Tor v3 Client Authorization](#appendix-c-tor-v3-client-authorization) *(coming soon)*
- [Appendix D: Vanguards](#appendix-d-vanguards) *(coming soon)*

### Roadmap
- [Future Development](#roadmap-future-development)

---

## 1. Base setup

Flash **Raspberry Pi OS Lite (64-bit)** and boot:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

---

## 2. Install dependencies

> **CRITICAL:** This includes `binutils-gold` which is required for the Haskell build.

```bash
sudo apt update
sudo apt install -y \
  curl wget gnupg2 lsb-release ca-certificates \
  git build-essential pkg-config \
  libssl-dev zlib1g-dev liblzma-dev libbz2-dev libgmp-dev \
  libffi-dev libncurses-dev \
  jq bc netcat-openbsd \
  nftables torsocks \
  binutils binutils-gold

# Verify gold linker is available
command -v ld.gold && ld.gold -v
```

> **Why binutils-gold?** GHC/Cabal uses `-fuse-ld=gold` for some packages. Without it, you get cryptic `cannot find 'ld'` errors during the build.

---

## 3. Create SimpleX service user

```bash
# Create service user (no login shell)
sudo useradd -r -s /usr/sbin/nologin -m -d /var/opt/simplex simplex

# Create directories
sudo mkdir -p /etc/opt/simplex
sudo mkdir -p /etc/opt/simplex-xftp
sudo mkdir -p /var/opt/simplex
sudo mkdir -p /var/opt/simplex-xftp
sudo mkdir -p /var/opt/simplex-xftp/files

# Set ownership BEFORE initialization
sudo chown -R simplex:simplex /etc/opt/simplex
sudo chown -R simplex:simplex /etc/opt/simplex-xftp
sudo chown -R simplex:simplex /var/opt/simplex
sudo chown -R simplex:simplex /var/opt/simplex-xftp

# Set permissions (simplex user must be able to enter directories)
sudo chmod 750 /etc/opt/simplex /etc/opt/simplex-xftp
sudo chmod 750 /var/opt/simplex /var/opt/simplex-xftp
sudo chmod 750 /var/opt/simplex-xftp/files
```

---

## 4. Install Tor

```bash
# Add Tor Project repository
curl -fsSL https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc \
  | sudo gpg --dearmor -o /usr/share/keyrings/tor-archive-keyring.gpg

CODENAME="$(. /etc/os-release && echo "$VERSION_CODENAME")"
echo "deb [signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org ${CODENAME} main" \
  | sudo tee /etc/apt/sources.list.d/tor.list

sudo apt update
sudo apt install -y tor deb.torproject.org-keyring

sudo systemctl enable --now tor@default
```

---

## 5. Configure Tor hidden services

```bash
sudo nano /etc/tor/torrc
```

Add at the end:

```bash
# === SimpleX SMP ===
HiddenServiceDir /var/lib/tor/simplex-smp/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5223

# === SimpleX XFTP ===
HiddenServiceDir /var/lib/tor/simplex-xftp/
HiddenServiceVersion 3
HiddenServicePort 443 127.0.0.1:443
```

> **Port architecture (simplified):**
> - SMP listens on **5223** only (avoids privileged port issues)
> - XFTP listens on **443** (with capability fix in systemd)
> - Each service has its own onion address, so no port conflicts

Restart Tor and get your onion addresses:

```bash
sudo systemctl restart tor@default
sleep 5

# Save these addresses!
SMP_ONION=$(sudo cat /var/lib/tor/simplex-smp/hostname)
XFTP_ONION=$(sudo cat /var/lib/tor/simplex-xftp/hostname)

echo "SMP:  $SMP_ONION"
echo "XFTP: $XFTP_ONION"
```

**Write these down!** You'll need them for initialization and client setup.

---

## 6. Build SimpleX from source

### 6.1 Install GHCup

```bash
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```

Answers when prompted:
- Base channel: `D`
- Pre-releases: `N`
- Add to PATH: `P`
- Install HLS: `N`
- Stack integration: `Y`

Load environment:

```bash
source ~/.ghcup/env
```

### 6.2 Install GHC and Cabal

```bash
ghcup install ghc 9.6.3
ghcup set ghc 9.6.3

ghcup install cabal 3.10.3.0
ghcup set cabal 3.10.3.0

# Verify
ghc --version
cabal --version
```

> **Ignore warnings** about newer versions. We use tested versions.

### 6.3 Build SimpleX

```bash
cd ~
git clone https://github.com/simplex-chat/simplexmq.git
cd simplexmq

git fetch --tags
LATEST_TAG=$(git describe --tags --abbrev=0)
echo "Building: $LATEST_TAG"
git checkout "$LATEST_TAG"

cabal update
cabal build exe:smp-server exe:xftp-server
```

> **This takes 45-60 minutes on a Pi 4.** Go get coffee. ☕

### 6.4 Install binaries

```bash
sudo install -m 0755 "$(cabal list-bin exe:smp-server)" /usr/local/bin/smp-server
sudo install -m 0755 "$(cabal list-bin exe:xftp-server)" /usr/local/bin/xftp-server

# Verify
smp-server -v
xftp-server -v
```

---

## 7. Initialize SMP server

> **CRITICAL:** Run as `simplex` user, not root!

```bash
SMP_ONION=$(sudo cat /var/lib/tor/simplex-smp/hostname)
sudo -u simplex smp-server init -n "$SMP_ONION"
```

### 7.1 Configure SMP

```bash
sudo nano /etc/opt/simplex/smp-server.ini
```

**Required changes:**

```ini
[TRANSPORT]
host: <your-smp-onion>.onion
port: 5223
socks_proxy: 127.0.0.1:9050
```

> **CRITICAL:** The `socks_proxy` setting is required for Private Message Routing. Without it, servers cannot forward messages to other .onion servers.

> **Use only ONE port (5223).** Multiple ports like `5223,443` cause client link format issues.

**Disable HTTPS web server** (find `[WEB]` section):

```ini
[WEB]
static_path: /var/opt/simplex/www
# https: 442
# cert: /etc/opt/simplex/web.crt
# key: /etc/opt/simplex/web.key
```

Or use sed:

```bash
# Disable HTTPS (handles both "https: on" and "https: 443" formats)
sudo sed -i 's/^https:.*/# &/' /etc/opt/simplex/smp-server.ini
sudo sed -i 's/^cert:/# cert:/' /etc/opt/simplex/smp-server.ini
sudo sed -i 's/^key:/# key:/' /etc/opt/simplex/smp-server.ini
sudo sed -i 's/^port: .*/port: 5223/' /etc/opt/simplex/smp-server.ini

# Enable SOCKS proxy for Tor (CRITICAL for Private Routing)
grep -q "^socks_proxy:" /etc/opt/simplex/smp-server.ini || \
  sudo sed -i '/^\[TRANSPORT\]/a socks_proxy: 127.0.0.1:9050' /etc/opt/simplex/smp-server.ini
```

> **Note:** The config may have `https: on` or `https: 443` depending on version. The sed command above handles both.

### 7.2 Fix permissions

```bash
# Ensure correct ownership
sudo chown -R simplex:simplex /etc/opt/simplex /var/opt/simplex

# Key file must be readable by simplex user
sudo chmod 600 /etc/opt/simplex/server.key
sudo chown simplex:simplex /etc/opt/simplex/server.key

# Verify
sudo ls -la /etc/opt/simplex/
```

All files should show `simplex simplex`.

---

## 8. Initialize XFTP server

> **CRITICAL:** Run as `simplex` user, not root!

```bash
XFTP_ONION=$(sudo cat /var/lib/tor/simplex-xftp/hostname)
sudo -u simplex xftp-server init -n "$XFTP_ONION" -p /var/opt/simplex-xftp/files -q 20gb
```

### 8.1 Configure XFTP

```bash
sudo nano /etc/opt/simplex-xftp/file-server.ini
```

Ensure:

```ini
[TRANSPORT]
port: 443
```

### 8.2 Fix permissions

```bash
# CRITICAL: Ensure files directory exists and has correct permissions
sudo mkdir -p /var/opt/simplex-xftp/files
sudo chown -R simplex:simplex /etc/opt/simplex-xftp /var/opt/simplex-xftp
sudo chmod 750 /var/opt/simplex-xftp/files
sudo chmod 600 /etc/opt/simplex-xftp/server.key

# Verify files directory exists
sudo ls -la /var/opt/simplex-xftp/
```

You should see:
```
drwxr-x--- simplex simplex files
```

---

## 9. Create systemd services

### 9.1 SMP service

```bash
sudo tee /etc/systemd/system/smp-server.service > /dev/null <<'EOF'
[Unit]
Description=SimpleX SMP Server
After=network.target tor@default.service
Requires=tor@default.service

[Service]
Type=simple
User=simplex
Group=simplex
ExecStart=/usr/local/bin/smp-server start
Restart=always
RestartSec=5

# Security hardening
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/opt/simplex
PrivateTmp=yes

[Install]
WantedBy=multi-user.target
EOF
```

### 9.2 XFTP service (with CAP_NET_BIND_SERVICE for port 443)

> **CRITICAL:** XFTP binds to port 443, which requires special capability for non-root users.

```bash
sudo tee /etc/systemd/system/xftp-server.service > /dev/null <<'EOF'
[Unit]
Description=SimpleX XFTP Server
After=network.target tor@default.service
Requires=tor@default.service

[Service]
Type=simple
User=simplex
Group=simplex
ExecStart=/usr/local/bin/xftp-server start
Restart=always
RestartSec=5

# CRITICAL: Allow binding to port 443 as non-root
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE

# Security hardening
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/opt/simplex-xftp
PrivateTmp=yes

[Install]
WantedBy=multi-user.target
EOF
```

### 9.3 Enable and start services

```bash
sudo systemctl daemon-reload
sudo systemctl enable smp-server xftp-server
sudo systemctl start smp-server xftp-server

# Check status
sudo systemctl status smp-server --no-pager
sudo systemctl status xftp-server --no-pager
```

Both should show `active (running)`.

---

## 10. Firewall (nftables)

```bash
sudo tee /etc/nftables.conf > /dev/null <<'EOF'
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
  chain input {
    type filter hook input priority 0; policy drop;

    iif lo accept
    ct state established,related accept

    # SSH from LAN only
    ip saddr 192.168.0.0/16 tcp dport 22 accept
    ip saddr 10.0.0.0/8 tcp dport 22 accept

    # Block SimpleX ports from clearnet
    tcp dport { 443, 5223 } drop

    ip protocol icmp accept
  }

  chain forward {
    type filter hook forward priority 0; policy drop;
  }

  chain output {
    type filter hook output priority 0; policy accept;
  }
}
EOF

sudo systemctl enable --now nftables
```

---

## 11. Verify everything

### 11.1 Service status

```bash
sudo systemctl is-active tor@default smp-server xftp-server
```

All should return `active`.

### 11.2 Ports listening

```bash
sudo ss -lntp | grep -E ':(443|5223)'
```

Expected:
```
LISTEN  *:5223   users:(("smp-server",...))
LISTEN  *:443    users:(("xftp-server",...))
```

### 11.3 Test via Tor

```bash
SMP_ONION=$(sudo cat /var/lib/tor/simplex-smp/hostname)
XFTP_ONION=$(sudo cat /var/lib/tor/simplex-xftp/hostname)

torsocks nc -zv "$SMP_ONION" 5223 && echo "✓ SMP OK"
torsocks nc -zv "$XFTP_ONION" 443 && echo "✓ XFTP OK"
```

### 11.4 Get server addresses

```bash
echo "=== SERVER ADDRESSES ==="
sudo journalctl -u smp-server --no-pager | grep "Server address:" | tail -1
sudo journalctl -u xftp-server --no-pager | grep "Server address:" | tail -1
```

Copy these for your SimpleX clients.

---

## 12. Client setup

### 12.1 Enable Tor in SimpleX app

- Android/iOS: Install Orbot, enable SOCKS proxy `127.0.0.1:9050`
- Desktop: Settings → Network → SOCKS proxy `127.0.0.1:9050`

### 12.2 Add your servers

1. Settings → Network & Servers → SMP Servers → Add
2. Paste: `smp://<fingerprint>:<password>@<onion>:5223`

3. Settings → Network & Servers → XFTP Servers → Add
4. Paste: `xftp://<fingerprint>@<onion>:443`

---

## 13. Troubleshooting

### Build error: `cannot find 'ld'`

**Cause:** Missing gold linker. GHC uses `-fuse-ld=gold`.

**Fix:**
```bash
sudo apt install binutils-gold
```

---

### SMP crash: `permission denied` on server.key

**Cause:** Key file owned by root or wrong permissions.

**Fix:**
```bash
sudo chown simplex:simplex /etc/opt/simplex/server.key
sudo chmod 600 /etc/opt/simplex/server.key
sudo chown -R simplex:simplex /etc/opt/simplex
```

---

### SMP crash: `Network.Socket.bind: permission denied` on port 443

**Cause:** Non-root user cannot bind to privileged ports (<1024).

**Fix:** Use port 5223 for SMP, or add capability:
```bash
# In systemd unit:
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
```

---

### Client link has comma: `onion:5223,443`

**Cause:** Multiple ports in config (`port: 5223,443`).

**Fix:** Use single port:
```bash
sudo sed -i 's/^port: .*/port: 5223/' /etc/opt/simplex/smp-server.ini
sudo systemctl restart smp-server
```

---

### `Unit smp-server.service not found`

**Cause:** Service file doesn't exist or no daemon-reload.

**Fix:**
```bash
# Create service file (see section 9)
sudo systemctl daemon-reload
sudo systemctl enable --now smp-server
```

---

### XFTP: `withFile: does not exist`

**Cause:** Files directory missing.

**Fix:**
```bash
sudo mkdir -p /var/opt/simplex-xftp/files
sudo chown -R simplex:simplex /var/opt/simplex-xftp
```

---

### Can't connect via Tor

**Checklist:**
1. Is Tor running? `systemctl status tor@default`
2. Do onion hostnames exist? `sudo cat /var/lib/tor/simplex-smp/hostname`
3. Does Tor port match service port?
   - `HiddenServicePort 5223 127.0.0.1:5223` → service must listen on 5223
   - `HiddenServicePort 443 127.0.0.1:443` → service must listen on 443

---

### Private Routing Error: `does not exist (Name or service not known)`

**Cause:** SMP servers cannot resolve .onion addresses for forwarding because SOCKS proxy is not configured.

**Symptom in logs:**
```
Error connecting: xxx.onion PCENetworkError (NEConnectError {connectError = "...does not exist (Name or service not known)"})
```

**Fix:** Enable SOCKS proxy for all SMP servers:
```bash
# Add to [TRANSPORT] section in smp-server.ini
socks_proxy: 127.0.0.1:9050

# Or use sed:
grep -q "^socks_proxy:" /etc/opt/simplex/smp-server.ini || \
  sudo sed -i '/^\[TRANSPORT\]/a socks_proxy: 127.0.0.1:9050' /etc/opt/simplex/smp-server.ini

sudo systemctl restart smp-server
```

---

## Quick reference commands

```bash
# Restart everything
sudo systemctl restart tor@default smp-server xftp-server

# View logs
sudo journalctl -u smp-server -f
sudo journalctl -u xftp-server -f

# Check ports
sudo ss -lntp | grep -E ':(443|5223)'

# Export addresses
echo "SMP:  $(sudo cat /var/lib/tor/simplex-smp/hostname)"
echo "XFTP: $(sudo cat /var/lib/tor/simplex-xftp/hostname)"
```

---

# Optional Hardening

The following appendices provide advanced security configurations for high-threat environments such as journalist networks, activist groups, or anyone requiring maximum metadata protection.

---

# Appendix A: Multi-SMP for Private Message Routing

> **Prerequisite:** Complete Sections 1-13 first.  
> **Difficulty:** Intermediate  
> **Time:** 30-45 minutes  
> **Result:** 10 SMP servers on a single Raspberry Pi with full Private Routing support

---

## A.1 What is Private Message Routing?

SimpleX Chat introduced **Private Message Routing** in v5.8 (enabled by default since v6.0) to protect sender IP addresses from recipient servers. It implements a **two-hop packet routing mechanism** similar to Tor's onion routing.

### The Problem It Solves

Without Private Routing:
```
Alice → [Alice's IP visible] → Bob's SMP Server → Bob
```

Bob's server (or Bob himself, if self-hosted) can see Alice's IP address when she sends messages.

### How Private Routing Works

With Private Routing enabled:
```
Alice → Forwarding Server → Destination Server → Bob
        (sees Alice's IP)   (sees Forwarding IP)
```

1. Alice's client encrypts the message for Bob's destination server
2. Wraps it in another encryption layer for a forwarding server
3. Forwarding server relays without knowing the final recipient's queue ID
4. Destination server receives from forwarding server, not Alice directly

**Result:** Per-packet anonymity. No single server sees both sender AND recipient.

---

## A.2 Why Multiple SMP Servers?

Running multiple SMP servers provides significant privacy advantages:

| Feature | 1 Server | 4+ Servers | 10 Servers |
|---------|----------|------------|------------|
| Private Routing | ❌ Limited | ✅ Functional | ✅ Optimal |
| Traffic Mixing | ❌ None | ✅ Basic | ✅ Strong |
| Metadata Correlation | High Risk | Reduced | Minimal |
| Single Point of Failure | Yes | Distributed | Highly Resilient |
| Queue Distribution | Centralized | Spread | Maximum Entropy |

### Traffic Mixing & Metadata Protection

With 10 servers, SimpleX clients:
- **Randomly allocate queues** across all available servers
- **Route messages through different server pairs** for each conversation
- **Prevent traffic analysis** by distributing patterns across multiple endpoints
- **Eliminate correlation** between incoming and outgoing traffic on any single server

Even if an adversary compromises one server, they only see a fraction of your network's traffic with no way to correlate it.

---

## A.3 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    RASPBERRY PI 4                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           │
│  │ SMP #1  │ │ SMP #2  │ │ SMP #3  │ │ SMP #4  │           │
│  │ :5223   │ │ :5224   │ │ :5225   │ │ :5226   │           │
│  │ onion1  │ │ onion2  │ │ onion3  │ │ onion4  │           │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘           │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           │
│  │ SMP #5  │ │ SMP #6  │ │ SMP #7  │ │ SMP #8  │           │
│  │ :5227   │ │ :5228   │ │ :5229   │ │ :5230   │           │
│  │ onion5  │ │ onion6  │ │ onion7  │ │ onion8  │           │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘           │
│  ┌─────────┐ ┌─────────┐ ┌─────────────────────┐           │
│  │ SMP #9  │ │ SMP #10 │ │      XFTP #1        │           │
│  │ :5231   │ │ :5232   │ │       :443          │           │
│  │ onion9  │ │ onion10 │ │      onionX         │           │
│  └─────────┘ └─────────┘ └─────────────────────┘           │
├─────────────────────────────────────────────────────────────┤
│                        TOR DAEMON                           │
│            11 Hidden Services → 11 .onion addresses         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │   TOR NETWORK   │
                    │   (3-hop relay) │
                    └─────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ SimpleX Clients │
                    │ (Private Routing│
                    │    enabled)     │
                    └─────────────────┘
```

**Port Mapping:**

| Instance | Local Port | Tor External | Onion Address |
|----------|------------|--------------|---------------|
| Original | 5223 | 5223 | unique .onion |
| SMP #2 | 5224 | 5223 | unique .onion |
| SMP #3 | 5225 | 5223 | unique .onion |
| SMP #4 | 5226 | 5223 | unique .onion |
| SMP #5 | 5227 | 5223 | unique .onion |
| SMP #6 | 5228 | 5223 | unique .onion |
| SMP #7 | 5229 | 5223 | unique .onion |
| SMP #8 | 5230 | 5223 | unique .onion |
| SMP #9 | 5231 | 5223 | unique .onion |
| SMP #10 | 5232 | 5223 | unique .onion |

Each server has its own Tor hidden service, so clients connect to `onion:5223` regardless of local port.

---

## A.4 Create Directories

```bash
# Create config and data directories for SMP 2-10
sudo mkdir -p /etc/opt/simplex-smp{2,3,4,5,6,7,8,9,10}
sudo mkdir -p /var/opt/simplex-smp{2,3,4,5,6,7,8,9,10}

# Set ownership
sudo chown -R simplex:simplex /etc/opt/simplex-smp{2,3,4,5,6,7,8,9,10}
sudo chown -R simplex:simplex /var/opt/simplex-smp{2,3,4,5,6,7,8,9,10}

# Verify
ls -la /etc/opt/ | grep simplex-smp
```

Expected: 9 directories (smp2 through smp10), all owned by `simplex:simplex`.

---

## A.5 Configure Tor Hidden Services

Add to `/etc/tor/torrc`:

```bash
sudo nano /etc/tor/torrc
```

Append:

```bash
# === SimpleX SMP Multi-Instance (2-10) ===
HiddenServiceDir /var/lib/tor/simplex-smp2/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5224

HiddenServiceDir /var/lib/tor/simplex-smp3/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5225

HiddenServiceDir /var/lib/tor/simplex-smp4/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5226

HiddenServiceDir /var/lib/tor/simplex-smp5/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5227

HiddenServiceDir /var/lib/tor/simplex-smp6/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5228

HiddenServiceDir /var/lib/tor/simplex-smp7/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5229

HiddenServiceDir /var/lib/tor/simplex-smp8/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5230

HiddenServiceDir /var/lib/tor/simplex-smp9/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5231

HiddenServiceDir /var/lib/tor/simplex-smp10/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5232
```

Restart Tor and collect onion addresses:

```bash
sudo systemctl restart tor@default
sleep 5

# Get all onion addresses
echo "=== ONION ADDRESSES ==="
for i in 2 3 4 5 6 7 8 9 10; do
  echo "SMP$i: $(sudo cat /var/lib/tor/simplex-smp$i/hostname)"
done
```

**Save these addresses!**

---

## A.6 Initialize All SMP Servers

The `smp-server` binary uses hardcoded paths. We use a symlink trick to initialize each instance.

**For each instance (2-10), repeat:**

```bash
# Example for SMP2 - repeat for 3,4,5,6,7,8,9,10
INSTANCE=2
ONION="your-smp2-onion.onion"  # Replace with actual onion
PASSWORD="YourSecurePassword"

# Backup original paths
sudo mv /etc/opt/simplex /etc/opt/simplex-backup 2>/dev/null || true
sudo mv /var/opt/simplex /var/opt/simplex-backup 2>/dev/null || true

# Create symlinks
sudo ln -s /etc/opt/simplex-smp$INSTANCE /etc/opt/simplex
sudo ln -s /var/opt/simplex-smp$INSTANCE /var/opt/simplex

# Initialize
sudo -u simplex smp-server init -n "$ONION" --password "$PASSWORD" -y

# Remove symlinks
sudo rm /etc/opt/simplex /var/opt/simplex

# Restore original
sudo mv /etc/opt/simplex-backup /etc/opt/simplex 2>/dev/null || true
sudo mv /var/opt/simplex-backup /var/opt/simplex 2>/dev/null || true
```

---

## A.7 Configure Ports, HTTPS, and SOCKS Proxy

```bash
# Configure all instances
for i in 2 3 4 5 6 7 8 9 10; do
  PORT=$((5222 + i))  # 5224, 5225, 5226, etc.
  echo "Configuring SMP$i on port $PORT"
  
  # Set port
  sudo sed -i "s/^port: .*/port: $PORT/" /etc/opt/simplex-smp$i/smp-server.ini
  
  # Disable HTTPS
  sudo sed -i 's/^https:.*/# &/' /etc/opt/simplex-smp$i/smp-server.ini
  
  # Enable SOCKS proxy (CRITICAL for Private Routing!)
  grep -q "^socks_proxy:" /etc/opt/simplex-smp$i/smp-server.ini || \
    sudo sed -i '/^\[TRANSPORT\]/a socks_proxy: 127.0.0.1:9050' /etc/opt/simplex-smp$i/smp-server.ini
done

# Also ensure original server has SOCKS proxy
grep -q "^socks_proxy:" /etc/opt/simplex/smp-server.ini || \
  sudo sed -i '/^\[TRANSPORT\]/a socks_proxy: 127.0.0.1:9050' /etc/opt/simplex/smp-server.ini

# Verify
echo "=== PORTS ==="
grep "^port:" /etc/opt/simplex/smp-server.ini
grep "^port:" /etc/opt/simplex-smp{2,3,4,5,6,7,8,9,10}/smp-server.ini

echo "=== SOCKS PROXY ==="
grep "socks_proxy" /etc/opt/simplex/smp-server.ini
grep "socks_proxy" /etc/opt/simplex-smp{2,3,4,5,6,7,8,9,10}/smp-server.ini
```

Expected: Ports 5223-5232, all with `socks_proxy: 127.0.0.1:9050`.

---

## A.8 Create systemd Template Unit

```bash
sudo tee /etc/systemd/system/smp-server@.service > /dev/null <<'EOF'
[Unit]
Description=SimpleX SMP Server (Instance %i)
After=network.target tor@default.service
Requires=tor@default.service

[Service]
Type=simple
User=simplex
Group=simplex
ExecStart=/usr/local/bin/smp-server start

# Map instance paths to default paths expected by smp-server
BindPaths=/etc/opt/simplex-smp%i:/etc/opt/simplex
BindPaths=/var/opt/simplex-smp%i:/var/opt/simplex

Restart=always
RestartSec=5

# Security hardening
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes

[Install]
WantedBy=multi-user.target
EOF
```

The magic is in `BindPaths=` – systemd remaps the directories so each instance thinks it's using the default paths.

---

## A.9 Enable and Start All Instances

```bash
sudo systemctl daemon-reload

# Enable all instances (2-10)
sudo systemctl enable smp-server@{2,3,4,5,6,7,8,9,10}

# Start all instances
sudo systemctl start smp-server@{2,3,4,5,6,7,8,9,10}

# Restart original to pick up SOCKS proxy config
sudo systemctl restart smp-server

# Check status
sudo systemctl status smp-server@{2,3,4,5,6,7,8,9,10} --no-pager | grep -E "(smp-server@|Active:)"
```

All should show `active (running)`.

---

## A.10 Verify All Servers

### Check ports:

```bash
sudo ss -lntp | grep smp-server | wc -l
```

Should show **10** (original + 9 new instances).

### Test Tor connectivity:

```bash
echo "=== TOR CONNECTIVITY TEST ==="
for i in "" 2 3 4 5 6 7 8 9 10; do
  if [ -z "$i" ]; then
    ONION=$(sudo cat /var/lib/tor/simplex-smp/hostname)
    NAME="Original"
  else
    ONION=$(sudo cat /var/lib/tor/simplex-smp$i/hostname)
    NAME="SMP$i"
  fi
  torsocks nc -zv "$ONION" 5223 2>/dev/null && echo "✓ $NAME OK" || echo "✗ $NAME FAIL"
done
```

All 10 should succeed.

---

## A.11 Client Configuration

### Add All Servers to SimpleX App

1. Open SimpleX Chat
2. Go to **Settings → Network & Servers → SMP Servers**
3. For each server, tap **+ Add Server** and paste:
   ```
   smp://FINGERPRINT:PASSWORD@ONION_ADDRESS:5223
   ```
4. **Enable "Use for new connections"** on ALL servers ✅

### Enable Private Routing

1. **Settings → Network & Servers → Private Message Routing**
2. Set **Private routing** to: `Always` or `Unknown servers`
3. Set **Allow downgrade** to: `Yes`

### Remove Default SimpleX Servers (Optional)

For maximum privacy, disable the default SimpleX Chat servers so all traffic routes through your infrastructure only.

---

## A.12 Resource Usage

After setup, check your Pi's load:

```bash
echo "=== SYSTEM STATUS ==="
uptime
free -h
ps aux | grep -E "(smp|xftp)" | grep -v grep | wc -l
```

**Expected with 10 SMP + 1 XFTP:**

| Resource | Usage |
|----------|-------|
| CPU | ~2-5% idle |
| RAM | ~400-500 MB |
| Load Average | < 0.5 |

A Raspberry Pi 4 (4GB) handles 10+ SMP servers easily.

---

## A.13 Security Considerations

### What This Setup Provides

✅ **IP Address Protection** – Private routing hides sender IPs  
✅ **Traffic Mixing** – Messages distributed across 10 servers  
✅ **Metadata Resistance** – No single server sees full traffic pattern  
✅ **Tor-Only Access** – Zero clearnet exposure  
✅ **Self-Hosted** – Full control over infrastructure  

### What This Does NOT Provide

❌ **Protection against global adversary** – Nation-state level traffic analysis  
❌ **Protection if ALL servers compromised** – Distribute across locations  
❌ **End-to-end encryption** – Handled by SimpleX protocol itself  

---

## A.14 Multi-SMP Quick Reference

```bash
# Start all instances
sudo systemctl start smp-server smp-server@{2,3,4,5,6,7,8,9,10}

# Stop all instances
sudo systemctl stop smp-server smp-server@{2,3,4,5,6,7,8,9,10}

# Restart all instances
sudo systemctl restart smp-server smp-server@{2,3,4,5,6,7,8,9,10}

# View logs for instance 5
sudo journalctl -u smp-server@5 -f

# Check all ports
sudo ss -lntp | grep smp-server

# Count running instances
sudo ss -lntp | grep smp-server | wc -l
```

---

# Appendix B: SSH over Tor

*Coming in v0.7*

---

# Appendix C: Tor v3 Client Authorization

*Coming in v0.8*

---

# Appendix D: Vanguards

*Coming in v0.9*

---

# Roadmap: Future Development

This project is actively developed. The following features are planned:

## Security Hardening

| Feature | Status | Description |
|---------|--------|-------------|
| **Multi-SMP Private Routing** | ✅ v0.6 | 10 SMP servers for traffic mixing |
| **SSH over Tor** | 🔜 v0.7 | Admin access only via .onion |
| **Tor v3 Client Authorization** | 🔜 v0.8 | Hidden services invisible without keys |
| **Vanguards** | 🔜 v0.9 | Guard discovery protection |
| **LUKS Full-Disk Encryption** | 📋 Planned | Protect data at rest |
| **Dead Man's Switch** | 📋 Planned | Auto-wipe on tampering |

## Anti-Abuse & Performance

| Feature | Status | Description |
|---------|--------|-------------|
| **Queue Creation Password** | ✅ v0.1 | Restrict who can create queues |
| **Proof-of-Work (PoW)** | 📋 Planned | CPU-based spam prevention |
| **Rate Limiting** | 📋 Planned | Connection/message limits |
| **Connection Limits** | 📋 Planned | HiddenServiceMaxStreams |

## Monitoring & Operations

| Feature | Status | Description |
|---------|--------|-------------|
| **Prometheus Metrics** | 📋 Planned | Server statistics export |
| **Grafana Dashboards** | 📋 Planned | Visual monitoring |
| **Web UI** | 📋 Planned | Browser-based admin panel |
| **Automated Backups** | 📋 Planned | Encrypted backup to remote |
| **Health Checks** | 📋 Planned | Automated alerting |

## Distribution & Deployment

| Feature | Status | Description |
|---------|--------|-------------|
| **Pre-built ARM64 Binaries** | 📋 Planned | Skip 60min build time |
| **GPG Signed Releases** | 📋 Planned | Verify authenticity |
| **Docker Images** | 📋 Planned | Container deployment |
| **Ansible Playbooks** | 📋 Planned | Automated setup |

## Testing & Security

| Feature | Status | Description |
|---------|--------|-------------|
| **Stress Testing Framework** | 📋 Planned | Load testing tools |
| **Penetration Testing Guide** | 📋 Planned | Security audit checklist |
| **Threat Model Documentation** | 📋 Planned | Detailed security analysis |

---

## Changelog

### v0.6 (Current)
- **ADDED:** Comprehensive introduction with use cases and threat model
- **ADDED:** SOCKS proxy configuration for Private Routing (CRITICAL fix)
- **ADDED:** Roadmap section with future development plans
- **FIXED:** Private Routing error "does not exist (Name or service not known)"
- **IMPROVED:** Troubleshooting section with Private Routing errors

### v0.5
- **ADDED:** Appendix A - Multi-SMP for Private Message Routing (10 servers)
- **ADDED:** Optional Hardening section structure

### v0.4
- **FIXED:** sed command now handles both `https: on` and `https: 443` formats
- **FIXED:** Explicit `mkdir` for files directory in section 8.2
- **ADDED:** Verification step for files directory

### v0.3
- **ADDED:** `binutils-gold` to dependencies (fixes linker error)
- **ADDED:** `CAP_NET_BIND_SERVICE` for XFTP (fixes port 443 binding)
- **CHANGED:** SMP uses port 5223 only (avoids comma in client link)
- **ADDED:** Explicit `chmod 600` for key files
- **ADDED:** Comprehensive troubleshooting section

### v0.2
- Fixed port conflict between SMP and XFTP

### v0.1
- Initial release

---

## License

This guide is released under **MIT**.

SimpleX software is licensed under **AGPL-3.0**.

---

## Contributing

Found a bug? Have an improvement? 

- Open an issue on GitHub
- Submit a pull request
- Contact via SimpleX (server addresses available on request)

---

(C) 2025 cannatoshi
