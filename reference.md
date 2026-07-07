# OnFRA API reference (skill supplement)

Base URL: `https://wallet-profile-orpin.vercel.app`

## JSON schemas

| Schema | Path |
|--------|------|
| Analysis request | `/schemas/walletAnalysisRequest.schema.json` |
| Analysis result | `/schemas/walletAnalysisResult.schema.json` |
| Chat request | `/schemas/chatRequest.schema.json` |
| Report request | `/schemas/reportRequest.schema.json` |
| Report result | `/schemas/reportResult.schema.json` |
| Lender screen request | `/schemas/lenderScreenRequest.schema.json` |
| Lender screen result | `/schemas/lenderScreenResult.schema.json` |

## Lender screen (optional)

For approve / review / decline underwriting signals:

```bash
curl -sS -X POST "${ONFRA_API_URL}/api/lender/screen" \
  -H "Content-Type: application/json" \
  -H "X-PAYMENT: <x402-signature>" \
  -d '{
    "walletAddress": "0xBorrower...",
    "callerAddress": "0xLender..."
  }'
```

Response includes `recommendation`, `signals.positive`, `signals.concerns`, `income`, `loanRange`.

## Health & stats (free)

```bash
curl -sS "${ONFRA_API_URL}/api/health/integrations"
curl -sS "${ONFRA_API_URL}/api/stats"
```

## Environment variables for agents

| Variable | Purpose |
|----------|---------|
| `ONFRA_API_URL` | Override API base (default: production URL above) |
| `X-PAYMENT` | x402 signature header for paid endpoints |

## Error handling

- `402 Payment Required` — include valid x402 `X-PAYMENT` header
- `400` — invalid wallet address or missing required fields
- `404` — no cached analysis or unknown REP id
- `status: "processing"` — analysis in flight; retry cached lookup after a few seconds
