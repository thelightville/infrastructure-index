# Domain Map

## Routing Architecture

All public domains route through:
```
[User] → Cloudflare DNS (orange cloud) → Cloudflare Tunnel → pve Nginx → Backend CT/VM
```

Exception: `mail.*` records are grey-cloud (DNS-only) for direct MX routing.

---

## Key Subdomains

| Subdomain | Backend | CT/Service | Notes |
|---|---|---|---|
| thelightville.com | CT101 or CT117 | cPanel / Docker | Root domain |
| mail.thelightville.com | CT201 | Mailcow | Grey cloud — direct MX |
| webmail.thelightville.com | CT201 | Mailcow | Orange cloud |
| radio.thelightville.com | CT103 | OnlineRadio | |
| stream.thelightville.com | CT109:8000 | AzuraCast | |
| app.thelightville.com | CT105 | ClaApp | |
| api.thelightville.com | CT107 | API Gateway | |
| grafana.thelightville.com | CT111:3000 | Grafana | |
| pbs.thelightville.com | CT115:8007 | PBS UI | Restrict to VPN |
| portainer.thelightville.com | CT117:9000 | Portainer | Restrict to VPN |
| ai.thelightville.com | VM100:3000 | Open WebUI | |
| chatbot.thelightville.com | CT113 | WhatsApp Chatbot | 502 — verify |

---

## cPanel Domains (CT101 → CT117 Migration)

The following 38 domains are hosted on CT101 cPanel, pending migration to CT117 Docker:

> **TODO**: Populate full list from cPanel Account Manager
> Run on CT101: `whmapi1 listaccts | grep 'domain:'`

Current status: Migration in progress — ALL cPanel domains must be migrated before CT101 can be destroyed.

---

## Managed via OpenTofu

Key subdomains are defined in `tofu/cloudflare/domains.yaml`.  
The full 148-subdomain list is managed via Cloudflare dashboard and should be imported:

```bash
cd tofu/cloudflare
tofu init
# Import each DNS record (use Cloudflare provider import)
# Or: Use cloudflare_record data source to reference existing records without managing them
```

---

## DNS Record Types

| Type | Count | Purpose |
|---|---|---|
| CNAME | 148 | All proxied subdomains → tunnel |
| MX | 34 | Mailcow per-domain email routing |
| TXT | 34+ | SPF, DKIM, DMARC per email domain |
| A | <10 | Direct IP records (Oracle VM, etc.) |

---

## Cloudflare Zone Structure

- **Primary zone**: thelightville.com (all main services)
- **Client zones**: ~37 additional domains hosted via cPanel (CT101)
- Total zones: 38+ domains managed in Cloudflare

> **Note**: When CT117 Docker migration completes, cPanel client sites will move to Traefik → CT117
> At that point, update `tofu/cloudflare/domains.yaml` with new backend IPs
