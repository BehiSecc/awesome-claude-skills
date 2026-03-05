---
name: octav-api
description: |
  Integrate with Octav API for cryptocurrency portfolio tracking, transaction history, and DeFi analytics across 65+ blockchain networks. Use when building applications that need to: (1) Track wallet balances and net worth across multiple chains, (2) Query transaction history with filtering and search, (3) Monitor DeFi protocol positions (Aave, Uniswap, etc.), (4) Access historical portfolio snapshots, (5) Analyze token distribution and holdings. Triggers on: "Octav API", "crypto portfolio API", "blockchain portfolio tracking", "DeFi analytics API", "wallet balance API", "transaction history API", "multi-chain portfolio".
license: MIT
metadata:
  author: Octav-Labs
  version: "1.0"
  website: https://octav.fi
---

# Octav API Integration

API for cryptocurrency portfolio tracking, transaction history, and DeFi analytics.

## Quick Reference

**Base URL:** `https://api.octav.fi`
**Auth:** Bearer token in Authorization header
**Rate Limit:** 360 requests/minute/key
**Pricing:** Credit-based ($0.02-0.025/credit)
**Dev Portal:** https://data.octav.fi

## Authentication

```bash
curl -X GET "https://api.octav.fi/v1/credits" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Store API key in environment variable `OCTAV_API_KEY`. Never hardcode.

## Endpoints Overview

| Endpoint | Method | Cost | Description |
|----------|--------|------|-------------|
| `/v1/portfolio` | GET | 1 credit | Portfolio holdings across chains/protocols |
| `/v1/nav` | GET | 1 credit | Net Asset Value (simple number) |
| `/v1/transactions` | GET | 1 credit | Transaction history with filtering |
| `/v1/token-overview` | GET | 1 credit | Token breakdown by protocol (PRO only) |
| `/v1/historical` | GET | 1 credit | Historical portfolio snapshots |
| `/v1/sync-transactions` | POST | 1+ credits | Trigger transaction sync |
| `/v1/status` | GET | Free | Check sync status |
| `/v1/credits` | GET | Free | Check credit balance |

## Core Endpoints

### Portfolio

Get holdings across wallets and DeFi protocols.

```javascript
const response = await fetch(
  `https://api.octav.fi/v1/portfolio?addresses=${address}`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const portfolio = await response.json();
// portfolio.networth, portfolio.assetByProtocols, portfolio.chains
```

**Parameters:**
- `addresses` (required): Comma-separated EVM/Solana addresses
- `includeNFTs`: Include NFT holdings (default: false)
- `includeImages`: Include asset/protocol image URLs (default: false)
- `waitForSync`: Wait for fresh data if stale (default: false)

**Response structure:**
```json
{
  "address": "0x...",
  "networth": "45231.89",
  "assetByProtocols": {
    "wallet": { "key": "wallet", "name": "Wallet", "value": "12453.20", "assets": [...] },
    "aave_v3": { "key": "aave_v3", "name": "Aave V3", "value": "8934.12", "assets": [...] }
  },
  "chains": {
    "ethereum": { "value": "25123.45", "protocols": [...] },
    "arbitrum": { "value": "20108.44", "protocols": [...] }
  }
}
```

### Nav (Net Asset Value)

Get simple net worth number.

```javascript
const response = await fetch(
  `https://api.octav.fi/v1/nav?addresses=${address}`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const nav = await response.json(); // Returns: 1235564.43434
```

### Transactions

Query transaction history with filtering.

```javascript
const params = new URLSearchParams({
  addresses: '0x...',
  limit: '50',
  offset: '0',
  sort: 'DESC',
  hideSpam: 'true'
});

const response = await fetch(
  `https://api.octav.fi/v1/transactions?${params}`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
```

**Required parameters:**
- `addresses`: Wallet address(es)
- `limit`: Results per page (1-250)
- `offset`: Pagination offset

**Optional filters:**
- `sort`: `DESC` (newest) or `ASC` (oldest)
- `networks`: Chain filter (e.g., `ethereum,arbitrum,base`)
- `txTypes`: Transaction type filter (e.g., `SWAP,DEPOSIT`)
- `protocols`: Protocol filter (e.g., `uniswap_v3,aave_v3`)
- `hideSpam`: Exclude spam (default: false)
- `hideDust`: Exclude dust transactions (default: false)
- `startDate`/`endDate`: ISO 8601 date range
- `initialSearchText`: Full-text search in assets

**Response (array of transactions):**
```json
[{
  "hash": "0xa1b2c3...",
  "timestamp": "1699012800",
  "chain": { "key": "ethereum", "name": "Ethereum" },
  "type": "SWAP",
  "protocol": { "key": "uniswap_v3", "name": "Uniswap V3" },
  "fees": "0.002134",
  "feesFiat": "7.12",
  "assetsIn": [{ "symbol": "WETH", "amount": "1.5", "value": "4800.00" }],
  "assetsOut": [{ "symbol": "USDC", "amount": "4795.23", "value": "4795.23" }]
}]
```

### Sync Transactions

Trigger manual sync for fresh transaction data.

```javascript
const response = await fetch('https://api.octav.fi/v1/sync-transactions', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ addresses: ['0x...'] })
});
// Returns: "Address is syncing" or "Address already syncing"
```

**Cost:** 1 credit + 1 credit per 250 transactions indexed (first-time only).

### Status (Free)

Check sync status before expensive operations.

```javascript
const response = await fetch(
  `https://api.octav.fi/v1/status?addresses=${address}`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const [status] = await response.json();
// status.portfolioLastSync, status.transactionsLastSync, status.syncInProgress
```

### Credits (Free)

Check remaining credit balance.

```javascript
const credits = await fetch('https://api.octav.fi/v1/credits', {
  headers: { 'Authorization': `Bearer ${apiKey}` }
}).then(r => r.json());
// Returns: 19033 (number)
```

### Historical Portfolio

Get portfolio snapshot for a specific date. Requires subscription.

```javascript
const response = await fetch(
  `https://api.octav.fi/v1/historical?addresses=${address}&date=2024-11-01`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
```

### Token Overview (PRO Only)

Detailed token breakdown by protocol.

```javascript
const response = await fetch(
  `https://api.octav.fi/v1/token-overview?addresses=${address}&date=2024-11-01`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
```

## Transaction Types

Common types for filtering:

| Type | Description |
|------|-------------|
| `TRANSFERIN` | Received tokens |
| `TRANSFEROUT` | Sent tokens |
| `SWAP` | Token exchange |
| `DEPOSIT` | DeFi deposit |
| `WITHDRAW` | DeFi withdrawal |
| `STAKE` | Staking tokens |
| `UNSTAKE` | Unstaking tokens |
| `CLAIM` | Reward claims |
| `ADDLIQUIDITY` | LP deposit |
| `REMOVELIQUIDITY` | LP withdrawal |
| `BORROW` | Lending protocol borrow |
| `LEND` | Lending protocol supply |
| `BRIDGEIN` / `BRIDGEOUT` | Cross-chain bridge |
| `APPROVAL` | Token approval |
| `MINT` | NFT/token minting |

## Supported Chains

**Full support (portfolio + transactions):** ethereum, arbitrum, base, polygon, optimism, avalanche, binance, solana, blast, linea, gnosis, sonic, starknet, fraxtal, unichain

**Portfolio only:** scroll, zksync (era), mantle, manta, fantom, cronos, celo, and 40+ more

Use chain keys in `networks` filter: `?networks=ethereum,arbitrum,base`

## Error Handling

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    const response = await fetch(url, options);

    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After') || 60;
      await new Promise(r => setTimeout(r, retryAfter * 1000));
      continue;
    }

    if (!response.ok) {
      const error = await response.json();
      throw new Error(`API Error ${response.status}: ${error.message}`);
    }

    return response;
  }
  throw new Error('Max retries exceeded');
}
```

**Common errors:**
- `401`: Invalid/missing API key
- `402`: Insufficient credits
- `403`: Endpoint requires PRO subscription
- `429`: Rate limit exceeded (wait and retry)
- `404`: Address not indexed (>100k transactions)

## Cost Optimization

1. **Batch addresses:** `?addresses=0x123,0x456,0x789` (1 credit vs 3)
2. **Use free endpoints:** `/v1/status` and `/v1/credits` cost nothing
3. **Filter on server:** Use `networks`, `txTypes` params vs client filtering
4. **Cache results:** Portfolio cached 1 minute, transactions 10 minutes
5. **Check status first:** Avoid unnecessary syncs

## Common Patterns

### Multi-wallet portfolio

```javascript
const addresses = ['0x123...', '0x456...', '0x789...'];
const response = await fetch(
  `https://api.octav.fi/v1/portfolio?addresses=${addresses.join(',')}`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
```

### Paginated transaction fetch

```javascript
async function getAllTransactions(address) {
  const transactions = [];
  let offset = 0;
  const limit = 250;

  while (true) {
    const response = await fetch(
      `https://api.octav.fi/v1/transactions?addresses=${address}&limit=${limit}&offset=${offset}&sort=DESC`,
      { headers: { 'Authorization': `Bearer ${apiKey}` } }
    );
    const batch = await response.json();
    if (batch.length === 0) break;
    transactions.push(...batch);
    offset += batch.length;
    if (batch.length < limit) break;
  }

  return transactions;
}
```

### Smart sync workflow

```javascript
async function smartSync(address) {
  // Check status first (free)
  const [status] = await fetch(
    `https://api.octav.fi/v1/status?addresses=${address}`,
    { headers: { 'Authorization': `Bearer ${apiKey}` } }
  ).then(r => r.json());

  const lastSync = new Date(status.transactionsLastSync);
  const minutesSinceSync = (Date.now() - lastSync) / 1000 / 60;

  if (minutesSinceSync > 10 && !status.syncInProgress) {
    await fetch('https://api.octav.fi/v1/sync-transactions', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${apiKey}`, 'Content-Type': 'application/json' },
      body: JSON.stringify({ addresses: [address] })
    });
  }
}
```

## TypeScript Interfaces

```typescript
interface Portfolio {
  address: string;
  networth: string;
  cashBalance: string;
  dailyIncome: string;
  dailyExpense: string;
  fees: string;
  feesFiat: string;
  lastUpdated: string;
  assetByProtocols: Record<string, Protocol>;
  chains: Record<string, Chain>;
}

interface Protocol {
  key: string;
  name: string;
  value: string;
  assets: Asset[];
}

interface Asset {
  balance: string;
  symbol: string;
  price: string;
  value: string;
  contractAddress?: string;
  chain?: string;
}

interface Transaction {
  hash: string;
  timestamp: string;
  chain: { key: string; name: string };
  from: string;
  to: string;
  type: string;
  protocol?: { key: string; name: string };
  value: string;
  valueFiat: string;
  fees: string;
  feesFiat: string;
  assetsIn: Asset[];
  assetsOut: Asset[];
  functionName?: string;
}
```

## Pricing

| Package | Credits | Price | Per Credit |
|---------|---------|-------|------------|
| Starter | 4,000 | $100 | $0.025 |
| Small Team | 100,000 | $2,500 | $0.025 |
| Intensive | 1,000,000 | $20,000 | $0.020 |

Credits never expire. First-time address indexing: 1 credit per 250 transactions.

## Resources

- **Dev Portal:** https://data.octav.fi
- **API Docs:** https://docs.octav.fi
- **Discord:** https://discord.com/invite/qvcknAa73A
- **Protocol List:** https://protocols.octav.fi
