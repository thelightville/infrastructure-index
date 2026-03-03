# Architecture Decision Log (ADRs)

## ADR-001: OpenTofu over Terraform/Pulumi

**Date**: 2025-07  
**Status**: Accepted

**Context**: Needed IaC tool for Proxmox + multi-cloud management. Evaluated: Terraform, OpenTofu, Pulumi.

**Decision**: OpenTofu v1.11.5

**Rationale**:
- HCL simpler than Pulumi (Pulumi needs programming knowledge)
- OpenTofu is Terraform-compatible but open source (no HashiCorp BSL licensing concerns)
- bpg/proxmox provider has best Proxmox LXC/VM support
- Strong community, same ecosystem as Terraform providers

**Rejected**: Pulumi (too complex for current skill level), Terraform (BSL license)

---

## ADR-002: SOPS + age over HashiCorp Vault / AWS KMS

**Date**: 2025-07  
**Status**: Accepted

**Context**: Needed secrets management for IaC credentials.

**Decision**: SOPS 3.9.4 + age encryption

**Rationale**:
- SOPS + age is cloud-independent — no dependency on AWS KMS (old account has $321 bill history)
- age is simple — single keypair, stored in Bitwarden
- Works offline, no API calls for encryption/decryption
- Git-friendly — encrypted files can be committed safely

**Rejected**: 
- AWS KMS (old account risk, cloud dependency)
- HashiCorp Vault (overkill for current scale, another service to manage)
- Email-based secrets (SMTP2GO logs + Gmail retention = security risk)

---

## ADR-003: S3 Backend for OpenTofu State

**Date**: 2025-07  
**Status**: Accepted

**Context**: Needed remote state backend for OpenTofu.

**Decision**: AWS S3 `thelightville-critical-backups` bucket

**Rationale**:
- S3 is within AWS free tier until ~Jan 2027
- State lock via DynamoDB (or native S3 conditional writes in OpenTofu 1.10+)
- All modules use separate key prefixes to avoid state conflicts

**Future consideration**: Migrate to Cloudflare R2 before Jan 2027 free tier expiry (zero egress costs)

---

## ADR-004: Cloudflare Tunnel Architecture (Inverted Ingress)

**Date**: Pre-IaC (existing)  
**Status**: Accepted

**Context**: Needed public access to 148 subdomains without exposing home WAN IP or opening firewall ports.

**Decision**: Cloudflare Tunnel (outbound from pve → Cloudflare edge)

**Rationale**:
- No inbound ports required on Sophos XG
- Cloudflare handles DDoS, WAF, SSL termination
- Single cloudflared daemon on pve routes all 148 subdomains via Nginx

---

## ADR-005: Oracle Inverted VPN Architecture

**Date**: Pre-IaC (existing)  
**Status**: Accepted

**Context**: Needed static public IP for non-Cloudflare services without paying for static ISP IP.

**Decision**: Oracle E2.1.Micro VM as VPN endpoint with IPsec back to home Sophos XG

**Rationale**:
- Oracle E2.1.Micro is Always Free tier — $0/month
- Provides static IP 40.233.116.184
- IPsec tunnels from home Sophos → Oracle DRG → Oracle VCN

---

## ADR-006: CT119 Destroy Strategy

**Date**: 2025-07  
**Status**: Accepted

**Context**: CT119 CyberPanel is corrupt and has no production traffic.

**Decision**: Import without `prevent_destroy`, destroy via normal tofu PR once CT117 migration confirmed

**Rationale**:
- Never attempting to repair a corrupt control panel with active data risk
- Clean destruction via IaC is safer than manual pct destroy

---

## ADR-007: Sophos XG — No IaC Provider

**Date**: 2025-07  
**Status**: Accepted (workaround)

**Context**: Sophos XG has no OpenTofu/Terraform provider.

**Decision**: Manual config exports to private git repo `sophos-config-backups`

**Rationale**:
- No viable automated option exists
- Manual + versioned backup is safer than no backup
- Weekly export added to RUNBOOKS.md
