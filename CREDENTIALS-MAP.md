# Credentials Map

> This file documents WHERE credentials are stored — NEVER the credentials themselves.
> All secrets are stored in Bitwarden and/or `secrets/providers.sops.yaml` (SOPS encrypted).

---

## Age Encryption Key (Master)

| Key | Location |
|---|---|
| Public key | `.sops.yaml` in proxmox-iac repo (safe to expose) |
| Private key | Bitwarden: `thelightville-iac-age-key` |
| Runtime location | `~/.config/sops/age/keys.txt` on pve (chmod 600) |
| GitHub Actions | Secret: `SOPS_AGE_KEY` in proxmox-iac repo settings |

---

## GitHub Actions Secrets Required

Navigate to: https://github.com/thelightville/proxmox-iac/settings/secrets/actions

| Secret Name | Provider | Source |
|---|---|---|
| `SOPS_AGE_KEY` | SOPS | Bitwarden: `thelightville-iac-age-key` |
| `AWS_ACCESS_KEY_ID` | AWS | AWS IAM → proxmox-iac-tofu user |
| `AWS_SECRET_ACCESS_KEY` | AWS | AWS IAM → proxmox-iac-tofu user |
| `PROXMOX_API_URL` | Proxmox | https://172.16.16.20:8006/api2/json |
| `PROXMOX_TOKEN_ID` | Proxmox | Proxmox UI → Datacenter → API Tokens |
| `PROXMOX_TOKEN_SECRET` | Proxmox | Proxmox UI → Datacenter → API Tokens |
| `CLOUDFLARE_API_TOKEN` | Cloudflare | Cloudflare → Profile → API Tokens |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare | Cloudflare dashboard URL |
| `CLOUDFLARE_TUNNEL_SECRET` | Cloudflare | Tunnel creation secret |
| `ORACLE_TENANCY_OCID` | Oracle | OCI Console → Tenancy |
| `ORACLE_USER_OCID` | Oracle | OCI Console → User settings |
| `ORACLE_FINGERPRINT` | Oracle | OCI Console → API Keys |
| `ORACLE_PRIVATE_KEY_PATH` | Oracle | Path to OCI private key on pve |
| `GCP_PROJECT_ID` | GCP | GCP Console → Project settings |
| `GCP_CREDENTIALS_FILE` | GCP | Service account JSON (base64 encoded) |
| `AZURE_SUBSCRIPTION_ID` | Azure | Azure Portal → Subscriptions |
| `AZURE_TENANT_ID` | Azure | Azure Portal → AAD → Properties |
| `AZURE_CLIENT_ID` | Azure | Azure Portal → App Registration |
| `AZURE_CLIENT_SECRET` | Azure | Azure Portal → App Registration → Secrets |
| `B2_APPLICATION_KEY_ID` | Backblaze | Backblaze → App Keys |
| `B2_APPLICATION_KEY` | Backblaze | Backblaze → App Keys |

---

## SOPS Encrypted Secrets File

**Path**: `proxmox-iac/secrets/providers.sops.yaml`

**To decrypt** (on pve with age key available):
```bash
sops -d secrets/providers.sops.yaml
```

**To edit**:
```bash
sops secrets/providers.sops.yaml
```

**Structure**: See `secrets/README.md` in proxmox-iac for the full template.

---

## Proxmox API Token Setup

```bash
# Run on pve — creates API token for OpenTofu
pveum user add opentofu@pve
pveum aclmod / -user opentofu@pve -role Administrator
pveum user token add opentofu@pve tofu-token --privsep=0
# Output shows token ID and secret — save both to Bitwarden
```

---

## Sophos XG Credentials

- Admin URL: https://172.16.16.16:4444  
- Credentials: Bitwarden entry `sophos-xg-admin`
- API backup: Private repo `sophos-config-backups` (separate from proxmox-iac)

---

## Mailcow API

- URL: https://mail.thelightville.com  
- API key location: Mailcow Admin → API → Long-lived API key  
- Store as: `mailcow_api_key` in `secrets/providers.sops.yaml`  
- Used by: `ansible/playbooks/mailcow-config-export.yml`
