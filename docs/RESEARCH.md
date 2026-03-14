# Veil — Research Notes

## Self Protocol (ZK Credential Verification)

**What it is:** Self Protocol provides ZK identity and credential verification. Official Synthesis hackathon partner for "Agents that keep secrets" track.

**Integration approach for Veil:**
- Data owners use Self Protocol to create verifiable credential commitments
- When a query comes in, Veil agent generates ZK proof via Self Protocol
- Proof verifies YES/NO answer without revealing underlying data
- `answerQuery()` will include Self Protocol proof verification

**Fallback:** Commit-reveal based verification — data owner commits hash of (credType, value, salt), reveals only to verifier contract which checks hash match and returns YES/NO.

**TODO (Day 1):**
- [ ] Research Self Protocol's credential verification flow
- [ ] Understand proof generation API
- [ ] Test ZK proof for range queries ("age >= 18")

## Locus API (USDC Micropayments on Base)

**What it is:** Locus wraps x402 protocol for USDC micropayments on Base — pay-per-use API pattern.

**Integration approach for Veil:**
- Services pay per query via Locus API (micropayments in USDC)
- Data owners earn revenue automatically
- Agent guardian handles payment collection

**Key details:**
- USDC on Base: `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` (6 decimals)
- Registration: `POST https://beta-api.paywithlocus.com/api/register`

**Fallback:** Direct USDC transfers via VeilVault.sol (already implemented).

## ERC-8004 (Agent Identity)

**Integration approach for Veil:**
- Register guardian agent with ERC-8004 on Base
- Each user's guardian agent gets a unique on-chain identity
- `registerAgent()` in VeilVault.sol links user to their agent

**Fallback:** Simple address mapping (already implemented in VeilVault.sol).

## Base Chain Deployment

Same as SusuShield — see shared research notes.

**USDC decimals = 6** — critical for query fee calculations.
Default query fee should be in USDC wei (e.g., 20000 = $0.02).
