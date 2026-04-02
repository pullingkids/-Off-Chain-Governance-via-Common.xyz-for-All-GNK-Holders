# Token-Based Governance: Splitting Technical and Community Decisions

## Motivation

Gonka governance currently works through the standard Cosmos SDK `x/gov` module. Voting power is derived from **compute weight** — a score each host earns through Proof of Compute activity (nonces delivered, inference work completed). In practice, this means only active hosts (node operators) participate in governance, and their influence is proportional to their hardware contribution.

This creates a fundamental mismatch between decision type and decision-maker:

- **Hosts are infrastructure operators.** Their compute weight reflects real contribution to the network — inference capacity, uptime, reliability. Giving them authority over technical decisions makes sense.
- **Hosts are not the only stakeholders.** Treasury allocations, marketing budgets, ecosystem grants, and partnership decisions affect all GNK holders — miners who earned GNK through inference work, developers who use the network, and anyone who holds GNK. None of these participants have a formal voice in community decisions today.

As Gonka grows — more hosts, more developers, more GNK holders — this mismatch will only become more acute. Community decisions should be made by the community.

---

## Solution

**Split governance into two tracks based on decision type**, each with the appropriate voter set:

| Track | Decision Type | Who Votes | Mechanism |
|-------|--------------|-----------|-----------|
| Technical | Chain upgrades, parameter changes, security | Hosts (weighted by compute) | Existing `x/gov` (current system) |
| Community | Treasury, marketing, grants, partnerships | All GNK holders | New Token-Based DAO |

Hosts retain full authority over technical decisions — software upgrades, consensus parameters, node configurations. Their compute weight reflects real contribution to the network and is the right signal for protocol-level decisions.

All other decisions — how community funds are spent, which partnerships are approved, which marketing initiatives are funded — move to a **Token-Based DAO** where any GNK holder votes proportionally to their balance, with results enforced automatically by smart contract.

---

## Why This Is Feasible Now

CosmWasm is already active on Gonka mainnet. The community sale contract and liquidity pool contract confirm that the `x/wasm` module is operational. Token-Based DAOs on DAO DAO are CosmWasm smart contracts — no protocol upgrade is required to deploy them.

Gonka's development team already has experience deploying and maintaining CosmWasm contracts. This is not introducing new infrastructure — it is extending what already exists.

---

## Technical Details

### Current Architecture

```
All proposals → x/gov → Host vote (weighted by compute) → Execution
```

### Proposed Architecture

```
Technical proposals  → x/gov (unchanged)  → Host vote            → Execution
Community proposals  → Token-Based DAO    → All GNK holders vote → Smart contract execution
```

### Token-Based DAO Components

DAO DAO uses three CosmWasm contracts to power a Token-Based DAO:

| Contract | Role |
|---------|------|
| `dao-core` | Central registry, holds DAO configuration and treasury |
| `dao-proposal-single` | Proposal creation, voting period management, tally |
| `dao-voting-token-staked` | Reads GNK wallet balances, calculates voting weight |

The `dao-voting-token-staked` module snapshots GNK balances at the block when a proposal is created. Holding GNK in any wallet is sufficient to vote — no Cold Account Key or node required.

### Decision Classification

| Category | Examples | Track |
|----------|----------|-------|
| Chain upgrade | New binary version, Cosmovisor upgrade | Technical → `x/gov` |
| Parameter change | Epoch length, collateral requirements, PoC V2 settings | Technical → `x/gov` |
| Security | Emergency halts, critical fixes | Technical → `x/gov` |
| Treasury spend | Marketing partnerships, media deals, any spend >10K GNK | Community → Token-Based DAO |
| Grants | Ecosystem fund allocations, developer grants | Community → Token-Based DAO |
| Partnerships | Exchange listings, integration agreements, co-marketing | Community → Token-Based DAO |
| Community initiatives | Events, educational content, ambassador programs | Community → Token-Based DAO |

### Proposed Governance Parameters (Token-Based DAO)

| Parameter | Proposed Value | Rationale |
|-----------|---------------|-----------|
| Voting period | 7 days | Sufficient time for community awareness |
| Quorum | 5% of circulating GNK | Realistic given current holder distribution |
| Passing threshold | 50% Yes (excluding Abstain) | Standard majority |
| Veto threshold | 33% NoWithVeto | Consistent with Cosmos standard |
| Proposal deposit | 500 GNK | Spam prevention; refunded on pass or fail, burned on veto |

All parameters are adjustable by community vote after deployment.

---

## Implementation Plan

**Week 1–2 — Preparation and Testing**
1. Audit Gonka's existing CosmWasm deployment — confirm `x/wasm` permissions and available code IDs
2. Deploy and configure DAO DAO contract suite in a local environment
3. Configure `dao-voting-token-staked` against GNK token denom
4. Validate voting mechanics and automatic execution

**Week 3 — Governance Proposal**
5. Submit on-chain `x/gov` proposal to authorize DAO DAO contract deployment on mainnet
6. Community review period

**Week 4 — Mainnet Deployment**
7. Deploy contracts to mainnet
8. Transfer Community Pool treasury governance to Token-Based DAO
9. `x/gov` remains active for technical decisions — only community decisions migrate

**90-Day Parallel Period**
Both systems run simultaneously. Community builds confidence in the new DAO before full migration of governance authority.

---

## What Does Not Change

- Hosts retain full authority over technical decisions
- The existing governance interface remains for technical proposals
- Consensus, security, and upgrade decisions continue through `x/gov`
- No changes to host economics, rewards, or node operations

---

## Open Questions

1. **Treasury migration scope** — Should the full Community Pool migrate immediately, or start with a governance sub-fund (e.g. 10M GNK) to reduce risk?
2. **Deposit threshold** — 500 GNK proposed. Does this create a barrier for smaller community members to submit proposals?
3. **Quorum** — 5% of circulating supply proposed. Is this achievable given current participation rates?
4. **CEX holders** — GNK held on exchanges cannot sign governance messages. This is a known limitation of all on-chain governance systems. Out of scope for this proposal.
5. **Classification disputes** — Who decides if a proposal is "technical" or "community"? A clear classification rule is needed to prevent gaming.

---

## Budget

**Requested: To be determined after initial validation**

Initial deployment and configuration will be contributed by the proposing team. A separate budget proposal for mainnet deployment and contract audit will be submitted once the implementation scope is confirmed.

This proposal authorizes the migration plan. No treasury spend is requested at this stage.

---

## References

1. **DAO DAO Token-Based DAO** — architecture and contract documentation. Source: [docs.daodao.zone](https://docs.daodao.zone)
2. **DAO DAO contracts** — deployed contract code IDs across Cosmos chains. Source: [github.com/DA0-DA0/dao-contracts](https://github.com/DA0-DA0/dao-contracts)
3. **CosmWasm on Gonka** — confirmed active via community sale and liquidity pool contracts. Source: [gonka-ai/gonka releases](https://github.com/gonka-ai/gonka/releases)
4. **Cosmos SDK `x/gov`** — existing governance module, retained for technical decisions. Source: [docs.cosmos.network/main/modules/gov](https://docs.cosmos.network/main/modules/gov)
5. **Gonka Tokenomics** — GNK supply distribution, community pool. Source: [docs/tokenomics.md](https://github.com/gonka-ai/gonka/blob/main/docs/tokenomics.md)
