# Cloud Services Map

## Summary

| Provider | Resources | Monthly Cost | Free Tier Expiry | IaC Module |
|---|---|---|---|---|
| Oracle Cloud | E2.1.Micro VM, VCN, IPsec VPN | $0 | Never (Always Free) | `tofu/oracle/` |
| GCP | Cloud Run, Firestore, GCS Archive | ~$0.30 | Never (per-product limits) | `tofu/gcp/` |
| AWS (new account) | S3 (state + backups), SNS | $0 | **~January 2027** ⚠️ | `tofu/aws/` |
| AWS (old account) | Glacier 88GB — **EXCLUDED** | $0.32 at risk | Account suspended | EXCLUDED |
| Azure | Front Door CDN (standby) | $0 ($200 credits) | **~July 2026** ⚠️ | `tofu/azure/` |
| Cloudflare | DNS, R2 (252GB), Tunnel, Workers | $2.73 | Never (paid plan) | `tofu/cloudflare/` |
| Backblaze B2 | 226GB DR backups | ~$1.30 | Never (pay-as-you-go) | `tofu/backblaze/` |

**Total monthly**: ~$4.33

---

## Oracle Cloud (Always Free)

- **Account**: thelightville Oracle Cloud
- **Region**: us-ashburn-1 (IAD)
- **Resources**:
  - 1x AMD E2.1.Micro (1 OCPU, 1GB RAM) — IP: 40.233.116.184
  - 1x ARM A1.Flex (up to 4 OCPU, 24GB RAM) — **Pending provisioning**, commented out in main.tf
  - VCN: 10.0.0.0/16, public subnet 10.0.1.0/24
  - 2x IPsec VPN tunnels to home Sophos XG
  - Object Storage: 20GB free tier
- **Free forever**: AMD Micro + ARM A1 are Always Free tier
- **Action needed**: Provision ARM A1 instance (update `tofu/oracle/main.tf` to uncomment)

---

## GCP

- **Project**: thelightville-chatbot (confirm project ID)
- **Region**: us-central1 (Cloud Run) / northamerica-northeast1 (GCS Archive)
- **Resources**:
  - Cloud Run: `whatsapp-webhook` — scales 0-10, deployed by app CI/CD
  - Firestore: native mode, default database
  - GCS: `thelightville-archive` — Archive class, 90-day lifecycle
- **Always Free limits**: Cloud Run (2M requests/mo), Firestore (1GB storage, 50K reads/20K writes/day)
- **Billing threshold**: Set alert at $5/month

---

## AWS (New Account — ~Jan 2027 Free Tier)

- **Account ID**: Confirm in AWS console
- **Region**: us-east-1
- **Resources**:
  - S3: `thelightville-critical-backups` — state backend + DR archive
  - SNS: Admin notification topic → admin@thelightville.com
- **Free tier**: S3 5GB, SNS 1M pub/delivery free until ~Jan 2027
- **Action before expiry**: Review costs, consider moving state backend to Cloudflare R2 (no egress)
- **⚠️ OLD ACCOUNT**: Separate account with $321 Glacier bill history — **DO NOT USE** — accepted suspension

---

## Azure ($200 Credits — ~July 2026)

- **Subscription**: Pay-as-you-go with $200 new account credits
- **Region**: southafricanorth (planned) / global (Front Door)
- **Resources**:
  - Azure Front Door Standard — standby CDN for Cloudflare failover
  - (Planned) Blob Storage southafricanorth — media files post-CT117 migration
- **⚠️ Credits expiry**: ~July 2026 — monitor in Cost Management
- **Cost at expiry**: Front Door Standard ~$35/month if kept — review need

---

## Cloudflare

- **Plan**: Pro (or Business — confirm in dashboard)
- **Account ID**: Stored in `secrets/providers.sops.yaml`
- **Resources**:
  - DNS: 38+ zones, 148 CNAME records (orange cloud proxied)
  - R2: `onlineradio-music` bucket — 252GB music files
  - Tunnel: 1 active tunnel → pve cloudflared daemon
  - Workers: Review active workers in dashboard
- **Pricing**: R2 free up to 10GB/month operations; storage $0.015/GB/month above 10GB

---

## Backblaze B2

- **Account**: thelightville
- **Bucket**: `proxmox-dr-backups` — 226GB
- **Pricing**: $0.006/GB/month storage + $0.01/GB download
- **DR role**: Tier 2 (after local PBS/LaCie, before AWS)
- **Sync**: Daily via rclone from CT115 PBS (see backup-scripts playbook)

---

## Free Tier Monitoring Calendar

| Date | Event | Action |
|---|---|---|
| Monthly | Review AWS Cost Explorer | Ensure S3 stays in free tier |
| Monthly | Review GCP billing | Verify Cloud Run stays $0 |
| 2026-07 | Azure $200 credits expiry | Decide: keep Front Door ($35/mo) or delete |
| 2027-01 | AWS free tier expiry | Move state to CF R2 or accept S3 costs |
