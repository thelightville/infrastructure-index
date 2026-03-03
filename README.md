# Infrastructure Index — thelightville

> Single source of truth for the entire thelightville infrastructure.
> Last updated: 2025-07
> Managed by: [@thelightville](https://github.com/thelightville)

---

## Quick Reference

| Document | Contents |
|---|---|
| [INFRASTRUCTURE-MAP.md](INFRASTRUCTURE-MAP.md) | All servers, VMs, containers with specs |
| [NETWORK-MAP.md](NETWORK-MAP.md) | IPs, VLANs, routing, VPN tunnels |
| [DOMAIN-MAP.md](DOMAIN-MAP.md) | All 148 subdomains → backend routing |
| [CLOUD-SERVICES.md](CLOUD-SERVICES.md) | All 7 cloud providers + free tier expiry |
| [CREDENTIALS-MAP.md](CREDENTIALS-MAP.md) | Where each credential lives |
| [RUNBOOKS.md](RUNBOOKS.md) | DR procedures per failure scenario |
| [DECISIONS.md](DECISIONS.md) | Architecture decision log (ADRs) |
| [CHANGELOG.md](CHANGELOG.md) | Infrastructure change history |

---

## IaC Repositories

| Repo | Purpose |
|---|---|
| [proxmox-iac](https://github.com/thelightville/proxmox-iac) | OpenTofu modules + Ansible playbooks |
| [infrastructure-index](https://github.com/thelightville/infrastructure-index) | This repo — documentation |

---

## Environment at a Glance

- **3-node Proxmox cluster**: pve (primary), pve2 (services), pve3 (hosting)
- **13 LXC containers + 2 VMs** under management
- **7 cloud providers**: Oracle, GCP, AWS, Azure, Cloudflare, Backblaze B2
- **38+ domains**, 148 subdomains via Cloudflare Tunnel → Nginx proxy
- **34 email domains** via Mailcow (CT201)
- **DR strategy**: Local PBS → Backblaze B2 → AWS S3 → AWS Glacier

---

## Emergency Contacts / Key Services

| Service | URL | Notes |
|---|---|---|
| Proxmox UI | https://172.16.16.20:8006 | pve (primary) |
| PBS UI | https://172.16.16.115:8007 | Backup Server |
| Grafana | http://172.16.16.111:3000 | Monitoring |
| Mailcow | https://mail.thelightville.com | Email admin |
| Open WebUI | http://172.16.16.19:3000 | AI (VM100) |
| Portainer | http://172.16.16.117:9000 | Docker (CT117) |
| Cloudflare | https://dash.cloudflare.com | DNS + Tunnel |
| Oracle Cloud | https://cloud.oracle.com | Free VM |
| GCP Console | https://console.cloud.google.com | WhatsApp chatbot |
