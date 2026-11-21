<img width="880" height="884" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/6e75ace7-725e-411d-be4b-bec087e7d103" />
# Nightwatch — AdGuard Home

**Self-hosted DNS filtering and network-wide ad blocking using Proxmox and AdGuard Home.**

## Overview

This project documents the deployment of AdGuard Home as a self-hosted DNS filtering solution within my homelab, running on Proxmox VE.

AdGuard Home provides DNS-level filtering to block advertisements, trackers, and unwanted domains across devices connected to my home network.

Rather than installing an ad blocker on every device, this implementation centralizes DNS filtering through a dedicated Linux container.

## Architecture

- **Hypervisor:** Proxmox VE
- **Host:** Nightwatch
- **Container:** Debian 12 LXC
- **Service:** AdGuard Home
- **DNS upstream:** Cloudflare DNS-over-HTTPS
- **Network:** Private LAN

## Network Configuration

| Component | Configuration |
|---|---|
| Proxmox Host | 192.168.68.199 |
| Proxmox Hostname | Nightwatch |
| AdGuard Container | CT 101 |
| Container Hostname | adguard |
| AdGuard IP | 192.168.68.201 |
| Gateway | 192.168.68.1 |
| Network Bridge | vmbr0 |
| Subnet | 192.168.68.0/24 |
| DNS Port | 53 |
| Web Interface | Port 80 |
| Container OS | Debian 12 |

## Network Flow

Devices on my home network use AdGuard Home as their DNS resolver.

1. Client device sends DNS query to AdGuard Home.
2. AdGuard Home evaluates the request against its filtering rules.
3. Blocked domains are denied.
4. Allowed DNS requests are forwarded to Cloudflare DNS-over-HTTPS.
5. The resolved response is returned to the client.

## DNS Configuration

### Upstream DNS

Cloudflare DNS-over-HTTPS:

`https://cloudflare-dns.com/dns-query`

### Bootstrap DNS

- 1.0.0.1
- 1.1.1.1
- 2606:4700:4700::1001
- 2606:4700:4700::1111

### Filtering

- AdGuard DNS Filter
- DNS-level advertisement filtering
- Tracking and unwanted domain blocking

## Deployment

### 1. Create a Debian LXC

Create a Linux container on Proxmox with the following configuration:

- Debian 12
- Unprivileged container
- 1 CPU core
- 512 MB RAM
- 4 GB storage
- Static IP: 192.168.68.201
- Gateway: 192.168.68.1
- Bridge: vmbr0

### 2. Install AdGuard Home

Install curl inside the container:

```bash
apt update
apt install curl -y
```

Run the official AdGuard Home installation script:

```bash
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh
```

### 3. Initial Configuration

Configure AdGuard Home through the initial setup wizard.

- Web interface: Port 80
- DNS listener: Port 53
- Listen on all interfaces
- Create administrator account

### 4. Configure Upstream DNS

Configure Cloudflare DNS-over-HTTPS:

```text
https://cloudflare-dns.com/dns-query
```

### 5. Configure Client DNS

Configure client devices to use:

```text
192.168.68.201
```

## Validation and Testing

### Test DNS resolution

```bash
nslookup google.com 192.168.68.201
```

Expected result: Google domains resolve to valid IP addresses.

### Test DNS blocking

```bash
nslookup doubleclick.net 192.168.68.201
```

Expected result: A blocked response, such as `0.0.0.0`.

### Dashboard

Monitor DNS requests, blocked queries, clients and filtering activity through the AdGuard Home dashboard.

## Security Considerations

- AdGuard Home is intended for private LAN use.
- Proxmox management is not exposed to the public internet.
- No public port forwarding is configured.
- DNS filtering is intended for trusted local devices.
- Upstream DNS traffic uses encrypted DNS-over-HTTPS.

## Limitations

DNS filtering does not block every advertisement.

Some advertising services, including YouTube advertisements, use delivery infrastructure that cannot reliably be blocked using DNS filtering alone.

Browser-level filtering may still be required.

## Future Improvements

- Configure router-wide DNS
- Configure client identification
- Add additional carefully selected blocklists
- Configure per-device filtering policies
- Monitor DNS activity
- Deploy additional homelab services
- Improve infrastructure documentation

---

*This project is part of my personal homelab and infrastructure learning journey.*
