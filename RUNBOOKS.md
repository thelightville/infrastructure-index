# Runbooks

## DR Scenario: Proxmox Node Failure

### pve (Primary Node) Failure
**Impact**: CT103, CT105, CT107, CT113, CT117, VM100, VM200 down + Nginx routing down = ALL SITES DOWN

1. **Assess**: Can pve be restarted? Check power, hardware
2. **If hardware failure**: Boot from PBS (CT115 on pve2) → Restore CTs to pve2
   ```bash
   # On pve2 — restore critical CTs from PBS
   pct restore 103 /var/lib/proxmox-backup/datastore/... --storage local-lvm
   ```
3. **DNS failover**: If downtime > 15min, update Cloudflare Tunnel to point to pve2 nginx
4. **VM100 (GPU)**: Cannot migrate — AI server offline until pve restored. Inform users.
5. **VM200 (Windows)**: Cannot migrate — Windows dev offline during pve failure

### pve2 (Services Node) Failure
**Impact**: CT111 (monitoring), CT115 (PBS), CT201 (Mailcow) down = Email + monitoring down

1. PBS offline — immediately limits restore capability. Fix pve2 first.
2. Mailcow: Email queued by senders, typically 4-7 day retry. Restore CT201 to pve as temporary measure.
3. Monitoring offline: Use cloud provider dashboards during outage

### pve3 (Hosting Node) Failure
**Impact**: CT101 (cPanel), CT119 (ignore — corrupt) down

1. CT101 cPanel hosts 30+ client sites. Priority restore from PBS.
2. CT119 is corrupt — do not attempt restore, delete via tofu

---

## DR Scenario: Full Hardware Loss

**3-tier DR recovery order**:
1. Restore from Backblaze B2 (Tier 2) — rclone pull to new PBS
2. If B2 unavailable: Restore from AWS S3 archive (Tier 3) — monthly snapshots only
3. If all offsite lost: Last-resort rebuild from tofu/ansible (recreates infrastructure, not data)

**Recovery steps**:
```bash
# 1. Provision new bare metal with Proxmox
# 2. Clone proxmox-iac repo
git clone git@github.com:thelightville/proxmox-iac.git
# 3. Install tools (age, sops, tofu)
# 4. Restore age key from Bitwarden
echo "$AGE_KEY" > ~/.config/sops/age/keys.txt && chmod 600 $_
# 5. Decrypt secrets
sops -d secrets/providers.sops.yaml > /tmp/providers.yaml
# 6. tofu import existing cloud resources, then apply
cd tofu/proxmox && tofu init && tofu import ...
# 7. Restore CT/VM data from B2 via rclone to new PBS datastore
rclone sync backblaze-b2:proxmox-dr-backups/ /mnt/new-datastore/
# 8. Run PBS → Proxmox restore for each CT/VM
```

---

## DR Scenario: AWS Glacier Old Account Issues

- Old account was suspended due to $321 Glacier bill
- **DO NOT USE** old account for anything
- If suspended account sends billing emails: ignore — accepted as resolved
- New account (`thelightville-critical-backups` S3 bucket) is the active DR destination

---

## Operational: SATA RAID Degraded (95% Concern)

Check on pve:
```bash
cat /proc/mdstat           # Software RAID status
mdadm --detail /dev/md0    # Detailed RAID status (if applicable)
smartctl -a /dev/sda        # SMART data for each disk
```

If degraded: Replace failed drive before proceeding with IaC apply on storage-heavy changes.

---

## Operational: CT113 (WhatsApp Chatbot) 502 Error

```bash
# On pve
pct enter 113
systemctl status <chatbot-service>  # Check service name
journalctl -u <chatbot-service> -n 50  # Recent logs
```

Common cause: Node.js process exited, port binding issue, or GCP token expired.

---

## Operational: Sophos XG Config Backup

```bash
# Manual process — no IaC provider
# 1. Log into Sophos: https://172.16.16.16:4444
# 2. Backup → Local Backup → Download
# 3. Upload to private repo sophos-config-backups:
git clone git@github.com:thelightville/sophos-config-backups.git
cp ~/Downloads/SophosXG-backup-*.tar.gz sophos-config-backups/
cd sophos-config-backups && git add . && git commit -m "chore: scheduled config backup $(date +%Y-%m-%d)"
git push
```

---

## Operational: Cloudflare Tunnel Reconnect

If tunnel shows disconnected in Cloudflare dashboard:
```bash
# On pve
systemctl status cloudflared
journalctl -u cloudflared -n 30
systemctl restart cloudflared
```

Verify: Check https://one.dash.cloudflare.com → Access → Tunnels

---

## Operational: AWS Free Tier Approaching Limit

```bash
# Check current S3 usage
aws s3 ls s3://thelightville-critical-backups --recursive --human-readable --summarize
# If approaching 5GB free tier:
# 1. Move old tofu state versions to Glacier class
# 2. Or migrate state to Cloudflare R2 (free, no egress)
```
