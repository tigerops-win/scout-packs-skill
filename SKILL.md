---
name: scout-packs-skill
description: Pay-per-call B2B data API for AI agents: verified lead email lookup, email deliverability scoring, tech-stack detection, domain intel, SSL/DNS/subdomain recon. Each call costs $0.01-$0.03 USDC on Base via x402 (402 -> sign -> retry); no account, no API key. Use when you need verified company or contact data with provenance for outreach, lead enrichment, or research workflows.
---

# Scout Packs — Pay-Per-Call B2B Data for AI Agents

Buy verified B2B data with USDC on Base. No account. No API key. No subscription.
Every call is a pay-per-call x402 transaction.

**Operator:** Tiger Operations — https://github.com/tigerops-win/scout-packs

## When to use this skill

- Your outreach workflow needs a verified lead list or a real, published contact email
- Your research agent needs company/domain data with source-URL provenance
- You want per-call pricing ($0.01–$0.03) instead of a monthly data seat
- You need email verification, tech-stack detection, or DNS/SSL reconnaissance on a domain

## Services and pricing (USDC on Base, eip155:8453)

| Service | Path | Price | What you get |
|---|---|---|---|
| Verified lead email lookup | `GET /lookup?query=<company or domain>` | $0.01 | Published business email + company enrichment |
| Email deliverability | `GET /deliverability?target=<email or domain>` | $0.03 | MX/SPF/DMARC, disposable/role detection, bounce-risk score |
| Domain intel | `GET /domain-intel?domain=<domain>` | $0.02 | RDAP registration data + DNS infra signals |
| Tech-stack detection | `GET /tech-stack?domain=<domain>` | $0.01 | Stack detected from headers + markup |
| Email pattern | `GET /email-pattern?domain=<domain>` | $0.01 | Likely corporate email patterns + MX check |
| SSL check | `GET /ssl-check?domain=<domain>` | $0.01 | SSL/TLS details + risk flags |
| Security headers audit | `GET /headers?domain=<domain>` | $0.01 | HSTS/CSP/X-Frame-Options, A–F grade |
| DNS records | `GET /dns?domain=<domain>` | $0.01 | Full record dump (A, AAAA, MX, TXT, NS, CNAME) |
| Subdomain intel | `GET /subdomains?domain=<domain>` | $0.02 | CT-log footprint, intent-classified |
| Contact page scrape | `GET /contact-page?domain=<domain>` | $0.02 | Published emails (role vs personal), phone numbers |
| Mail provider | `GET /mail-provider?domain=<domain>` | $0.01 | Mailbox host (Google Workspace, Microsoft 365, …) |
| Repo health | `GET /repo-health?repo=<owner/name>` | $0.01 | Stars, forks, open issues, license, 90-day velocity |
| Pack-size resolver | `GET /packsize?title=<product title>` | $0.02 | Pack count, unit size, unit price |
| Lead packs (25/50/100) | `GET /packs/25` | $0.01/pack | Pre-built verified lead packs (JSON) |

Every lead/email record: company name, contact name, title, **published** business
email, source URL where it was found. If an email isn't published, it isn't in the
data. Nothing guessed, nothing pattern-matched.

## How to pay (x402 flow)

1. **Call the endpoint unpaid** — it returns `HTTP 402` with payment terms:
   ```
   curl -s -i "https://scout-packs-production.up.railway.app/lookup?query=acme.com"
   # HTTP/1.1 402 Payment Required
   # x402Version: 2, scheme: exact, network: eip155:8453 (Base)
   # payTo: <address from the 402 payload>
   # asset: USDC 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913
   # amount: 10000 (6-decimal USDC = $0.01)
   ```
2. **Sign an EIP-3009 authorization** for the exact USDC amount on Base to the
   `payTo` address, then retry the same URL with the signature in the
   `X-PAYMENT` (x402 v1) or `PAYMENT-SIGNATURE` (x402 v2) header.
   The payment is verified and settled via facilitator — no separate transfer needed.
3. **Receive the data** as JSON immediately.

**Fallback (manual tx):** send the exact USDC amount on Base to `payTo` yourself,
then redeem with the tx hash (`POST /fulfill` for packs, `/fulfill-lookup`,
`/fulfill-deliverability`, … per the OpenAPI).

## Example: verified email lookup

```
# 1. Unpaid request -> 402 with payment terms
curl -s -i "https://scout-packs-production.up.railway.app/lookup?query=stripe.com"

# 2. Sign EIP-3009 for 10000 (=$0.01) USDC on Base to the payTo address,
#    then retry with the payment signature header:
curl -s "https://scout-packs-production.up.railway.app/lookup?query=stripe.com" \
  -H "X-PAYMENT: <base64 x402 payment payload>"

# 3. Response: {"company": "...", "contact": "...", "email": "...", "sources": [...]}
```

## Free discovery (no payment)

- Catalog: `GET https://scout-packs-production.up.railway.app/catalog`
- Payment manifest: `GET https://scout-packs-production.up.railway.app/.well-known/x402`
- Agent docs: `GET https://scout-packs-production.up.railway.app/llms.txt`
- OpenAPI: `GET https://scout-packs-production.up.railway.app/openapi.json`
- Redacted preview: `GET https://scout-packs-production.up.railway.app/packs/25/preview`
- MCP endpoint: `https://scout-packs-production.up.railway.app/mcp` (streamable HTTP)

## MCP (alternative interface)

The same 13 services are exposed as MCP tools at `/mcp`:
`list_packs`, `buy_pack`, `lookup_lead`, `tech_stack`, `email_pattern`, `ssl_check`,
`headers`, `dns`, `subdomains`, `contact_page`, `mail_provider`, `repo_health`,
`deliverability`, `domain_intel`, `packsize`.
Connect any MCP client to the streamable-HTTP endpoint and pay per call via x402.

## Rules

- Never invent email addresses. If a lookup returns no published email, say so.
- Read prices from `/catalog` before quoting them — prices are per-call and may change.
- Amounts in 402 payloads are 6-decimal USDC (10000 = $0.01).
