# Infrastructure Map

## Physical Nodes

| Node | IP | Cores | RAM | Role | OS |
|---|---|---|---|---|---|
| pve | 172.16.16.20 | 32 | 125GB | Primary | Proxmox VE |
| pve2 | 172.16.16.21 | 20 | 31GB | Services | Proxmox VE |
| pve3 | 172.16.16.22 | 4 | 62GB | Hosting | Proxmox VE |

---

## LXC Containers

> **Last audited**: 2026-03-03 via `pvesh get /nodes/*/lxc` — corrected node assignments, sizes, and discovered CT121.

| CT | IP | Service | Node | OS | Disk | Cores | RAM | Status | Notes |
|---|---|---|---|---|---|---|---|---|---|
| CT101 | .101 | cPanel/WHM | pve3 | AlmaLinux 8 (centos) | 300GB (pve3-nvme-storage) | 4 | 16GB | ACTIVE | 30+ hosted sites; prevent_destroy until CT117 migration |
| CT103 | .103 | OnlineRadio App | **pve2** | Ubuntu (debian) | 25GB (pve2-nvme) | 2 | 4GB | ACTIVE | Apache + PHP; app/admin/deepsound.onlineradio.com.ng |
| CT105 | .105 | ClaApp | pve | Ubuntu | 8GB (nvme-storage) | 8 | 4GB | ACTIVE | Nginx, 14 microservices; claapp.myapps.com.ng |
| CT107 | .107 | API Gateway | **pve2** | Ubuntu | 40GB (pve2-nvme) | 4 | 4GB | ACTIVE | Nginx; api/auth/music/stations endpoints |
| CT109 | .109 | AzuraCast | **pve2** | Ubuntu | 670GB (pve2-nvme) | 3 | 6GB | ACTIVE | Docker-based; 81GB media; channel.onlineradio.com.ng |
| CT111 | .111 | Grafana + Prometheus | pve2 | Ubuntu | 20GB (pve2-nvme) | 2 | 4GB | ACTIVE | Monitoring stack; monitoring.thelightville.xyz |
| CT113 | .113 | WhatsApp Chatbot | pve | Ubuntu | 20GB (nvme-storage) | 4 | 4GB | ACTIVE | chatbot.myapps.com.ng |
| CT115 | .115 | PBS | **pve** | Debian 12 | 32GB (pbs-ssd) + NFS mounts | 4 | 4GB | ACTIVE | Proxmox Backup Server; 2 NFS datastores mounted; prevent_destroy |
| CT117 | .117 | Docker Host | pve | Ubuntu | 800GB (ssd-storage) | 12 | 16GB | ACTIVE | Migration in progress — 135 containers; privileged |
| CT119 | .119 | CyberPanel | pve3 | Ubuntu | 145GB (pve3-nvme-storage) | 4 | 4GB | CORRUPT | Ready to destroy after CT117 migration completes |
| **CT121** | **.121** | **Billing System** | **pve** | **Ubuntu** | **50GB (nvme-storage)** | **4** | **4GB** | **ACTIVE** | **bill.i.ng — discovered 2026-03-03; prevent_destroy; service TBD (WHMCS/FOSSBilling?)** |
| CT201 | .201 | Mailcow | pve2 | Debian 12 | 200GB (pve2-nvme) | 4 | 6GB | ACTIVE | 34 email domains; webmail.thelightville.net; prevent_destroy |

## Virtual Machines

| VM | IP | Service | Node | OS | vCPU | RAM | Disk | Status | Notes |
|---|---|---|---|---|---|---|---|---|---|
| VM100 | .100 | AI Server | pve | Ubuntu 22.04 | 16 | 32GB | 250GB (nvme-storage) | ACTIVE | NVIDIA Quadro K2200 GPU passthrough (hostpci0=0000:03:00); NOT migratable; q35 machine |
| VM200 | .200 | Windows Dev | pve | Windows 11 | 8 | 32GB | 150GB (ssd-storage) | ACTIVE | OVMF BIOS; virtio-scsi-single; EFI disk; 2 NICs (vmbr0+vmbr1); local backup only; prevent_destroy |

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
- PCI IDs (confirmed 2026-03-03 via `lspci` on pve):
  - `0000:03:00` — combined VGA + HDMI audio passed as single `hostpci0` device
  - `03:00.0` — NVIDIA Quadro K2200 VGA controller
  - `03:00.1` — NVIDIA HD Audio controller
- tofu config: `hostpci0 = "0000:03:00", pcie=true, rombar=false, xvga=true`
- Cannot live-migrate (PCI passthrough locks to pve node)
