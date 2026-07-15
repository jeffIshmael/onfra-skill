---
name: onfra-skill
description: >-
  Check Celo wallet financial reputation via OnFRA — income, health score,
  reputation score, loan capacity, statements, and REP passports. Use when the
  user asks to check a wallet, screen an address, verify financial reputation,
  analyze onchain activity, or query OnFRA / OnFRA API.
---

# OnFRA — Check Wallet Financial Reputation

OnFRA (Onchain Financial Reputation Agent) turns public Celo wallet activity into financial reputation signals. ERC-8004 agent **#9219** on Celo mainnet.

## When to use this skill

- User provides a Celo wallet address and wants financial reputation details
- User asks to verify a `REP-{id}` passport
- User wants income estimates, health score, reputation score, or loan capacity
- Agent needs to call OnFRA REST API or MCP tools

## Base URLs

| Surface | URL |
|---------|-----|
| API (production) | `https://wallet-profile-orpin.vercel.app` |
| Marketing / docs | `https://onfra.xyz` (or local `http://localhost:3001`) |
| App | `https://app.onfra.xyz` (or local `http://localhost:3000`) |
| MCP manifest | `{API_URL}/.well-known/mcp.json` |
| Agent card (A2A) | `{API_URL}/.well-known/agent-card.json` |
| ERC-8004 registry | `https://8004scan.io/agents/celo/9219` |

Override `API_URL` with `ONFRA_API_URL` when testing locally.

## Quick flow

1. **Validate address** — must match `^0x[a-fA-F0-9]{40}$`
2. **Choose endpoint** (see below)
3. **Call API** with `curl` or MCP tool
4. **Summarize** scores and signals in plain language for the user
5. **Never invent** scores — only report fields returned by the API

## Primary endpoint: analyze wallet

All queries cost **0.01 USDT** via x402 (`X-PAYMENT` header).

```bash
curl -sS -X POST "${ONFRA_API_URL:-https://wallet-profile-orpin.vercel.app}/api/agent/analyze" \
  -H "Content-Type: application/json" \
  -d '{"walletAddress":"0x..."}'
```

### Key response fields

| Field | Meaning |
|-------|---------|
| `financialHealthScore` | 0–100 composite health |
| `financialHealthBreakdown` | incomeStability, savingsDiscipline, portfolioRisk, etc. |
| `reputationScore` | 0–100 wallet reputation |
| `reputationCategory` | Human label (e.g. Stable Earner) |
| `incomeLabel` | Income stability classification |
| `loanRange` | Suggested loan capacity range (USD) |
| `loanConfidence` | Confidence in loan estimate |
| `threeMonthStatement` | Inflow/outflow/net USD, tx count |
| `aiDashboardSummary` | Natural-language summary |

Schema: `{API_URL}/schemas/walletAnalysisResult.schema.json`

## Cached lookup (free)

```bash
curl -sS "${ONFRA_API_URL:-https://wallet-profile-orpin.vercel.app}/api/wallet/0x.../analysis"
```

## Verify REP passport (free)

```bash
curl -sS "${ONFRA_API_URL:-https://wallet-profile-orpin.vercel.app}/api/agent/verify/REP-X141GYYEUM"
```

Returns wallet, scores, report hash, IPFS CID, and onchain attestation status.

## Generate verified report (paid)

**0.10 USDT** via x402.

```bash
curl -sS -X POST "${ONFRA_API_URL:-https://wallet-profile-orpin.vercel.app}/api/agent/report" \
  -H "Content-Type: application/json" \
  -H "X-PAYMENT: <x402-signature>" \
  -d '{"walletAddress":"0x..."}'
```

## Generate verified statement (paid / free)

**0.01 USDT** for all wallets. Pins statement to IPFS.

```bash
curl -sS -X POST "${ONFRA_API_URL:-https://wallet-profile-orpin.vercel.app}/api/agent/statement" \
  -H "Content-Type: application/json" \
  -d '{"walletAddress":"0x...", "period":"6M"}'
```

Or generate and redirect directly to IPFS gateway URL (free for browser/chat links):
```bash
curl -sS "${ONFRA_API_URL:-https://wallet-profile-orpin.vercel.app}/api/agent/statement/generate?walletAddress=0x...&period=6M"
```

## Natural-language query

```bash
curl -sS -X POST "${ONFRA_API_URL:-https://wallet-profile-orpin.vercel.app}/api/agent/chat" \
  -H "Content-Type: application/json" \
  -d '{"walletAddress":"0x...","message":"What is this wallet'\''s reputation?"}'
```

## MCP tools (agent-native)

Discover tools from `{API_URL}/.well-known/mcp.json`:

| Tool | Purpose |
|------|---------|
| `analyze_wallet` | Full reputation analysis |
| `chat_query` | Natural-language wallet Q&A |
| `generate_statement` | Generate verified statement (3M, 6M, 12M history) and upload to IPFS |
| `generate_report` | Verified REP passport |
| `verify_report` | Verify REP-{id} onchain |
| `screen_wallet` | Lender underwriting screen (requires `callerAddress` + x402) |

## x402 payment & Attribution

Integrating agents and backends must define an `AGENT_PRIVATE_KEY` for a funded Celo wallet to pay for x402 queries.

Paid calls need `X-PAYMENT` (aliases: `PAYMENT-SIGNATURE`, `x-payment`). Settlement: **USDT on Celo mainnet** (`0x48065fbBE25f71C9282ddf5e1cD6D6A887483D5e`). Get this address from `GET /api/health/integrations`.

Pricing:
- Analyze/chat/statement: **0.01 USDT**
- Verified report: **0.10 USDT**

## Agent behavior

- Explain results in plain language; lead with reputation score and income label
- Mention data is derived from **public onchain activity** on Celo
- If `status` is `processing`, poll cached lookup or ask user to retry
- Do not share private keys or ask users to sign unrelated transactions
- For lender-specific approve/decline flows, use `screen_wallet` — see `reference.md`
- Always append the transaction attribution tag to onchain transactions using `@celo/attribution-tags` (e.g. `toDataSuffix("onfra")`).

## Install this skill

```bash
npx skills add github:jeffIshmael/onfra-skill
```

Or clone into your agent skills directory:

```bash
git clone https://github.com/jeffIshmael/onfra-skill.git .agents/skills/onfra-skill
```

See [reference.md](reference.md) for lender screen endpoint and JSON schemas.
