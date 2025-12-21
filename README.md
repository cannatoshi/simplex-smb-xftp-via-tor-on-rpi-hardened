# SimpleX Private Infrastructure on Raspberry Pi

## Tor-Only Closed User Group Messaging with Multi-Instance Private Routing

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Platform](https://img.shields.io/badge/Platform-Raspberry%20Pi%204-c51a4a.svg)](https://www.raspberrypi.com/)
[![Tor v3](https://img.shields.io/badge/Tor-v3%20Onion-7D4698.svg)](https://www.torproject.org/)
[![SimpleX](https://img.shields.io/badge/SimpleX-6.4.5.1-1a73e8.svg)](https://simplex.chat/)
[![Encryption](https://img.shields.io/badge/Encryption-E2EE%20+%20Post--Quantum-blue.svg)](#security-model)
[![Infrastructure](https://img.shields.io/badge/Infrastructure-Closed%20System-critical.svg)](#closed-user-group-architecture)
[![Self-Hosted](https://img.shields.io/badge/Self--Hosted-100%25-success.svg)](#about-this-project)
[![No Tracking](https://img.shields.io/badge/Tracking-None-brightgreen.svg)](#security-model)
[![Maintenance](https://img.shields.io/badge/Maintained-Actively-success.svg)](https://github.com/cannatoshi/simplex-smp-xftp-via-tor-on-rpi-hardened/commits/main)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen.svg)](#contributing)

A battle-tested, reproducible guide to deploy your own **closed-system, high-security SimpleX messaging infrastructure** on a Raspberry Pi using Tor v3 hidden services.

> **Version:** 0.7.1-alpha (21. December 2025)  
> **Tested on:** Raspberry Pi 4 (4GB), Raspberry Pi OS Lite 64-bit (Bookworm)  
> **SimpleX Version:** 6.4.5.1

---

## About This Project

This guide provides a **complete, self-hosted messaging infrastructure for closed user groups** – families, organizations, teams, or any group requiring **maximum privacy and operational security**.

Unlike using public SimpleX infrastructure, this setup creates an **isolated, invisible messaging network** where:

- **Zero clearnet exposure** – All traffic routes exclusively through Tor v3 onion services
- **No public server listing** – Your .onion addresses exist nowhere on the internet
- **Closed user group** – Only people you explicitly invite can communicate
- **Zero metadata leakage** – No third-party servers see your traffic patterns
- **Full infrastructure control** – You own and operate every component

---

## Closed User Group Architecture

### The Problem with Public Infrastructure

When using **public SimpleX servers** (default preset servers or Flux servers), your messaging infrastructure is:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PUBLIC SIMPLEX INFRASTRUCTURE                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   smp11.simplex.im ◄──── Publicly known address                     │
│   smp12.simplex.im ◄──── Can be monitored by adversaries            │
│   smp14.simplex.im ◄──── Traffic patterns observable                │
│   flux-smp-*.runonflux.io ◄── Third-party operated                  │
│                                                                     │
│   ⚠️  Adversary knows WHERE to look                                 │
│   ⚠️  Server operators can see connection metadata                  │
│   ⚠️  Hosting providers log IP addresses                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

Even with SimpleX's excellent encryption and privacy features, using public servers means:
- Adversaries **know which servers to monitor**
- Server operators (even trusted ones) **can see connection patterns**
- Hosting providers **may log transport metadata**
- Your traffic is **mixed with millions of other users** (which can be good or bad)

### The Closed System Solution

This guide creates a **completely isolated infrastructure**:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    YOUR CLOSED INFRASTRUCTURE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   abc123...xyz.onion ◄──── Address known only to your group         │
│   def456...xyz.onion ◄──── Not listed anywhere on the internet      │
│   ghi789...xyz.onion ◄──── Cannot be discovered by scanning         │
│   ... (10 servers)   ◄──── You control ALL infrastructure           │
│                                                                     │
│   ✅ Adversary doesn't know WHERE to look                           │
│   ✅ YOU are the only operator                                      │
│   ✅ No third-party sees any metadata                               │
│   ✅ Traffic patterns invisible to outside observers                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**The fundamental security advantage:** An adversary must first **discover that your infrastructure exists** before they can even attempt to monitor it. With Tor v3 hidden services, this is computationally infeasible.

### Server Configuration: Closed vs. Hybrid Mode

#### Closed Mode (Maximum Security)

In your SimpleX client, **disable ALL public servers**:

```
Settings → Network & Servers → SMP Servers:
  ☐ SimpleX Chat servers (DISABLED)
  ☐ Flux servers (DISABLED)
  ☑ Your .onion servers (ENABLED + "Use for new connections" ✓)

Settings → Network & Servers → XFTP Servers:
  ☐ SimpleX Chat servers (DISABLED)
  ☐ Flux servers (DISABLED)  
  ☑ Your .onion server (ENABLED + "Use for new connections" ✓)
```

**Result:** 
- All communication stays within your closed infrastructure
- No metadata leaks to any third party
- Only members of your group can communicate
- Maximum operational security

**Limitation:** You cannot communicate with users outside your group.

#### Hybrid Mode (Compatibility with Reduced Security)

If you need to communicate with **external SimpleX users**:

```
Settings → Network & Servers → SMP Servers:
  ☑ SimpleX Chat servers (ENABLED, but "Use for new connections" ✗)
  ☐ Flux servers (DISABLED)
  ☑ Your .onion servers (ENABLED + "Use for new connections" ✓)
```

**How this works:**
- **New connections** you create use **only your servers** (your queues stay private)
- **External users** can reach you via public servers (you add their server as "known")
- Your infrastructure remains hidden, but you can communicate outside

**Trade-off:** 
- ✅ You can message anyone on SimpleX network
- ⚠️ Public servers see some of your connection metadata
- ⚠️ Traffic to external contacts is partially visible

#### Server Mode Comparison

| Feature | Closed Mode | Hybrid Mode |
|---------|-------------|-------------|
| **External communication** | ❌ Only your group | ✅ Anyone on SimpleX |
| **Infrastructure visibility** | ✅ Completely hidden | ⚠️ Partially exposed |
| **Metadata to third parties** | ✅ Zero | ⚠️ Some (external contacts) |
| **Recommended for** | High-security groups | Mixed use cases |

---

## Who Is This For?

This infrastructure is designed for **closed user groups** requiring maximum operational security:

| Use Case | Why Closed Infrastructure? |
|----------|---------------------------|
| **Journalist Networks** | Protect sources, editorial communications invisible to adversaries |
| **Whistleblower Systems** | Tip submission infrastructure that cannot be discovered |
| **Activist Cells** | Organize without exposing existence of communication network |
| **Legal Teams** | Attorney-client privilege with infrastructure you control |
| **Medical Teams** | Patient data on systems with zero third-party access |
| **Family Privacy** | Private communication without Big Tech or government visibility |
| **Research Groups** | Protect subjects and collaboration from surveillance |
| **Corporate Security** | M&A, trade secrets on infrastructure competitors cannot find |
| **Government/Military** | Classified communications on controlled infrastructure |

### When NOT to Use This

This guide is **overkill** if you:
- Just want better privacy than WhatsApp (use SimpleX with default servers)
- Need to communicate with many external people (use hybrid mode at minimum)
- Don't have time to maintain infrastructure (use managed services)
- Trust SimpleX/Flux operators (their servers are well-run)

---

## Why Raspberry Pi?

This guide uses a Raspberry Pi as a **low-budget, high-security** reference platform:

| Advantage | Description |
|-----------|-------------|
| **Cost** | ~€50-80 total hardware investment |
| **Power** | 5-15W consumption, runs 24/7 for pennies |
| **Stealth** | Small form factor, easy to conceal or relocate |
| **Air-Gap Ready** | Can operate completely isolated from main networks |
| **Reproducible** | Identical setup across multiple locations |
| **Disposable** | SD card can be destroyed; device is replaceable |
| **Plausible Deniability** | "It's just a media server" |

> **Note:** This architecture scales to VPS, dedicated servers, or Kubernetes clusters. The Raspberry Pi version proves the concept at minimal cost before larger deployments.

---

## What You Will Build

```
┌─────────────────────────────────────────────────────────────────┐
│                    YOUR RASPBERRY PI                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │              10 × SMP SERVERS                           │   │
│   │         (Private Message Routing Pool)                  │   │
│   │                                                         │   │
│   │   Each server = unique .onion address                   │   │
│   │   Addresses known ONLY to your group                    │   │
│   │   Traffic distributed across all instances              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │              1 × XFTP SERVER                            │   │
│   │         (Encrypted File Transfer)                       │   │
│   │                                                         │   │
│   │   20GB storage, files auto-expire                       │   │
│   │   Chunked, encrypted uploads/downloads                  │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │              TOR DAEMON                                 │   │
│   │         (12 Hidden Services)                            │   │
│   │                                                         │   │
│   │   All services Tor-only, zero clearnet                  │   │
│   │   v3 onion addresses (ed25519 crypto)                   │   │
│   │   Addresses not discoverable by any scan                │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │   TOR NETWORK   │
                    │  (Your traffic  │
                    │   hidden among  │
                    │   all Tor users)│
                    └─────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │  YOUR CLIENTS   │
                    │  (Only people   │
                    │   you invited)  │
                    └─────────────────┘
```

---

## Security Model

| Layer | Protection |
|-------|------------|
| **Discovery** | Tor v3 onion = address cannot be found without knowing it |
| **Transport** | Tor hidden services (ed25519, no exit nodes) |
| **Application** | SimpleX double-ratchet E2EE + post-quantum resistance |
| **Metadata** | Private routing, traffic mixing, no user IDs |
| **Infrastructure** | Self-hosted, YOU are the only operator |
| **Network** | Firewall blocks all clearnet access to services |

### The Discovery Problem (Why Closed Systems Matter)

Traditional attacks on messaging require knowing **where** to look:

| Attack Type | Public Servers | Your Closed Infrastructure |
|-------------|----------------|---------------------------|
| **Traffic Analysis** | ✅ Possible (known endpoints) | ❌ Must find endpoints first |
| **Server Compromise** | ✅ Target known servers | ❌ Which .onion to target? |
| **Legal Requests** | ✅ Subpoena known operators | ❌ Operator unknown |
| **Network Monitoring** | ✅ Monitor known IPs | ❌ No IPs, only Tor traffic |

**The security advantage is not just encryption – it's invisibility.**

---

## Threat Model & Limitations

> **Transparency is security.** This section documents what this setup protects against – and what it doesn't.

### What This Setup Protects Against ✅

| Threat Actor | Protection Level | How It Works |
|--------------|------------------|--------------|
| **Mass surveillance** | ✅ Strong | Your infrastructure doesn't exist in any database |
| **Targeted server monitoring** | ✅ Strong | Adversary must discover your .onion addresses first |
| **ISP / Network observer** | ✅ Strong | All traffic is Tor-encrypted; destination hidden |
| **Third-party metadata** | ✅ Strong | No SimpleX/Flux servers see your patterns |
| **Server operator (SimpleX Ltd.)** | ✅ Strong | You ARE the operator; they see nothing |
| **Traffic correlation (external)** | ✅ Strong | 10 servers = 10 separate Tor circuits |

### What This Setup Does NOT Protect Against ❌

| Threat Actor | Protection Level | Why |
|--------------|------------------|-----|
| **Physical device access** | ❌ None | One device = one point of compromise |
| **Root compromise of Pi** | ❌ None | Attacker sees all 10 server instances |
| **You (the operator)** | ❌ None | You control all servers |
| **Insider threat** | ⚠️ Limited | Group member can leak .onion addresses |
| **Nation-state with Tor visibility** | ⚠️ Limited | Long-term traffic analysis possible |
| **Endpoint compromise** | ❌ None | Client device hacked = game over |

### The Single-Operator Consideration

All 10 SMP servers run on **one device** under **one operator** (you). This means:

**Advantages:**
- Simpler administration
- Lower cost
- Full control

**Limitations:**
- No protection from yourself (you can correlate everything)
- Single point of physical failure
- If compromised, ALL servers are compromised

**For higher security, consider:**
- Distributing servers across multiple physical locations
- Having different trusted people operate different servers
- Geographic distribution across jurisdictions

### Recommended Use Cases by Threat Level

| Threat Level | Recommended Setup | Example Users |
|--------------|-------------------|---------------|
| **Standard** | Closed Mode, single Pi | Families, small teams |
| **Elevated** | Closed Mode + Appendix B-D | NGOs, journalists |
| **High** | Multi-location + different operators | Activists in hostile states |
| **Extreme** | Air-gapped + multi-jurisdiction + burner devices | Conflict zones, whistleblowers |

---

## Table of Contents

### Core Setup (Required)
1. [Base setup](#1-base-setup)
2. [Install dependencies](#2-install-dependencies)
3. [Create SimpleX service user](#3-create-simplex-service-user)
4. [Install Tor](#4-install-tor)
5. [Configure Tor hidden services](#5-configure-tor-hidden-services)
6. [Install SimpleX binaries](#6-install-simplex-binaries)
7. [Initialize SMP server](#7-initialize-smp-server)
8. [Initialize XFTP server](#8-initialize-xftp-server)
9. [Create systemd services](#9-create-systemd-services)
10. [Firewall (nftables)](#10-firewall-nftables)
11. [Verify everything](#11-verify-everything)
12. [Client setup](#12-client-setup)
13. [Troubleshooting](#13-troubleshooting)

### Optional Hardening (Recommended for High-Security)
- [Appendix A: Multi-SMP for Private Message Routing](#appendix-a-multi-smp-for-private-message-routing)
- [Appendix B: SSH over Tor](#appendix-b-ssh-over-tor)
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

> **CRITICAL:** This includes `binutils-gold` which is required for the Haskell build (Option B).

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

# Verify gold linker is available (needed for Option B)
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

**⚠️ CRITICAL: Store these addresses securely!**

These .onion addresses are the **only way** to reach your infrastructure. They should be:
- Shared only with members of your closed group
- Stored in encrypted form (password manager, encrypted note)
- Never posted publicly or sent over insecure channels

---

## 6. Install SimpleX Binaries

You have **two options** to get the SimpleX server binaries:

| Option | Time | Skill Level | Recommended For |
|--------|------|-------------|-----------------|
| **A) Pre-built binaries** | ~2 minutes | Beginner | Most users |
| **B) Build from source** | ~60 minutes | Intermediate | Security auditors, custom builds |

---

### Option A: Download Pre-built Binaries (Recommended)

Download verified ARM64 binaries from our GitHub releases:

```bash
# Download binaries
wget https://github.com/cannatoshi/simplex-smp-xftp-via-tor-on-rpi-hardened/releases/download/v0.7.1-alpha/smp-server
wget https://github.com/cannatoshi/simplex-smp-xftp-via-tor-on-rpi-hardened/releases/download/v0.7.1-alpha/xftp-server
wget https://github.com/cannatoshi/simplex-smp-xftp-via-tor-on-rpi-hardened/releases/download/v0.7.1-alpha/SHA256SUMS

# Verify checksums (IMPORTANT!)
sha256sum -c SHA256SUMS
```

Expected output:
```
smp-server: OK
xftp-server: OK
```

> **⚠️ If verification fails, DO NOT USE the binaries!** Re-download or build from source.

Install the binaries:

```bash
# Install to system path
sudo install -m 0755 smp-server /usr/local/bin/smp-server
sudo install -m 0755 xftp-server /usr/local/bin/xftp-server

# Verify installation
smp-server -v
xftp-server -v
```

Expected output: `SMP server v6.4.5.1` and `XFTP server v6.4.5.1`

**Done!** Skip to [Section 7: Initialize SMP server](#7-initialize-smp-server).

---

### Option B: Build from Source

For those who want to verify the source code or make custom modifications.

#### 6.B.1 Install GHCup

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

#### 6.B.2 Install GHC and Cabal

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

#### 6.B.3 Build SimpleX

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

#### 6.B.4 Install binaries

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

### 7.1 Configure SMP for Private Routing (CRITICAL!)

> **⚠️ IMPORTANT:** The SOCKS proxy configuration for Private Message Routing **MUST** be in the `[PROXY]` section, NOT in `[TRANSPORT]`! This is a common mistake that breaks Private Routing.

```bash
sudo nano /etc/opt/simplex/smp-server.ini
```

**Required changes in `[TRANSPORT]` section:**

```ini
[TRANSPORT]
host: <your-smp-onion>.onion
port: 5223
log_tls_errors: off
```

**Required changes in `[PROXY]` section (CRITICAL for Private Routing!):**

Find the `[PROXY]` section and configure:

```ini
[PROXY]
# SOCKS proxy for forwarding messages to destination servers
# MUST use IP address, not "localhost"!
socks_proxy: 127.0.0.1:9050

# For Tor-only .onion servers, use 'onion' mode
# 'onion' = SOCKS only for .onion destinations (default)
# 'always' = SOCKS for all destinations
socks_mode: onion
```

**Disable HTTPS web server** (find `[WEB]` section):

```ini
[WEB]
static_path: /var/opt/simplex/www
# https: 443
# cert: /etc/opt/simplex/web.crt
# key: /etc/opt/simplex/web.key
```

Or use these sed commands to automate:

```bash
# Set correct port
sudo sed -i 's/^port: .*/port: 5223/' /etc/opt/simplex/smp-server.ini

# Disable HTTPS
sudo sed -i 's/^https:.*/# &/' /etc/opt/simplex/smp-server.ini
sudo sed -i 's/^cert:/# cert:/' /etc/opt/simplex/smp-server.ini
sudo sed -i 's/^key:/# key:/' /etc/opt/simplex/smp-server.ini

# CRITICAL: Configure SOCKS proxy in [PROXY] section for Private Routing
# First, remove any existing socks_proxy entries (might be in wrong section)
sudo sed -i '/^socks_proxy:/d' /etc/opt/simplex/smp-server.ini

# Add socks_proxy and socks_mode in [PROXY] section
sudo sed -i '/^\[PROXY\]$/a socks_proxy: 127.0.0.1:9050\nsocks_mode: onion' /etc/opt/simplex/smp-server.ini
```

**Verify the configuration:**

```bash
sudo grep -A5 "\[PROXY\]" /etc/opt/simplex/smp-server.ini | head -10
```

Expected output:
```
[PROXY]
socks_proxy: 127.0.0.1:9050
socks_mode: onion
```

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

    # SSH from LAN only (remove after completing Appendix B)
    ip saddr 192.168.0.0/16 tcp dport 22 accept
    ip saddr 10.0.0.0/8 tcp dport 22 accept

    # Block SimpleX ports from clearnet (only accessible via Tor)
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

### 12.2 Configure for Closed User Group (Recommended)

**Step 1: Disable public servers**

```
Settings → Network & Servers → SMP Servers:
  → SimpleX Chat servers → Disable (toggle off)
  → Flux servers → Disable (toggle off, if present)

Settings → Network & Servers → XFTP Servers:
  → SimpleX Chat servers → Disable (toggle off)
  → Flux servers → Disable (toggle off, if present)
```

**Step 2: Add your servers**

```
Settings → Network & Servers → SMP Servers → + Add Server
  → Paste: smp://<fingerprint>:<password>@<onion>:5223
  → Enable "Use for new connections" ✓

Settings → Network & Servers → XFTP Servers → + Add Server
  → Paste: xftp://<fingerprint>@<onion>:443
  → Enable "Use for new connections" ✓
```

**Step 3: Verify configuration**

Your server list should show:
- ✅ Your .onion servers: Enabled + "Use for new connections"
- ❌ SimpleX Chat servers: Disabled
- ❌ Flux servers: Disabled

### 12.3 Configure Private Message Routing

For maximum privacy with your closed infrastructure:

```
Settings → Network & Servers → Private Message Routing:
  → Private routing: Always
  → Allow downgrade: No
```

> **Note:** "Allow downgrade: No" is recommended for closed groups where all servers are under your control and properly configured.

### 12.4 Distribute to Group Members

Share server addresses **securely** with your group:
- In person (QR code scan)
- Via already-secure channel (existing SimpleX contact, Signal, encrypted email)
- Never via unencrypted email, SMS, or public channels

---

## 13. Troubleshooting

### Build error: `cannot find 'ld'`

**Cause:** Missing gold linker. GHC uses `-fuse-ld=gold`.

**Fix:**
```bash
sudo apt install binutils-gold
```

---

### Checksum verification failed

**Cause:** Download corrupted or tampered with.

**Fix:**
```bash
# Re-download
rm smp-server xftp-server SHA256SUMS
wget https://github.com/cannatoshi/simplex-smp-xftp-via-tor-on-rpi-hardened/releases/download/v0.7.1-alpha/smp-server
wget https://github.com/cannatoshi/simplex-smp-xftp-via-tor-on-rpi-hardened/releases/download/v0.7.1-alpha/xftp-server
wget https://github.com/cannatoshi/simplex-smp-xftp-via-tor-on-rpi-hardened/releases/download/v0.7.1-alpha/SHA256SUMS

# Verify again
sha256sum -c SHA256SUMS
```

If still failing, build from source (Option B) or report an issue on GitHub.

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

**Cause:** SMP servers cannot resolve .onion addresses for forwarding. This happens when:
1. `socks_proxy` is in the wrong section (`[TRANSPORT]` instead of `[PROXY]`)
2. `socks_proxy` is missing entirely
3. Using `localhost` instead of `127.0.0.1`

**Symptom in logs:**
```
Error connecting: xxx.onion PCENetworkError (NEConnectError {connectError = "...does not exist (Name or service not known)"})
```

**Fix:** Configure SOCKS proxy correctly in the `[PROXY]` section:

```bash
# Remove any existing socks_proxy entries (might be in wrong section)
sudo sed -i '/^socks_proxy:/d' /etc/opt/simplex/smp-server.ini

# Add correct configuration in [PROXY] section
sudo sed -i '/^\[PROXY\]$/a socks_proxy: 127.0.0.1:9050\nsocks_mode: onion' /etc/opt/simplex/smp-server.ini

# Restart server
sudo systemctl restart smp-server
```

**Verify:**
```bash
sudo grep -A3 "\[PROXY\]" /etc/opt/simplex/smp-server.ini
```

Should show:
```
[PROXY]
socks_proxy: 127.0.0.1:9050
socks_mode: onion
```

---

### Private Routing Error on Client: "Error connecting to forwarding server"

**Cause:** Server-side SOCKS proxy misconfiguration.

**Quick Client Workaround (temporary):**
```
Settings → Network & Servers → Private Message Routing:
  → Allow downgrade: Yes
```

This allows messages to be delivered directly if Private Routing fails.

**Permanent Fix:** Fix server configuration as described above, then set `Allow downgrade: No` again.

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

# Check SOCKS proxy config (should be in [PROXY] section!)
sudo grep -A3 "\[PROXY\]" /etc/opt/simplex/smp-server.ini

# Export addresses (KEEP THESE SECRET!)
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

## A.2 Why Multiple SMP Servers for a Closed Group?

Even within your closed infrastructure, running multiple SMP servers provides significant privacy advantages:

| Feature | 1 Server | 4+ Servers | 10 Servers |
|---------|----------|------------|------------|
| Private Routing | ❌ Limited | ✅ Functional | ✅ Optimal |
| Traffic Mixing | ❌ None | ✅ Basic | ✅ Strong |
| Internal Correlation | High | Reduced | Minimal |
| Queue Distribution | Centralized | Spread | Maximum Entropy |

### Traffic Mixing Within Your Group

With 10 servers, even within your closed group:
- **Queues distributed randomly** across all servers
- **Different server pairs** used for each conversation
- **Traffic patterns harder to analyze** even for you as operator
- **Separation of concerns** – compromise of one server doesn't reveal all traffic

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
│            (All addresses known ONLY to your group)         │
└─────────────────────────────────────────────────────────────┘
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
echo "=== ONION ADDRESSES (KEEP SECRET!) ==="
for i in 2 3 4 5 6 7 8 9 10; do
  echo "SMP$i: $(sudo cat /var/lib/tor/simplex-smp$i/hostname)"
done
```

**Save these addresses securely!** Share only with your closed group members.

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

## A.7 Configure Ports and SOCKS Proxy (CRITICAL!)

> **⚠️ CRITICAL:** The SOCKS proxy configuration **MUST** be in the `[PROXY]` section for Private Message Routing to work! Placing it in `[TRANSPORT]` will NOT work.

```bash
# Configure all instances
for i in 2 3 4 5 6 7 8 9 10; do
  PORT=$((5222 + i))  # 5224, 5225, 5226, etc.
  echo "Configuring SMP$i on port $PORT"
  
  # Set port in [TRANSPORT]
  sudo sed -i "s/^port: .*/port: $PORT/" /etc/opt/simplex-smp$i/smp-server.ini
  
  # Disable HTTPS
  sudo sed -i 's/^https:.*/# &/' /etc/opt/simplex-smp$i/smp-server.ini
  
  # CRITICAL: Remove any existing socks_proxy entries (might be in wrong section)
  sudo sed -i '/^socks_proxy:/d' /etc/opt/simplex-smp$i/smp-server.ini
  
  # CRITICAL: Add SOCKS proxy in [PROXY] section (NOT [TRANSPORT]!)
  sudo sed -i '/^\[PROXY\]$/a socks_proxy: 127.0.0.1:9050\nsocks_mode: onion' /etc/opt/simplex-smp$i/smp-server.ini
done

# Also fix the original server if not already done
sudo sed -i '/^socks_proxy:/d' /etc/opt/simplex/smp-server.ini
sudo sed -i '/^\[PROXY\]$/a socks_proxy: 127.0.0.1:9050\nsocks_mode: onion' /etc/opt/simplex/smp-server.ini
```

**Verify the configuration is correct:**

```bash
echo "=== VERIFYING [PROXY] SECTION CONFIG ==="
echo "Original server:"
sudo grep -A3 "\[PROXY\]" /etc/opt/simplex/smp-server.ini | head -5

for i in 2 3 4 5 6 7 8 9 10; do
  echo "SMP$i:"
  sudo grep -A3 "\[PROXY\]" /etc/opt/simplex-smp$i/smp-server.ini | head -5
done
```

**Expected output for each server:**
```
[PROXY]
socks_proxy: 127.0.0.1:9050
socks_mode: onion
```

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

# Restart original to pick up any config changes
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

### Verify Private Routing works (check logs for errors):

```bash
# Wait 30 seconds, then check for SOCKS/routing errors
sleep 30 && sudo journalctl -u 'smp-server*' --since "1 min ago" | grep -iE "(error|fail)" | tail -10
```

If there are no errors (empty output), Private Routing is working correctly!

---

## A.11 Client Configuration for Closed Group

### Add All 10 Servers to SimpleX App

For each member of your closed group:

1. Open SimpleX Chat
2. Go to **Settings → Network & Servers → SMP Servers**
3. **Disable all public servers** (SimpleX Chat, Flux)
4. For each of your 10 servers, tap **+ Add Server** and paste:
   ```
   smp://FINGERPRINT:PASSWORD@ONION_ADDRESS:5223
   ```
5. **Enable "Use for new connections"** on ALL 10 servers ✅

### Enable Private Routing

1. **Settings → Network & Servers → Private Message Routing**
2. Set **Private routing** to: `Always`
3. Set **Allow downgrade** to: `No` (for closed groups, all servers support it)

### Verify Closed Configuration

Each group member's server list should show:
- ✅ Your 10 .onion servers: Enabled + "Use for new connections"
- ❌ SimpleX Chat servers: Disabled
- ❌ Flux servers: Disabled

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

✅ **Complete isolation** – All traffic stays within your closed infrastructure  
✅ **Traffic mixing** – Messages distributed across 10 servers  
✅ **Private routing** – No server sees both sender and recipient  
✅ **Discovery protection** – Adversary must find your .onion addresses first  
✅ **Self-hosted** – You are the only operator  

### What This Does NOT Provide

❌ **Multi-operator mixing** – All servers under your control  
❌ **Physical distribution** – All servers on one device  
❌ **Protection from you** – You can correlate all traffic if you choose  

> **See [Threat Model & Limitations](#threat-model--limitations)** for detailed analysis.

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

# Verify SOCKS config in [PROXY] section (CRITICAL!)
for i in "" 2 3 4 5 6 7 8 9 10; do
  if [ -z "$i" ]; then
    echo "=== Original ===" && sudo grep -A3 "\[PROXY\]" /etc/opt/simplex/smp-server.ini | head -5
  else
    echo "=== SMP$i ===" && sudo grep -A3 "\[PROXY\]" /etc/opt/simplex-smp$i/smp-server.ini | head -5
  fi
done

# Export all addresses (KEEP SECRET!)
echo "=== ALL SERVER ADDRESSES ==="
echo "SMP1: $(sudo cat /var/lib/tor/simplex-smp/hostname)"
for i in 2 3 4 5 6 7 8 9 10; do
  echo "SMP$i: $(sudo cat /var/lib/tor/simplex-smp$i/hostname)"
done
echo "XFTP: $(sudo cat /var/lib/tor/simplex-xftp/hostname)"
```

---

# Appendix B: SSH over Tor

> **Prerequisite:** Complete Sections 1-13 (Core Setup) first.  
> **Difficulty:** Beginner-Intermediate  
> **Time:** 15-20 minutes  
> **Result:** SSH access exclusively via Tor hidden service – zero clearnet exposure

---

## B.1 Why SSH over Tor?

Your SimpleX infrastructure is Tor-only and invisible. Your SSH access should be too.

### The Problem

```
┌──────────────┐                    ┌──────────────┐
│   Admin PC   │ ──── SSH:22 ────▶  │ Raspberry Pi │
│ (Real IP)    │    (Clearnet)      │ (Real IP)    │
└──────────────┘                    └──────────────┘
        ↓
   ISP logs connection
   Pi exposed on port 22
   Brute-force attacks possible
   Administration reveals Pi location
```

Even with your SimpleX services hidden, SSH over clearnet:
- Reveals the Pi's IP address to your ISP
- Creates a target for attackers
- Links your admin sessions to a physical location
- Undermines the "invisible infrastructure" concept

### The Solution

```
┌──────────────┐                    ┌──────────────┐
│   Admin PC   │ ══ Tor Circuit ══▶ │ Raspberry Pi │
│ (Anonymous)  │   (.onion:22)      │ (Invisible)  │
└──────────────┘                    └──────────────┘
        ↓
   No ISP visibility to destination
   No exposed ports on clearnet
   No brute-force surface
   Admin from anywhere anonymously
```

### Benefits

| Feature | Description |
|---------|-------------|
| **Zero clearnet exposure** | Port 22 never touches the internet |
| **Location independence** | Admin from any country, any network |
| **ISP invisibility** | Only Tor traffic visible, destination hidden |
| **NAT/Firewall bypass** | Works behind CGNAT, hotel WiFi, corporate firewalls |
| **Brute-force immunity** | Attackers can't find what doesn't exist |
| **Consistent security model** | Everything via Tor, nothing via clearnet |

---

## B.2 Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                      RASPBERRY PI                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   ┌────────────────────┐     ┌────────────────────────────┐   │
│   │      SSHD          │     │        TOR DAEMON          │   │
│   │   127.0.0.1:22     │◄────│   HiddenServicePort 22     │   │
│   │                    │     │   → xyzabc...onion:22      │   │
│   └────────────────────┘     └────────────────────────────┘   │
│                                                                │
│   ┌────────────────────────────────────────────────────────┐   │
│   │                     NFTABLES                           │   │
│   │           tcp dport 22 from clearnet → DROP            │   │
│   │           tcp dport 22 from 127.0.0.1 → ACCEPT         │   │
│   └────────────────────────────────────────────────────────┘   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼ (Tor only)
                    ┌─────────────────┐
                    │   TOR NETWORK   │
                    └─────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │   Admin Client  │
                    │   (torsocks)    │
                    └─────────────────┘
```

---

## B.3 Configure Tor Hidden Service for SSH

Add the SSH hidden service to your existing Tor configuration:

```bash
sudo nano /etc/tor/torrc
```

Append at the end:

```bash
# === SSH Administration ===
HiddenServiceDir /var/lib/tor/ssh/
HiddenServiceVersion 3
HiddenServicePort 22 127.0.0.1:22
```

Restart Tor and retrieve your SSH onion address:

```bash
sudo systemctl restart tor@default
sleep 5

# Get your SSH onion address
SSH_ONION=$(sudo cat /var/lib/tor/ssh/hostname)
echo "SSH Onion: $SSH_ONION"
```

**⚠️ CRITICAL: Save this address securely!** Without it, you cannot access your Pi remotely.

```bash
# Backup to a safe location (example: encrypted file)
echo "$SSH_ONION" | gpg -c > ssh_onion_backup.gpg
```

---

## B.4 Configure SSHD for Tor-Only Access

### B.4.1 Bind SSH to localhost only

Edit the SSH daemon configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Find and modify (or add) these lines:

```bash
# Bind ONLY to localhost - Tor will forward connections
ListenAddress 127.0.0.1

# Disable password authentication (key-only)
PasswordAuthentication no
PubkeyAuthentication yes

# Harden against attacks
PermitRootLogin no
MaxAuthTries 3
LoginGraceTime 30

# Disable unnecessary features
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no
```

> **Note:** `ListenAddress 127.0.0.1` is the key change. SSH will only accept connections from localhost, which means only Tor can reach it.

### B.4.2 Ensure you have SSH keys set up

Before restarting SSH, verify your public key is installed:

```bash
# Check authorized_keys exists
cat ~/.ssh/authorized_keys
```

If empty or missing, add your public key now:

```bash
# On your ADMIN MACHINE, generate key if needed:
ssh-keygen -t ed25519 -C "simplex-admin"

# Copy public key to Pi (while you still have LAN access!)
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@raspberry-pi-ip
```

### B.4.3 Restart SSH

```bash
sudo systemctl restart sshd

# Verify it's listening on localhost only
sudo ss -lntp | grep sshd
```

Expected output:
```
LISTEN  127.0.0.1:22   users:(("sshd",pid=...))
```

If you see `0.0.0.0:22` or `*:22`, the config change didn't apply. Check `/etc/ssh/sshd_config` again.

---

## B.5 Update Firewall

Update nftables to block all clearnet SSH:

```bash
sudo nano /etc/nftables.conf
```

Replace the entire file with:

```bash
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
  chain input {
    type filter hook input priority 0; policy drop;

    # Loopback - required for Tor→SSH and Tor→SimpleX
    iif lo accept
    
    # Established connections
    ct state established,related accept

    # BLOCK SSH from clearnet (Tor connects via loopback)
    tcp dport 22 drop

    # Block SimpleX ports from clearnet
    tcp dport { 443, 5223-5232 } drop

    # ICMP disabled for stealth (optional: uncomment to enable)
    # ip protocol icmp accept
  }

  chain forward {
    type filter hook forward priority 0; policy drop;
  }

  chain output {
    type filter hook output priority 0; policy accept;
  }
}
```

Apply the firewall:

```bash
sudo nft -f /etc/nftables.conf
sudo systemctl restart nftables
```

---

## B.6 Client Configuration

### B.6.1 Install Tor on Admin Machine

**Debian/Ubuntu:**
```bash
sudo apt install tor torsocks
sudo systemctl enable --now tor
```

**macOS:**
```bash
brew install tor torsocks
brew services start tor
```

**Windows:**
- Install [Tor Expert Bundle](https://www.torproject.org/download/tor/) or use WSL2

### B.6.2 Connect via torsocks

The simplest method:

```bash
SSH_ONION="your-ssh-onion-address.onion"
torsocks ssh user@$SSH_ONION
```

### B.6.3 SSH Config (Recommended)

Add to `~/.ssh/config` for convenience:

```bash
Host simplex-pi
    HostName your-ssh-onion-address.onion
    User pi
    IdentityFile ~/.ssh/id_ed25519
    ProxyCommand nc -X 5 -x 127.0.0.1:9050 %h %p
    ServerAliveInterval 60
    ServerAliveCountMax 3
    ConnectTimeout 120
```

Now connect with:

```bash
ssh simplex-pi
```

> **Note:** `ProxyCommand` uses netcat to route through Tor SOCKS proxy. This works on Linux/macOS. For Windows, use PuTTY with a SOCKS proxy configuration.

---

## B.7 Verify Tor-Only Access

### Test 1: Clearnet SSH should fail

From your admin machine, try connecting directly (without Tor):

```bash
# This should fail/timeout
ssh user@raspberry-pi-lan-ip
```

If this works, your firewall or SSHD config is wrong.

### Test 2: Tor SSH should work

```bash
# This should succeed
torsocks ssh user@your-ssh-onion.onion
```

### Test 3: Check from the Pi

```bash
# Verify SSH is localhost-only
sudo ss -lntp | grep :22

# Should show:
# LISTEN 127.0.0.1:22 ...
```

---

## B.8 Emergency Recovery

**"I locked myself out!"**

If you lose Tor access, you need physical access to the Pi:

1. Connect monitor + keyboard to Pi
2. Login locally (local console still works)
3. Fix configuration:
   ```bash
   # Temporarily allow LAN SSH
   sudo sed -i 's/ListenAddress 127.0.0.1/ListenAddress 0.0.0.0/' /etc/ssh/sshd_config
   sudo systemctl restart sshd
   ```
4. Fix your issue, then restore Tor-only config

### Backup Your Onion Keys

The SSH onion address is derived from keys in `/var/lib/tor/ssh/`. Back these up:

```bash
# On the Pi
sudo tar -czf /tmp/tor-ssh-keys.tar.gz -C /var/lib/tor ssh/
sudo chown $USER /tmp/tor-ssh-keys.tar.gz

# Transfer to admin machine
torsocks scp user@your-onion:/tmp/tor-ssh-keys.tar.gz ./

# Encrypt and store securely
gpg -c tor-ssh-keys.tar.gz
```

If you lose the SD card, you can restore these keys to get the same onion address.

---

## B.9 Performance Considerations

SSH over Tor is slower than clearnet SSH due to:
- 3-hop Tor circuit (6 relays round-trip)
- Onion service rendezvous overhead
- Tor network latency

**Typical latencies:**
- Clearnet SSH: 10-50ms
- Tor SSH: 200-800ms

**Tips for usability:**
- Enable SSH compression: Add `Compression yes` to SSH config
- Increase timeouts: `ServerAliveInterval 60` prevents disconnects
- Use tmux/screen: Persistent sessions survive disconnects
- Batch operations: Script repetitive tasks

---

## B.10 Security Notes

### What This Provides

✅ **No exposed ports** – SSH invisible on clearnet  
✅ **Location anonymity** – Admin IP hidden from Pi  
✅ **NAT traversal** – Works from any network  
✅ **Consistent security model** – Everything via Tor  

### What This Does NOT Provide

❌ **Key compromise protection** – If your SSH key leaks, attacker can connect  
❌ **Onion address secrecy** – If leaked, anyone with Tor can attempt connection  

### Recommended Additional Hardening

- **Appendix C** (Tor v3 Client Authorization) – Make onion invisible without auth key
- Use hardware security key (FIDO2) for SSH authentication
- Implement fail2ban (monitors auth failures even on localhost)

---

## B.11 Quick Reference

```bash
# Get SSH onion address
sudo cat /var/lib/tor/ssh/hostname

# Connect via Tor
torsocks ssh user@your-ssh-onion.onion

# Or with SSH config
ssh simplex-pi

# Check SSH is localhost-only
sudo ss -lntp | grep :22

# Restart SSH
sudo systemctl restart sshd

# Restart Tor
sudo systemctl restart tor@default

# Test Tor connectivity
torsocks nc -zv your-ssh-onion.onion 22
```

---

## B.12 Complete torrc Reference

After completing Appendices A and B, your `/etc/tor/torrc` should contain:

```bash
# === SSH Administration ===
HiddenServiceDir /var/lib/tor/ssh/
HiddenServiceVersion 3
HiddenServicePort 22 127.0.0.1:22

# === SimpleX SMP (Original) ===
HiddenServiceDir /var/lib/tor/simplex-smp/
HiddenServiceVersion 3
HiddenServicePort 5223 127.0.0.1:5223

# === SimpleX XFTP ===
HiddenServiceDir /var/lib/tor/simplex-xftp/
HiddenServiceVersion 3
HiddenServicePort 443 127.0.0.1:443

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

Total: **12 hidden services** (1 SSH + 10 SMP + 1 XFTP)

---

# Appendix C: Tor v3 Client Authorization

*Coming in v0.8*

This will add an additional layer of protection: even if someone discovers your .onion address, they cannot connect without the correct authorization key.

---

# Appendix D: Vanguards

*Coming in v0.9*

This will protect against guard discovery attacks that could be used to locate your hidden service.

---

# Roadmap: Future Development

This project is actively developed. The following features are planned:

## Security Hardening

| Feature | Status | Description |
|---------|--------|-------------|
| **Multi-SMP Private Routing** | ✅ v0.6 | 10 SMP servers for traffic mixing |
| **SSH over Tor** | ✅ v0.7 | Admin access only via .onion |
| **Pre-built ARM64 Binaries** | ✅ v0.7 | Skip 60min compile time |
| **Closed User Group Docs** | ✅ v0.7 | Documentation for isolated infrastructure |
| **SOCKS Proxy Fix** | ✅ v0.7.1 | Correct [PROXY] section configuration |
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
| **Pre-built ARM64 Binaries** | ✅ v0.7 | Skip 60min build time |
| **GPG Signed Releases** | 📋 Planned | Verify authenticity |
| **Docker Images** | 📋 Planned | Container deployment |
| **Ansible Playbooks** | 📋 Planned | Automated setup |

## Testing & Security

| Feature | Status | Description |
|---------|--------|-------------|
| **Stress Testing Framework** | 📋 Planned | Load testing tools |
| **Penetration Testing Guide** | 📋 Planned | Security audit checklist |
| **Threat Model Documentation** | ✅ v0.7 | Detailed security analysis |

---

## Changelog

### v0.7.1-alpha (Current)
- **FIXED:** CRITICAL - SOCKS proxy must be in `[PROXY]` section, NOT `[TRANSPORT]`
- **FIXED:** Private Message Routing now works correctly with Tor-only servers
- **UPDATED:** Section 7.1 with correct SOCKS proxy configuration
- **UPDATED:** Appendix A.7 with correct Multi-SMP SOCKS configuration
- **UPDATED:** Troubleshooting section with detailed Private Routing error fixes
- **ADDED:** Client-side Private Routing configuration instructions (Section 12.3)
- **ADDED:** Verification commands to check `[PROXY]` section configuration

### v0.7.0-alpha
- **ADDED:** Pre-built ARM64 binaries (Option A in Section 6)
- **ADDED:** SHA256 checksum verification
- **ADDED:** Closed User Group Architecture documentation
- **ADDED:** Server configuration guide (Closed vs. Hybrid mode)
- **ADDED:** Appendix B - SSH over Tor (Tor-only administration)
- **ADDED:** Threat Model & Limitations section
- **ADDED:** GitHub badges for project status
- **ADDED:** Discovery problem explanation
- **UPDATED:** Section 6 restructured with two installation options
- **UPDATED:** Entire guide refocused on closed user group concept
- **UPDATED:** Hidden Services count to 12 (added SSH)
- **UPDATED:** Firewall configuration for Tor-only access
- **UPDATED:** Troubleshooting with checksum verification errors

### v0.6
- **ADDED:** Comprehensive introduction with use cases
- **ADDED:** SOCKS proxy configuration for Private Routing
- **ADDED:** Roadmap section with future development plans
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

This guide is released under **AGPL-3.0**.

SimpleX software is licensed under **AGPL-3.0**.

Pre-built binaries are unmodified builds from [simplex-chat/simplexmq](https://github.com/simplex-chat/simplexmq).

---

## Contributing

Found a bug? Have an improvement? 

- Open an issue on GitHub
- Submit a pull request
- Contact via SimpleX (server addresses available on request)

---

*Built with 🔐 by [cannatoshi](https://github.com/cannatoshi)*

*"The best hiding place is one nobody knows exists."*
