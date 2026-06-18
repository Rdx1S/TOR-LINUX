# 🧅 Route All Traffic Through Tor on Kali Linux

<div align="center">

**English** · [Українська](README.uk.md)

Force **all** outgoing traffic on Kali Linux through the Tor network, and make it persist across reboots with a systemd service.

![Tor](https://img.shields.io/badge/Tor-7E4798?style=for-the-badge&logo=torproject&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)

</div>

> ⚠️ **For educational and authorized use only.** Routing all traffic through Tor changes your network behavior significantly. Use it responsibly and only on systems you own or are permitted to test.

---

## 📑 Table of Contents

- [1. Install Tor](#1-install-tor)
- [2. Configure Tor](#2-configure-tor)
- [3. Install Privoxy](#3-install-privoxy)
- [4. Enable IP forwarding](#4-enable-ip-forwarding)
- [5. Apply the changes](#5-apply-the-changes)
- [6. Redirect traffic with iptables](#6-redirect-traffic-with-iptables)
- [7–12. Persist across reboots](#7-12-persist-across-reboots)

---

## 1. Install Tor

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install tor
```

---

## 2. Configure Tor

Edit the Tor configuration file:

```bash
sudo nano /etc/tor/torrc
```

Uncomment the `SocksPort 9050` line (or set any other free port) as the SOCKS port.

---

## 3. Install Privoxy

```bash
sudo apt install privoxy
```

Edit the Privoxy configuration:

```bash
sudo nano /etc/privoxy/config
```

- Uncomment `listen-address localhost:8118` and replace `localhost` with `127.0.0.1`.
- Add the following line to the **end** of the file to forward traffic to Tor:

```
forward-socks5 / 127.0.0.1:9050 .
```

---

## 4. Enable IP forwarding

Edit the sysctl configuration:

```bash
sudo nano /etc/sysctl.conf
```

Uncomment the line:

```
net.ipv4.ip_forward=1
```

---

## 5. Apply the changes

```bash
sudo sysctl -p
```

---

## 6. Redirect traffic with iptables

Add a NAT rule that redirects outgoing TCP traffic through Privoxy:

```bash
sudo iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 8118
```

All your traffic should now be routed through Tor. Verify it by checking your public IP — it should belong to the Tor network:

```bash
curl ident.me
```

---

## 7–12. Persist across reboots

The iptables rule resets on reboot. Wrap it in a script and register it as a systemd service so it runs automatically.

**7. Create a script** in `/usr/local/bin` (the standard location for custom binaries):

```bash
sudo nano /usr/local/bin/tort.sh
```

**8. Add the following:**

```bash
#!/bin/bash
iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 8118
```

**9. Make it executable:**

```bash
sudo chmod ugo+x /usr/local/bin/tort.sh
```

**10. Create a systemd unit** that runs the script:

```bash
sudo systemctl edit --force --full script.service
```

This opens a text editor — paste:

```ini
[Unit]
Description=My Script Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/usr/local/bin/tort.sh

[Install]
WantedBy=multi-user.target
```

> 💡 In `ExecStart` you can specify either the path to the script or the raw `iptables` command directly. Using a script is more convenient — you can simply edit it to add more commands to run at boot.

**11. Enable the service** so it starts on boot:

```bash
sudo systemctl enable script
```

**12. If systemd doesn't see the service,** reload the unit definitions:

```bash
sudo systemctl daemon-reload
```
