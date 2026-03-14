# Veil — Research Notes

## Self Protocol (ZK Credential Verification)

**CRITICAL: Self Protocol is Celo-only.** No Base deployment. For Veil on Base, use **off-chain verification** via `SelfBackendVerifier` (Node.js, chain-agnostic).

**Self Protocol uses Groth16 ZK-SNARKs** to verify passport attributes (age, nationality) without revealing raw data. User scans passport NFC, proof generated locally, verified server-side.

**Integration for Veil (off-chain path):**
1. Data owner verifies identity via Self Protocol (passport NFC scan)
2. `SelfBackendVerifier` validates proof server-side
3. Server attests credential commitments to VeilVault.sol on Base
4. Queries answered with off-chain proof + onchain YES/NO result

**npm:** `@selfxyz/core`, `@selfxyz/qrcode`

**Practical reality:** Self Protocol verifies passport-based identity (age, nationality). For Veil's broader credential types (credit range, income), we'll use the **commit-reveal scheme already in VeilVault.sol** — data owner commits hash, agent verifies locally, answers YES/NO. Self Protocol adds the proof-of-personhood layer on top.

**Fallback:** Pure commit-reveal for all credential types (no Self Protocol dependency).

## Locus API (USDC Micropayments on Base)

**Beta self-registration:**
```bash
curl -X POST https://beta-api.paywithlocus.com/api/register \
  -H "Content-Type: application/json" \
  -d '{"name": "Veil Guardian Agent"}'
# Returns: apiKey, ownerPrivateKey, ownerAddress
# Default: 10 USDC allowance, 5 USDC max/tx
```

**Key endpoints for Veil query payments:**

| Endpoint | Use Case |
|----------|----------|
| `POST /api/pay/send` | Distribute query revenue to data owner |
| `GET /api/pay/balance` | Check guardian agent balance |
| `POST /api/x402/call` | Pay-per-query via x402 protocol |

**x402 integration for Veil queries:**
Services call Veil guardian's API endpoint → get 402 response → pay via x402 → get YES/NO answer. Locus wraps this via `/api/x402/call`.

**Alternative:** Use `x402-express` middleware directly to paywall the guardian's query endpoint:
```typescript
import { paymentMiddleware } from "x402-express";
app.use(paymentMiddleware("0xGuardianWallet", {
  "POST /query": { price: "$0.02", network: "base" }
}, { url: "https://x402.org/facilitator" }));
```

**Gas is sponsored** by Locus. npm: `locus-agent-sdk`

## ERC-8004 (Agent Identity) — LIVE ON BASE

**Base contract addresses:**
- Identity Registry: `0x8004A818BFB912233c491871b3d84c89A494BD9e`
- Reputation Registry: `0x8004B663056A597Dffe9eCcC1965A193B7388713`

**Registration:** `register(agentURI)` → mints ERC-721 NFT → returns `agentId`

**npm:** `@agentic-trust/8004-sdk`, `@agentic-trust/8004-ext-sdk`

**For Veil:** Each guardian agent registers via ERC-8004. The `agentId` is stored in VeilVault.sol via `registerAgent()`. Services can verify the guardian's identity before submitting queries.

## Base Chain Deployment

- USDC: `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` (**6 decimals**)
- Default query fee: `20000` = $0.02 USDC (20000 / 10^6)
- Gas: ~$0.004 per transfer (sponsored by Locus for API calls)
- Chain ID: 8453, RPC: `https://mainnet.base.org`

**TODO (Day 1):**
- [ ] Register Locus beta agent
- [ ] Register ERC-8004 guardian identity on Base
- [ ] Implement x402 paywall on guardian query endpoint
- [ ] Write Deploy.s.sol for VeilVault
- [ ] Deploy to Base Sepolia first
