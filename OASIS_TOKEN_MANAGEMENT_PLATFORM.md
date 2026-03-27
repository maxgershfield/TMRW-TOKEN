# OASIS Token Management Platform
## Building a Magna-Equivalent Service on Our Backend

> **Reference:** Screenshots from Magna (app.magna.so) — W3AI demo workspace — are in `/Token-Management/references/`
> **Context:** Magna manages $10B+ TVL, starts at $500/month. Now powered by Kraken/Payward. We have the backend architecture to build a comparable (and in several ways superior) service.

---

## What Magna Does — Mapped to Our Screens

| Screenshot | Magna Feature | What It Shows |
|---|---|---|
| `magna-01-add-admins.png` | Admin Management | Add team members with role-based access (Workspace Admin, Edit Access) |
| `magna-02-add-wallet.png` | Custody Wallet | Add an admin wallet address used to deploy contracts & sign transactions |
| `magna-03-add-token.png` | Token Registry | Register a token (W3AI on Ethereum Sepolia), set total supply (2B), valuation |
| `magna-04-smart-contract.png` | Contract Binding | Bind a vesting/transfer smart contract to the token (Direct Transfer type) |
| `magna-05-token-cap-table.png` | Cap Table | Visual cap table: token status, mint date, total supply, allocated vs unallocated |
| `magna-06-allocations.png` | Allocations | Per-stakeholder allocations with wallet, category, status, and amount |
| `magna-07-smart-contract-2.png` | Contract Management | Multi-contract view per token, balance tracking, contract IDs |

**Magna's full product surface:**
1. Onchain Vesting (audited smart contracts, "set and forget")
2. Offchain Vesting & Tax (custody-based distributions, HRIS integrations)
3. Whitelabel Claim Portals (branded airdrop sites, no-code)
4. Grant Management (milestone tracking, KYC, wallet gathering)
5. Custody & Escrow (3rd-party custody, OTC, flexible signing)
6. Staking (custom branded pools, LSTs, claim-and-stake)

---

## Our Existing Endpoints That Already Cover This

### 1. Token Layer (`/api/token`)

| Endpoint | Method | Magna Equivalent |
|---|---|---|
| `/api/token/create-web4-token` | POST | Add Token (step 3) — creates a multi-chain token identity |
| `/api/token/add-web3-token-to-web4` | POST | Links a deployed contract address on a specific chain to the Web4 token |
| `/api/token/web4-token/{id}` | GET | Fetch token details + all chain representations |
| `/api/token/web4-tokens` | GET | List all tokens in workspace |

**Web4 advantage over Magna:** Our token is chain-agnostic from creation — one token ID that spans Ethereum, Solana, Arbitrum, Polygon etc. Magna is per-chain. This is a genuine differentiator.

---

### 2. Wallet Layer (`/api/wallet`)

| Endpoint | Method | Magna Equivalent |
|---|---|---|
| `/api/wallet/avatar/{id}/create-wallet` | POST | Add Admin Wallet (step 2) |
| `/api/wallet/avatar/{id}/wallets/false/false` | GET | List all wallets for a stakeholder |
| `/api/wallet/avatar/{id}/default-wallet` | GET/POST | Set signing/admin wallet |
| `/api/wallet/avatar/{id}/import/private-key` | POST | Import existing custody wallet |
| `/api/wallet/avatar/{id}/import/public-key` | POST | Import watch-only wallet |
| `/api/wallet/send_token` | POST | Direct transfer (mirrors Magna's "Direct Transfer" contract type) |
| `/api/wallet/transfer` | POST | Cross-wallet transfers |
| `/api/wallet/supported-chains` | GET | Returns all 10+ supported chains |
| `/api/wallet/avatar/{id}/portfolio/value` | GET | Portfolio valuation view |
| `/api/wallet/avatar/{id}/wallet/{wId}/tokens` | GET | Token balances per wallet |

---

### 3. Escrow Layer (`/api/escrow`)

| Endpoint | Method | Magna Equivalent |
|---|---|---|
| `/api/escrow/create` | POST | Custody & Escrow — create a new escrow contract |
| `/api/escrow/{id}` | GET | Fetch escrow status |
| `/api/escrow/avatar/{id}` | GET | All escrows for a stakeholder (filterable by status) |
| `/api/escrow/{id}/fund` | POST | Fund escrow (MNEE stablecoin) |
| `/api/escrow/{id}/release` | POST | Release to payee |

---

### 4. Vault Layer (`/api/vault`)

| Endpoint | Method | Magna Equivalent |
|---|---|---|
| `/api/vault/create-web4-vault` | POST | Custody vault creation — multi-chain vault identity |
| `/api/vault/add-web3-vault-to-web4` | POST | Add a specific chain's vault contract to the Web4 vault |
| `/api/vault/web4-vault/{id}` | GET | Get vault state |

---

### 5. Liquidity Layer (`/api/v1/liquidity`)

| Endpoint | Method | Magna Equivalent |
|---|---|---|
| `/api/v1/liquidity/pools` | POST | Staking pool creation |
| `/api/v1/liquidity/pools/{id}` | GET | Pool state — reserves, TVL, LP supply |
| + add/remove/swap | POST | Pool operations |

---

### 6. Avatar/Identity Layer (`/api/avatar`) — Powers Stakeholder Management

| What It Provides | Magna Equivalent |
|---|---|
| JWT auth + email verification | Login system for founders, investors, employees |
| Role system (Wizard/Admin) | Workspace Admin vs Edit Access roles |
| Avatar profiles with metadata | Stakeholder profiles (name, email, contact) |
| KarmaManager | Contribution/reputation tracking |
| Multi-avatar search | Stakeholder lookup by email, username, or ID |

---

## The Gaps — New Endpoints to Build

These are the Magna features we don't have endpoints for yet, in priority order:

---

### 🔴 Priority 1 — Vesting Schedules (Core Product)

Magna's most-used feature. No equivalent exists in OASIS today.

**New controller needed: `VestingController`**

```
POST   /api/vesting/schedules                    — Create vesting schedule
GET    /api/vesting/schedules/{id}               — Get schedule details
GET    /api/vesting/schedules/token/{tokenId}    — All schedules for a token
PUT    /api/vesting/schedules/{id}               — Update schedule
DELETE /api/vesting/schedules/{id}/cancel        — Cancel + reclaim tokens

POST   /api/vesting/schedules/{id}/claim         — Stakeholder claims vested tokens
GET    /api/vesting/schedules/{id}/claimable     — How much is claimable now
GET    /api/vesting/schedules/{id}/history       — Full unlock event history

POST   /api/vesting/onchain/deploy               — Deploy audited vesting smart contract
GET    /api/vesting/onchain/{contractAddress}    — Read live on-chain vesting state
```

**VestingSchedule model:**
```json
{
  "tokenId": "web4-token-guid",
  "stakeholderAvatarId": "avatar-guid",
  "walletAddress": "0x...",
  "totalAllocation": 1000000,
  "cliffDays": 365,
  "vestingDays": 1460,
  "vestingType": "Linear | Milestone | Custom",
  "startDate": "2026-01-01",
  "milestones": [],
  "categoryId": "team | investor | advisor | community",
  "contractAddress": "0x...",
  "chain": "EthereumOASIS"
}
```

---

### 🔴 Priority 2 — Token Cap Table (`CapTableController`)

The cap table view in `magna-05-token-cap-table.png` is a read aggregation across all allocations.

```
GET    /api/captable/token/{tokenId}             — Full cap table with ownership %
GET    /api/captable/token/{tokenId}/summary     — Summary: allocated, unallocated, total
GET    /api/captable/token/{tokenId}/ownership   — Ownership donut chart data
GET    /api/captable/token/{tokenId}/categories  — Breakdown by category (team, investors...)
POST   /api/captable/token/{tokenId}/export      — Export cap table as CSV
```

---

### 🔴 Priority 3 — Allocations (`AllocationController`)

The allocations view in `magna-06-allocations.png` — per-stakeholder allocations with wallet, status, schedule.

```
POST   /api/allocation/create                    — Add an allocation
GET    /api/allocation/{id}                      — Get allocation details
GET    /api/allocation/token/{tokenId}           — All allocations for a token
PUT    /api/allocation/{id}                      — Update allocation
DELETE /api/allocation/{id}/cancel               — Cancel allocation

POST   /api/allocation/import                    — Bulk import allocations (CSV)
GET    /api/allocation/token/{tokenId}/export    — Export as CSV (matches Magna's Export CSV)

POST   /api/allocation/{id}/notify               — Send notification to stakeholder
GET    /api/allocation/token/{tokenId}/stats     — Distribution stats
```

---

### 🟡 Priority 4 — Grant Management (`GrantController`)

Magna's grant management for ecosystem grant teams.

```
POST   /api/grant/create                         — Create a grant
GET    /api/grant/{id}                           — Get grant details
PUT    /api/grant/{id}/milestone/{mId}/complete  — Submit milestone completion evidence
POST   /api/grant/{id}/kyc                       — KYC submission for grant recipient
GET    /api/grant/token/{tokenId}                — All grants for a token
POST   /api/grant/{id}/distribute                — Trigger distribution
```

Note: Quest system in OASIS already handles milestone tracking — `QuestManager` can be extended to back this.

---

### 🟡 Priority 5 — Whitelabel Claim Portals (`ClaimPortalController`)

Powers the "claim your tokens" experience for airdrops and vesting.

```
POST   /api/claim-portal/create                  — Create a branded claim portal
GET    /api/claim-portal/{slug}                  — Public: load portal config by slug
POST   /api/claim-portal/{id}/whitelist          — Add/update whitelist
GET    /api/claim-portal/{id}/allocations        — All claimable allocations
POST   /api/claim-portal/{id}/claim              — Stakeholder claims tokens
GET    /api/claim-portal/{id}/analytics          — Claims stats, conversion, geography

ClaimPortalConfig {
  slug, title, description,
  logoUrl, primaryColor, brandUrl,
  claimWindowStart, claimWindowEnd,
  geofencing: ["US", "UK"],
  termsAndConditions,
  bonusClaimEnabled,
  postClaimAction: "stake | nft-mint | none"
}
```

Note: The whitelabel UI can live in our existing OPORTAL/CRE infrastructure. The claim portal backend just needs these endpoints + config stored as a Holon.

---

### 🟡 Priority 6 — Staking Pools (extend `LiquidityController`)

Liquidity pools exist but lack the staking-specific surface:

```
POST   /api/v1/liquidity/pools/{id}/stake        — Stake tokens
POST   /api/v1/liquidity/pools/{id}/unstake      — Unstake
POST   /api/v1/liquidity/pools/{id}/claim-rewards — Claim staking rewards
GET    /api/v1/liquidity/pools/{id}/stakers       — List stakers + amounts
GET    /api/v1/liquidity/pools/{id}/apy           — Current APY
POST   /api/v1/liquidity/pools/{id}/configure     — Set min stake time, max reward rate, pool size
```

---

### 🟢 Priority 7 — Workspace / Project Management

Magna uses a "Workspace" concept (we saw W3AI Demo workspace in the screenshots). OASIS holons map perfectly.

```
POST   /api/workspace/create                     — Create a token management workspace
GET    /api/workspace/{id}                       — Get workspace
POST   /api/workspace/{id}/invite                — Invite admin (maps to magna-01-add-admins.png)
PUT    /api/workspace/{id}/member/{avatarId}/role — Update role (Admin/Edit)
DELETE /api/workspace/{id}/member/{avatarId}     — Remove member
GET    /api/workspace/{id}/tokens                — All tokens in workspace
GET    /api/workspace/{id}/activity              — Audit log of all actions
```

This is a thin wrapper — a Workspace is just a named Holon with child holons for tokens/allocations/schedules.

---

## Architecture Diagram — How It All Fits

```
┌─────────────────────────────────────────────────────────┐
│                 OASIS TOKEN MANAGEMENT                   │
│              (Magna-equivalent service)                  │
├─────────────────┬───────────────────┬───────────────────┤
│   ADMIN LAYER   │   TOKEN LAYER     │  STAKEHOLDER LAYER│
│                 │                   │                   │
│ WorkspaceCtrl   │ TokenController   │ AvatarController  │
│ (new)           │ (exists ✅)       │ (exists ✅)        │
│                 │                   │                   │
│ Role mgmt       │ Web4Token         │ JWT auth          │
│ Member invites  │ (multi-chain)     │ KYC hooks         │
│ Audit log       │ CapTableCtrl      │ Email notifs      │
│                 │ (new)             │ Karma tracking    │
├─────────────────┴───────────────────┴───────────────────┤
│                   DISTRIBUTION LAYER                     │
│                                                         │
│  VestingController (new)    AllocationController (new) │
│  ├─ Onchain vesting         ├─ Per-stakeholder amounts  │
│  ├─ Offchain tracking       ├─ Bulk CSV import/export   │
│  ├─ Cliff + linear/milestone├─ Status tracking          │
│  └─ Claim endpoint          └─ Notification triggers    │
├─────────────────────────────────────────────────────────┤
│                    PORTAL LAYER                          │
│                                                         │
│  ClaimPortalController (new)   GrantController (new)   │
│  ├─ Whitelabel config          ├─ Milestone tracking    │
│  ├─ Public claim endpoint      ├─ KYC + wallet gather   │
│  ├─ Geofencing + T&Cs          └─ Backs into QuestMgr   │
│  └─ Post-claim hooks (stake)                            │
├─────────────────────────────────────────────────────────┤
│                   CUSTODY LAYER                          │
│                                                         │
│  WalletController (exists ✅)  EscrowController (✅)    │
│  VaultController (exists ✅)   LiquidityController (✅) │
│                                                         │
│  Integrations: Fireblocks, Safe, Squads, Anchorage     │
│  (via existing OASIS Provider system — plug-in)        │
├─────────────────────────────────────────────────────────┤
│              OASIS PROVIDER SYSTEM (multi-chain)        │
│                                                         │
│  Ethereum  Solana  Arbitrum  Polygon  Base  Avalanche  │
│  BNB  Optimism  Telos  EOSIO  Cardano  TON  + 20 more  │
└─────────────────────────────────────────────────────────┘
```

---

## OASIS Advantages Over Magna

| Dimension | Magna | OASIS |
|---|---|---|
| **Chain support** | ~10 EVM + Solana | 30+ chains (EVM, Solana, Cardano, TON, TRON, Hedera, XRP…) |
| **Token identity** | Per-chain only | Web4 Token = single ID across all chains |
| **Vesting contracts** | Audited, deployed for you | Same (to build) + open source |
| **Identity layer** | Email/wallet login only | Full Avatar system (karma, quests, credentials, holonic identity) |
| **Custody integrations** | Fireblocks, Anchorage, Safe, Squads | Same (via provider plugins) + any new chain |
| **Grant management** | Standalone module | Backed by QuestManager (milestone quests already exist) |
| **Claim portals** | Whitelabel via their platform | Whitelabel via OPORTAL / CRE infrastructure |
| **Impact tracking** | None | KarmaManager + Pan-Galactic Timebank (Hitchhikers) |
| **Pricing** | $500/month | TBD — but open source core = defensible |
| **Hackathon/community use** | Not designed for it | Native to Hitchhikers / Fabulous Machine |
| **AI agent integration** | None | MCP tools, A2A agents, Marvin |

**Key differentiator to lead with:** OASIS tokens aren't just managed — they're *holonic*. A token allocation can be linked to a Quest, a Karma score, a cross-game identity, and appear across multiple front-ends simultaneously. Magna manages tokens. OASIS makes tokens meaningful.

---

## Build Priority & Phasing

### Phase 1 — MVP (replicate Magna's onboarding flow)
Covers: `magna-01` through `magna-07` screenshots

1. `WorkspaceController` — project/admin setup
2. `AllocationController` — create, list, export
3. `VestingController` — schedule CRUD + claim endpoint
4. `CapTableController` — aggregation + ownership chart

**Estimated effort:** 4–6 new controller files + manager extensions. The data model maps directly to Holons — each vesting schedule is a Holon with the token Holon as parent.

### Phase 2 — Portal & Grants
5. `ClaimPortalController` — whitelabel claim experience
6. `GrantController` — extend QuestManager for grant milestones

### Phase 3 — Staking & Custody Extensions
7. Extend `LiquidityController` for staking-specific operations
8. Custody provider plugins (Fireblocks, Safe, Squads) — via existing provider architecture

---

## Immediate Next Step

The fastest path to a working demo is to implement `VestingController` and wire it to the existing `TokenController` + `WalletController`. The W3AI token is already set up in Magna's demo workspace — we can replicate that exact setup against our own API on Ethereum Sepolia (our configured testnet) in a single sprint.

The W3AI demo shows:
- Token: W3AI, Ethereum Sepolia, 2,000,000,000 supply, $0.0015 valuation
- 20,000,000 W3AI allocated (1% of supply), status: Not Started
- Wallet: `0x34A5f6acFb4F7f3C7A1abBF8Cb8B99ecb5a69042`
- Contract: `0xC8a7...0730` (Direct Transfer type)

This is exactly replicable with our existing `create-web4-token` + `create-wallet` + new `create-allocation` endpoints.
