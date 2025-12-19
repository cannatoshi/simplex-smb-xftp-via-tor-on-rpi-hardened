# SimpleX on Raspberry Pi OS Lite (Tor-Only) — SMP + XFTP over Onion v3 (hardened version)

A battle-tested, reproducible guide to run your own private SimpleX infrastructure on a Raspberry Pi (ARM64) using Tor v3 hidden services & tools.

> **Version:** 0.4 (19. December 2025)  
> **Tested on:** Raspberry Pi 4, Raspberry Pi OS Lite 64-bit (Bookworm)

---

## What you will get

After completing this guide:

- **SMP server** reachable at: `smp://<fingerprint>:<password>@<SMP_ONION>:5223`
- **XFTP server** reachable at: `xftp://<fingerprint>@<XFTP_ONION>:443`

> **Threat model:** Nothing reachable from clearnet. All traffic via Tor v3 onion services.  
> **Admin access:** SSH via LAN only (or optionally via Tor later).

---

## Table of Contents

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
```

> **Use only ONE port (5223).** Multiple ports like `5223,443` cause client link format issues.

**Disable HTTPS web server** (find `[WEB]` section):

```ini
[WEB]
static_path: /var/opt/simplex/www
# https: on
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
ls -la /etc/opt/simplex/
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

## Appendix: Quick reference commands

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

## Changelog

### v0.4 (Current)
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

(C) by cannatoshi
