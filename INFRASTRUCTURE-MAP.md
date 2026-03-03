# Infrastructure Map

## Physical Nodes

| Node | IP | Cores | RAM | Role | OS |
|---|---|---|---|---|---|
| pve | 172.16.16.20 | 32 | 125GB | Primary | Proxmox VE |
| pve2 | 172.16.16.21 | 20 | 31GB | Services | Proxmox VE |
| pve3 | 172.16.16.22 | 4 | 62GB | Hosting | Proxmox VE |

---

## LXC Containers

| CT | IP | Service | Node | OS | Disk | Status | Notes |
|---|---|---|---|---|---|---|---|
| CT101 | .101 | cPanel | pve3 | AlmaLinux 8 | 100GB | ACTIVE | 30+ hosted sites; prevent_destroy until CT117 migration |
| CT103 | .103 | OnlineRadio App | pve | Ubuntu 22.04 | 20GB | ACTIVE | Apache + PHP |
| CT105 | .105 | ClaApp | pve | Ubuntu 22.04 | 20GB | ACTIVE | Nginx, 14 microservices |
| CT107 | .107 | API Gateway | pve | Ubuntu 22.04 | 10GB | ACTIVE | Nginx, 5 endpoints |
| CT109 | .109 | AzuraCast | pve | Ubuntu 22.04 | 100GB | ACTIVE | Docker-based, 81GB media |
| CT111 | .111 | Grafana + Prometheus | pve2 | Ubuntu 22.04 | 20GB | ACTIVE | Monitoring stack |
| CT113 | .113 | WhatsApp Chatbot | pve | Ubuntu 22.04 | 10GB | ACTIVE | 502 error logged — verify |
| CT115 | .115 | PBS | pve2 | Debian 12 | 50GB + LaCie | ACTIVE | Proxmox Backup Server |
| CT117 | .117 | Docker Host | pve | Ubuntu 22.04 | 200GB | ACTIVE | Migration in progress — 135 containers |
| CT119 | .119 | CyberPanel | pve3 | AlmaLinux 8 | 50GB | CORRUPT | Ready to destroy via tofu PR |
| CT201 | .201 | Mailcow | pve2 | Debian 12 | 100GB | ACTIVE | 34 email domains; prevent_destroy |

## Virtual Machines

| VM | IP | Service | Node | OS | vCPU | RAM | Disk | Status | Notes |
|---|---|---|---|---|---|---|---|---|---|
| VM100 | .19 | AI Server | pve | Ubuntu 22.04 | 8 | 16GB | 100GB | ACTIVE | NVIDIA Quadro K2200 GPU passthrough; NOT migratable |
| VM200 | .200 | Windows Dev | pve | Windows 11 | 4 | 8GB | 100GB | ACTIVE | Local backup only; prevent_destroy |

---

## Storage

| Device | Capacity | Location | Purpose |
|---|---|---|---|
| Local ZFS (pve) | ~2TB | pve | VM/CT local storage |
| LaCie 5.5TB | 5.5TB | pve2 (USB) | PBS primary datastore |
| Cloudflare R2 | 252GB | Cloud | OnlineRadio music files |
| Backblaze B2 | 226GB | Cloud | DR backup replica |
| AWS S3 | New account | Cloud | State + critical backup |

---

## Hardware Specifics

### VM100 GPU Passthrough
- GPU: NVIDIA Quadro K2200 (4GB VRAM)
- PCI ID: **Run `lspci | grep -i nvidia` on pve to confirm**
- Update: `tofu/proxmox/main.tf` VM100 `hostpci` block with confirmed PCI IDs
- Cannot live-migrate (PCI passthrough locks to pve node)
