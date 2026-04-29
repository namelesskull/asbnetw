# asbnetw - Asep Bensin Super Network Tools

All-in-one WiFi network toolkit for Linux. Single binary, no runtime dependencies.

```
               _                _            
   __ _ ___| |__  _ __   ___| |___      __
  / _' / __| '_ \| '_ \ / _ \ __\ \ /\ / /
 | (_| \__ \ |_) | | | |  __/ |_ \ V  V / 
  \__,_|___/_.__/|_| |_|\___|\__| \_/\_/  
  v0.1.0
```

## Install

### 1. Download & install binary

```bash
sudo curl -L https://github.com/namelesskull/asbnetw/releases/download/v0.1.0/asbnetw-x86_64 \
  -o /usr/local/bin/asbnetw && sudo chmod +x /usr/local/bin/asbnetw
```

Verify:

```bash
asbnetw version
```

### 2. Install dependencies

**asbnetw** leverages common Linux system tools. Install only what you need based on the features you want to use.

#### Required (minimum for WiFi scanning)

```bash
# Arch / Manjaro
sudo pacman -S networkmanager iw nmap

# Debian / Ubuntu
sudo apt install network-manager iw nmap
```

#### WiFi cracking

```bash
# Arch / Manjaro
sudo pacman -S aircrack-ng

# Debian / Ubuntu
sudo apt install aircrack-ng

# Download wordlist (rockyou.txt, 14 million passwords)
sudo mkdir -p /usr/share/wordlists
sudo curl -L -o /usr/share/wordlists/rockyou.txt.gz \
  https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

#### Traffic sniffing, device kicking, MITM

```bash
# Arch / Manjaro
sudo pacman -S dsniff tcpdump mitmproxy

# Debian / Ubuntu
sudo apt install dsniff tcpdump mitmproxy
```

#### Port scanning & access probing

```bash
# Arch / Manjaro
sudo pacman -S nmap smbclient sshpass

# Debian / Ubuntu
sudo apt install nmap smbclient sshpass
```

#### Install everything at once

```bash
# Arch / Manjaro
sudo pacman -S networkmanager iw nmap aircrack-ng dsniff tcpdump mitmproxy smbclient sshpass

# Debian / Ubuntu
sudo apt install network-manager iw nmap aircrack-ng dsniff tcpdump mitmproxy smbclient sshpass
```

#### Verify dependencies

```bash
asbnetw version              # show detected tools
asbnetw crack --check        # check crack dependencies
asbnetw kick --check         # check kick dependencies
asbnetw mitm --check         # check mitm dependencies
```

---

## Usage

### Scan WiFi Networks

Discover all nearby WiFi networks with signal strength, channel, frequency, and security info.

```bash
asbnetw scan                        # scan all networks
asbnetw scan --sort signal          # sort by signal strength
asbnetw scan --sort name            # sort by SSID name
asbnetw scan --filter "home"        # filter SSIDs containing "home"
asbnetw scan --json                 # JSON output
```

### Show Saved WiFi Passwords

Retrieve stored WiFi passwords from NetworkManager for previously connected networks.

```bash
asbnetw password "WiFi Name"        # show password for specific network
asbnetw password --all              # show all saved passwords
asbnetw password --all --json       # JSON output
```

Aliases: `asbnetw pwd`, `asbnetw pass`

### Crack WiFi Password

Discover unknown WiFi passwords using dictionary attack or brute-force. Requires a WiFi adapter that supports monitor mode.

```bash
# Full attack (capture handshake + dictionary crack)
sudo asbnetw crack "Target WiFi"
sudo asbnetw crack "Target WiFi" --wordlist /path/to/wordlist.txt

# Brute-force attack
sudo asbnetw crack "Target WiFi" --method bruteforce --bf-min 8 --bf-max 10
sudo asbnetw crack "Target WiFi" --method bruteforce --bf-charset numeric

# Offline crack (already have a .cap file)
sudo asbnetw crack "Target WiFi" --capture file.cap --bssid AA:BB:CC:DD:EE:FF
```

Brute-force charset options: `numeric`, `lower`, `upper`, `alpha`, `alphanumeric`, `all`

### List Connected Devices

Scan the local network and list all connected devices.

```bash
sudo asbnetw devices                # list all devices
sudo asbnetw devices --sort vendor  # sort by vendor
sudo asbnetw devices --sort ip      # sort by IP address
sudo asbnetw devices --json         # JSON output
```

Aliases: `asbnetw dev`, `asbnetw clients`

Shows IP, MAC address, vendor/manufacturer, hostname, and latency. Your device is marked `YOU`, the router is marked `GATEWAY`.

### Port Scan & Access Probe

Scan open ports on a target device and find available access methods (SSH, SMB, FTP, ADB, HTTP, etc).

```bash
asbnetw access 192.168.0.102                  # scan + probe services
asbnetw access 192.168.0.102 --try-login      # also try default credentials
asbnetw access 192.168.0.102 --no-probe       # port scan only (fastest)
asbnetw access 192.168.0.102 --json           # JSON output
```

Aliases: `asbnetw probe`, `asbnetw port`

With `--try-login`, the tool automatically tests common default username/password combinations against each discovered service (SSH, SMB, FTP, Telnet, HTTP Basic Auth).

### Sniff Device Traffic

Capture and log all network traffic from a specific device. Extracts DNS queries, TLS SNI (HTTPS domains), and HTTP requests — no certificate installation needed.

```bash
# Sniff another device (--spoof is REQUIRED)
sudo asbnetw sniff 192.168.0.108 --spoof

# DNS only mode — see what domains they visit
sudo asbnetw sniff 192.168.0.108 --spoof --dns

# Quiet mode — only show summary after Ctrl+C
sudo asbnetw sniff 192.168.0.108 --spoof --quiet

# Custom output directory
sudo asbnetw sniff 192.168.0.108 --spoof -o ./logs
```

Aliases: `asbnetw traffic`, `asbnetw capture`

> **Important:** The `--spoof` flag is required when sniffing other devices on WiFi. Without it, their traffic doesn't pass through your machine (WiFi is a switched network). The `--spoof` flag uses ARP spoofing to redirect the target's traffic through your PC while keeping their internet connection alive.

Press `Ctrl+C` to stop. A summary will be displayed including:
- All domains accessed (DNS + TLS SNI)
- Top destination IPs with reverse DNS
- Protocol breakdown

### Kick Device from Network

Disconnect a device from the WiFi network using ARP spoofing. The target loses internet access until you stop.

```bash
sudo asbnetw kick 192.168.0.102              # kick until Ctrl+C
sudo asbnetw kick 192.168.0.102 -d 60        # kick for 60 seconds
sudo asbnetw kick 192.168.0.102 --method raw # without arpspoof (fallback)
sudo asbnetw kick 192.168.0.102 -y           # skip confirmation prompt
```

Aliases: `asbnetw deauth`, `asbnetw disconnect`

Press `Ctrl+C` to stop. ARP tables are automatically restored and the target device will reconnect within seconds.

### MITM Intercept HTTPS

Man-in-the-Middle attack to intercept and log all HTTP/HTTPS traffic from a target device.

```bash
sudo asbnetw mitm 192.168.0.108              # intercept all traffic
sudo asbnetw mitm 192.168.0.108 -p 8888      # custom proxy port
sudo asbnetw mitm 192.168.0.108 --quiet      # stats only
sudo asbnetw mitm 192.168.0.108 -o ./logs    # custom output directory
```

Aliases: `asbnetw spy`, `asbnetw intercept`

> **HTTPS note:** The target device will see certificate warnings because mitmproxy uses its own CA certificate. For clean interception without warnings:
>
> 1. Run `sudo asbnetw mitm` once to generate the CA cert
> 2. Copy `~/.mitmproxy/mitmproxy-ca-cert.pem` to the target device
> 3. Install the cert on the device: Settings > Security > Install certificate
>
> Without the cert installed, plain HTTP is still fully intercepted. HTTPS will fail on apps that use certificate pinning.

---

## Dependency Reference

| Feature | Required tools |
|---------|---------------|
| `scan` | nmcli or iw |
| `password` | nmcli |
| `crack` | aircrack-ng, wordlist |
| `devices` | nmap (sudo) |
| `access` | nmap, smbclient, sshpass (optional) |
| `sniff` | tcpdump, arpspoof (for `--spoof`) |
| `kick` | arpspoof or arping |
| `mitm` | mitmproxy, arpspoof, iptables |

---

## Tips

- Most network features require `sudo` for raw socket access and monitor mode.
- Run `asbnetw devices` first to find target device IPs before using `sniff`, `kick`, `access`, or `mitm`.
- All commands support `--json` for machine-readable output: `asbnetw scan --json > scan.json`
- Disable colored output with `--no-color` on any command.
- Log files are automatically saved to `/tmp/netw-traffic/` (sniff) and `/tmp/netw-mitm/` (mitm).
