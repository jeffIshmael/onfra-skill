# OnFRA API reference (skill supplement)

Base URL: `https://app.onfra.xyz`

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
| Wallet signal result | `/schemas/walletSignalResult.schema.json` |

## Granular signals (cache reads)

After `POST /api/agent/analyze`, read individual reputation fields for free:

```bash
# Cache status + signal ids
curl -sS "${ONFRA_API_URL}/api/wallet/0xBorrower.../signals"

# One signal
curl -sS "${ONFRA_API_URL}/api/wallet/0xBorrower.../signals/loan-capacity"
```

Signal ids: `monthly-income`, `financial-health`, `reputation-score`, `loan-capacity`, `statement`, `assessment`.

`POST /api/agent/analyze` accepts optional `fields` (body or `?fields=loanCapacity,reputationScore`) to return a subset without full `walletData`. One x402 charge per wallet refresh window on analyze; signal GETs do not charge.

## Lender screen (optional)

For approve / review / decline underwriting signals:

```bash
curl -sS -X POST "${ONFRA_API_URL}/api/lender/screen" \
  -H "Content-Type: application/json" \
  -H "X-PAYMENT: <x402-signature>" \
  -d '{
    "walletAddress": "0xBorrower..."
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
| `AGENT_PRIVATE_KEY` | Private key (Hex format) for signing x402 payments and onchain transactions |

## EIP-3009 Payment Example (TypeScript)

To construct the `X-PAYMENT` header for the Celo facilitator, agents should sign an EIP-3009 authorization using their `AGENT_PRIVATE_KEY` via `viem`. Also, to append the transaction attribution tag, use `@celo/attribution-tags`.

```typescript
import { createWalletClient, http, concat, type Hex } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { celo } from "viem/chains";
import { toDataSuffix } from "@celo/attribution-tags";

// 1. Setup the agent account
const account = privateKeyToAccount(process.env.AGENT_PRIVATE_KEY as Hex);
const client = createWalletClient({ account, chain: celo, transport: http() });

// 2. Fetch the required payTo treasury address
const { x402Status } = await fetch("https://app.onfra.xyz/api/health/integrations").then(r => r.json());
const payTo = x402Status.payTo;

// 3. Generate X-PAYMENT header (using viem signTypedData for EIP-3009)
// For demonstration only. Actual EIP-3009 requires constructing a standard ReceiveWithAuthorization domain & types.
const nonce = crypto.randomUUID();
const signature = await account.signTypedData({
  domain: { name: "USD Tether", version: "1", chainId: 42220, verifyingContract: "0x48065fbBE25f71C9282ddf5e1cD6D6A887483D5e" },
  types: { ReceiveWithAuthorization: [/* ... */] },
  primaryType: "ReceiveWithAuthorization",
  message: { /* ... */ }
});
const xPayment = JSON.stringify({ signature, nonce, ... });

// 4. Send a transaction with an attribution tag
const originalCalldata = "0x..."; // Contract execution payload
const taggedData = concat([originalCalldata, toDataSuffix("onfra")]);

const hash = await client.sendTransaction({
  to: "0x...", // Target smart contract
  data: taggedData,
  value: 0n,
  type: "legacy"
});
```

## Error handling

- `402 Payment Required` — include valid x402 `X-PAYMENT` header
- `400` — invalid wallet address or missing required fields
- `404` — no cached analysis or unknown REP id
- `status: "processing"` — analysis in flight; retry cached lookup after a few seconds
