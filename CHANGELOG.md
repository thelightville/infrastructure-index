# Changelog

All infrastructure changes are documented here.
Format: `[YYYY-MM-DD] TYPE: description — by @actor`

Types: `feat` (new resource), `fix` (bugfix), `destroy` (removed resource), `config` (configuration change), `docs`, `sec` (security), `maint` (maintenance)

---

## 2026-03

### Phase 1: Proxmox Import + Inventory Correction

**[2026-03-03] feat: Phase 1 - Complete Proxmox Import** — by @thelightville

- Created Proxmox API token: `opentofu@pam!tofu-token`
- Imported all 14 Proxmox resources into tofu local state
- tofu plan shows **0 replacements, 0 destroys** after inventory correction
- Plan: `0 to add, 14 to change (in-place), 0 to destroy`

**[2026-03-03] fix: Corrected node assignments** — by @thelightville

Live `pvesh` audit revealed 4 wrong node assignments in original docs:
- CT103 OnlineRadio App: was documented as `pve`, actually on **pve2**
- CT107 API Gateway: was documented as `pve`, actually on **pve2**
- CT109 AzuraCast: was documented as `pve`, actually on **pve2**
- CT115 PBS: was documented as `pve2`, actually on **pve**

**[2026-03-03] fix: Corrected infrastructure specs** — by @thelightville

Live pvesh audit corrected several wrong values in documentation:
- CT109 disk: 100GB → 670GB (81GB media + OS)
- CT117 disk: 200GB → 800GB; datastore: nvme-storage → ssd-storage; RAM: 32GB → 16GB
- CT201 disk: 100GB → 200GB; datastore: local → pve2-nvme; RAM: 8GB → 6GB
- CT115 disk: 50GB → 32GB; datastore: local → pbs-ssd; has 2 NFS mounts
- VM100: corrected to 16 vCPU, 32GB RAM, 250GB disk; GPU PCI ID confirmed: 0000:03:00
- VM200: corrected to 8 vCPU, 32GB RAM, 150GB (ssd-storage), OVMF BIOS, 2 NICs
- Gateway across all CTs: was 172.16.16.1, actually 172.16.16.16

**[2026-03-03] feat: Discovered CT121 billing system** — by @thelightville

Previously undocumented container found via pvesh audit:
- CT121 | IP: 172.16.16.121 | Node: pve | Hostname: bill.i.ng
- 4 cores, 4GB RAM, 50GB nvme-storage | Ubuntu | unprivileged | nesting=1
- Service: Billing system (WHMCS/FOSSBilling/custom — TBD)
- Added to tofu IaC with `prevent_destroy = true`
- Added to all documentation maps

---

## 2025-07

### Phase 0: IaC Foundation

**[2025-07] feat: Phase 0 Foundation** — by @thelightville

- Installed OpenTofu v1.11.5, SOPS 3.9.4, age on pve
- Generated age keypair for secrets encryption
- Created GitHub repos: proxmox-iac, infrastructure-index
- Created OpenTofu modules for all 7 providers:
  - `tofu/proxmox/` — All 13 CTs + 2 VMs defined (not yet applied — import first)
  - `tofu/cloudflare/` — DNS zones, Tunnel, R2 bucket, CNAME records
  - `tofu/oracle/` — E2.1.Micro, IPsec VPN, VCN, subnet
  - `tofu/gcp/` — Cloud Run, Firestore, GCS Archive
  - `tofu/aws/` — S3 state/backup bucket, SNS notifications
  - `tofu/azure/` — Front Door CDN (standby)
  - `tofu/backblaze/` — B2 DR backup bucket with 90-day lifecycle
- Created Ansible playbooks:
  - proxmox-hardening.yml — node security baseline
  - nginx-routing.yml — Nginx config backup + health check
  - backup-scripts.yml — rclone + PBS sync cron jobs
  - ct117-docker-host.yml — Docker + Traefik + Portainer
  - vm100-ai-server.yml — Ollama + Open WebUI
  - mailcow-config-export.yml — Mailcow config to git
  - monitoring.yml — Prometheus config + alerting rules
  - ssl-certs.yml — Certbot + Cloudflare DNS challenge
- Created GitHub Actions CI/CD:
  - tofu-plan.yml — PR auto-plan with matrix strategy
  - tofu-apply.yml — merge-to-main apply with environment approval
- Created pre-commit hook for secrets scanning
- Created infrastructure-index documentation repo

---

## Pending (Phase 1)

- [ ] Import all existing Proxmox resources into tofu state
- [ ] Import Cloudflare DNS records into tofu state
- [ ] Import Oracle infrastructure into tofu state
- [ ] Run proxmox-hardening.yml on all 3 nodes
- [ ] Populate `secrets/providers.sops.yaml` with real credentials
- [ ] Add all GitHub Actions secrets
- [ ] Verify CT113 WhatsApp chatbot 502 resolved
- [ ] Confirm SATA RAID status on pve
- [ ] Run `lspci | grep -i nvidia` on pve to get GPU PCI IDs for VM100
- [ ] Save age private key to Bitwarden: `thelightville-iac-age-key`
