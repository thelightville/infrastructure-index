# Network Map

## LAN Addressing

- **Subnet**: 172.16.16.0/24
- **Gateway**: 172.16.16.1 (Sophos XG Firewall LAN) / **172.16.16.16** (Sophos mgmt — this is the actual gateway used by all CTs/VMs)
- **DNS**: 172.16.16.1 (Sophos internal DNS)

| Host | IP | Role |
|---|---|---|
| Sophos XG | 172.16.16.1 (LAN), 172.16.16.16 (mgmt/gateway) | Firewall / Router — CTs use .16 as gw |
| pve | 172.16.16.20 | Proxmox primary |
| pve2 | 172.16.16.21 | Proxmox services |
| pve3 | 172.16.16.22 | Proxmox hosting |
| VM100 | 172.16.16.100 | AI server (corrected from .19) |
| VM200 | 172.16.16.200 | Windows dev |
| CT101 | 172.16.16.101 | cPanel/WHM hosting (pve3) |
| CT103 | 172.16.16.103 | OnlineRadio App (pve2) |
| CT105 | 172.16.16.105 | ClaApp (pve) |
| CT107 | 172.16.16.107 | OnlineRadio API Gateway (pve2) |
| CT109 | 172.16.16.109 | AzuraCast streaming (pve2) |
| CT111 | 172.16.16.111 | Grafana + Prometheus (pve2) |
| CT113 | 172.16.16.113 | WhatsApp Chatbot (pve) |
| CT115 | 172.16.16.115 | Proxmox Backup Server (pve) |
| CT117 | 172.16.16.117 | Docker Host (pve) |
| CT119 | 172.16.16.119 | CyberPanel — CORRUPT (pve3) |
| **CT121** | **172.16.16.121** | **Billing System — bill.i.ng (pve) — discovered 2026-03-03** |
| CT201 | 172.16.16.201 | Mailcow email server (pve2) |

---

## External / WAN

| IP | Provider | Purpose |
|---|---|---|
| Dynamic (ISP) | Home WAN | Sophos XG outbound NAT |
| 40.233.116.184 | Oracle Cloud | E2.1.Micro VM — VPN endpoint |
| `<tunnel-id>.cfargotunnel.com` | Cloudflare | Tunnel CNAME target |

---

## VPN Tunnels

### Oracle IPsec VPN (Inverted Architecture)
- HomeWAN → OCI IPsec → pve via Oracle BGP
- Purpose: Static public IP for non-Cloudflare services, Oracle ARM VM access
- **Tunnel 1**: Oracle DRG ↔ Home (via 140.238.150.158)
- **Tunnel 2**: Oracle DRG ↔ Home (via 140.238.151.76) — HA failover
- CPE (Customer Premises Equipment): Sophos XG at 172.16.16.16
- Oracle VCN: 10.0.0.0/16, public subnet 10.0.1.0/24
- Managed by: `tofu/oracle/main.tf`

---

## Traffic Routing (Public → Internal)

```
Internet
    │
    ├── *.thelightville.com (148 subdomains)
    │       └── Cloudflare Proxy (orange cloud)
    │               └── Cloudflare Tunnel (zero-trust, outbound only)
    │                       └── pve:172.16.16.20 (cloudflared daemon)
    │                               └── Nginx /etc/nginx/sites-available/cloudflare-tunnel-proxy.conf
    │                                       └── Route to backend CT/VM by subdomain
    │
    ├── mail.thelightville.com (SMTP/IMAP/POP3)
    │       └── Cloudflare DNS (grey cloud — NOT proxied)
    │               └── Home WAN → Sophos XG → CT201 Mailcow
    │
    └── Direct VPN services
            └── Oracle IPsec → pve LAN
```

---

## Nginx Routing (pve — Cloudflare Tunnel Proxy)

- Config: `/etc/nginx/sites-available/cloudflare-tunnel-proxy.conf`
- Cloudflared config: `/etc/cloudflared/config.yml`
- Pattern: Each subdomain has a `server {}` block proxying to the appropriate CT/VM IP

---

## Firewall (Sophos XG)

- Management: https://172.16.16.16:4444
- **No IaC provider available** — config versioned in private repo `sophos-config-backups`
- Export procedure: See [RUNBOOKS.md — Sophos Config Backup](RUNBOOKS.md)
- Key rules: Cloudflare Tunnel outbound (CT/pve → Cloudflare), IPsec IKE (UDP 500/4500), Mailcow inbound (25/587/993)
