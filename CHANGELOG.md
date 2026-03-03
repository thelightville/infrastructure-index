# Changelog

All infrastructure changes are documented here.
Format: `[YYYY-MM-DD] TYPE: description — by @actor`

Types: `feat` (new resource), `fix` (bugfix), `destroy` (removed resource), `config` (configuration change), `docs`, `sec` (security), `maint` (maintenance)

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
