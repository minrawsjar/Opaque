<div align="center">

# Opaque × The Graph

### Privacy coordination for post-quantum private settlement

**The Graph orchestrates Opaque’s complete privacy setup. It is the privacy coordination layer—the brain of Opaque’s privacy mechanism—turning live public conditions into better decoys, stronger relay paths, and smarter settlement timing.**

</div>

![Opaque × The Graph privacy feedback loop](../diagrams/graph.svg)

<div align="center">

[`Live Arc subgraph`](https://api.studio.thegraph.com/query/1760100/opaque/v0.3.0) · [`Privacy Sentinel`](https://<sentinel-domain>/healthz) · [`MCP endpoint`](https://<sentinel-domain>/mcp) · [`Schema`](../graph/schema.graphql) · [`Graph Client composition`](../graph/companion)

</div>

---

## Opaque Privacy Sentinel

**Privacy Sentinel is Opaque’s public Graph intelligence product.** It uses
The Graph Client to compose live Arc privacy conditions with an independent
Arbitrum USDC context, then exposes the combined public state for judges,
agents, and developers.

```text
Arc Studio ───────────────────────┐
                                 ├─→ Graph Client ─→ Privacy Sentinel ─→ /mcp
Arbitrum USDC Graph source ──────┘
```

It answers one useful question in real time: **are the public conditions that
give an Opaque payment cover actually strong right now?** The full signal model
and interface appear below; the payment rail continues to use that same Graph
intelligence for decoys, routing, and privacy-window settlement.

---

## A privacy intelligence loop

Most privacy systems make one payment private in isolation. Opaque uses The
Graph to learn from the public environment around every payment, so the next
one can enter a deeper ring, take a less predictable route, and settle under
stronger cover.

```text
Arc events ─→ Opaque Studio ─→ privacy coordination signals
                                      ├─ wallet ranks best eligible decoys
                                      ├─ mesh makes a diverse Markov route
                                      └─ CRE evaluates the privacy window
                                                   ↓
                                             next settlement
                                                   ↓
                                      fresh public events close the loop
```

The Graph is not a reporting layer. It is the shared state that makes Opaque’s
ring, mesh, and settlement mechanisms adapt together.

| Privacy mechanism | Graph signal | What improves |
|---|---|---|
| Eight-member ring | Same-denomination depth, ring reuse, coarse concentration, note age. | The wallet ranks the strongest eligible seven decoys locally. |
| Three-hop relay mesh | Reliability, batch occupancy, recent selection pressure, operator diversity. | A local Markov walk builds a less predictable path. |
| Conditional settlement | Versioned ring and mesh readiness. | CRE settles as soon as privacy is strong, or at the payer’s deadline. |

---

## Composed live Graph infrastructure

```text
Opaque Studio — Arc testnet
pools · ring pressure · relay health
                  ├─→ wallet / mesh / CRE policy
                  │
                  └─→ The Graph Client ─→ Privacy Sentinel / MCP
                                            public Arc privacy context

EVM Privacy Signals Substreams ───────→ reusable aggregate-only data product
```

| Graph product | Opaque use | Evidence |
|---|---|---|
| **Subgraph Studio** | Live Arc pool and relay events power the core privacy coordination loop. | [`opaque/v0.3.0`](https://api.studio.thegraph.com/query/1760100/opaque/v0.3.0) |
| **The Graph Client** | Provides a typed, composable query layer over Opaque’s live Arc privacy state. | [`graph/companion`](../graph/companion) |
| **Subgraph MCP** | Serves the composed context to judges, agents, and developers. | [`/mcp`](https://<sentinel-domain>/mcp) |
| **Substreams** | Reusable EVM module emits the same aggregate privacy-signal contract from pool and relay events. | [`indexer/privacy-signals`](../indexer/privacy-signals) |

This is a native Graph composition: Arc Studio is Opaque’s operational privacy
source of truth; the Arbitrum USDC source is an independent contextual signal;
Graph Client composes both; and Privacy Sentinel exposes the combined public
context. The sidecar enriches inspection but never controls an Opaque payment.

---

## Composed Privacy Sentinel

Privacy Sentinel makes Opaque’s privacy intelligence directly inspectable. It
builds a typed public context from two live Graph query surfaces:

```text
OpaquePublicContext                 SettlementPublicContext
Studio / Arc                        Graph Network / Arbitrum USDC
├─ pool + ring signals              ├─ indexed block freshness
└─ relay health + diversity         └─ public transfer context
                 │                         │
                 └────── Graph Client ─────┘
                               ↓
                    combined public context
                               ↓
                     Privacy Sentinel / MCP
```

| Context signal | Meaning |
|---|---|
| `privacyScore` | The weaker of ring freshness and mesh health—the privacy layer is only as strong as its weakest side. |
| `ringFreshness` | Whether same-denomination cover is deep enough for an eight-member ring. |
| `meshHealth` | Whether reliable, operator-diverse relays are available for a three-hop path. |
| `contextStatus` | Whether the combined context is current, degraded, or stale. |
| `settlementStatus` | Whether the independent Arbitrum public context is current, stale, or unavailable. |
| `recentTransfers` | A bounded liveness signal from the independent public USDC source. |

The public MCP interface provides three direct views of this model:

| Tool | Returns |
|---|---|
| `get_privacy_context` | Current aggregate ring, mesh, and sidecar context. |
| `explain_privacy_readiness` | Why observed privacy conditions are strong, weak, or stale. |
| `list_public_privacy_signals` | The signal families Opaque uses to coordinate privacy. |

Judges can inspect the conditions that improve Opaque’s decisions without
needing to reproduce or disclose a user payment.

---

## How Graph strengthens every layer

### Better decoys, not blind randomness

For each fixed-denomination payment, the wallet reads the relevant public pool
snapshot and ranks eligible candidates by low reuse, low coarse concentration,
healthy activity, and age diversity. It selects the best seven candidates
locally; the real note never leaves the wallet.

```text
same-denomination pool
        ↓
discard exhausted / invalid candidates
        ↓
rank by reuse · concentration · activity · age diversity
        ↓
7 best eligible decoys + local note = 8-member ring
```

### Adaptive mesh diversity

Relay health is not a static allowlist. The wallet uses Graph-indexed occupancy
and selection pressure in a local Markov policy:

```text
weight(relay) = batch occupancy / (1 + recent selection count)

hop 1 → best eligible transition
hop 2 → eligible transition with a different operator
hop 3 → eligible transition with a third operator
```

Active cover traffic receives positive weight, while an overused relay or
operator loses relative priority and cannot become a repeatable fingerprint.

### Privacy as a measurable settlement condition

CRE evaluates the same versioned public policy used by the wallet:

```text
ringFreshness = min(10000, poolSize × 10000 / 8)
meshHealth    = min(relay capacity, mean reliability)
privacyScore  = min(ringFreshness, meshHealth)
```

`min` is deliberate: a deep pool cannot compensate for a weak mesh, and a
healthy mesh cannot compensate for insufficient same-denomination cover.
When the score reaches the payer’s sealed threshold, settlement proceeds;
otherwise it waits until the latest-settlement deadline.

---

## Privacy-aware indexing

Opaque indexes the crowd, never the private choice made from it.

| Indexed public information | Not part of the model |
|---|---|
| Pool population by denomination | Real signer or payment linkage |
| Aggregate ring reuse and note-age signals | Final ring composition |
| Coarse funding concentration bucket | Funder-to-note link |
| Relay reliability, occupancy, operator diversity | Message origin or selected route |
| Directory version and freshness | Recipient or sealed intent |

---

## Reusable EVM Privacy Signals

[`indexer/privacy-signals`](../indexer/privacy-signals) packages Opaque’s
event model as an aggregate-only EVM Substreams module. It consumes public
`Deposited`, `RingUsed`, `RelayAnnounced`, and `RelayHealth` events and emits
one typed `PrivacySignalBlock` per block.

Any EVM payment pool and relay network that implements the same event ABI can
reuse this signal contract to build privacy coordination without inheriting
Opaque’s wallet, settlement, or backend.

---

## Verify it

| Surface | Link |
|---|---|
| Live Arc subgraph | [`opaque/v0.3.0`](https://api.studio.thegraph.com/query/1760100/opaque/v0.3.0) |
| Sentinel health | [`/healthz`](https://<sentinel-domain>/healthz) |
| Sentinel MCP | [`/mcp`](https://<sentinel-domain>/mcp) |
| GraphQL schema | [`graph/schema.graphql`](../graph/schema.graphql) |
| Graph Client configuration | [`graph/companion/.graphclientrc.yml`](../graph/companion/.graphclientrc.yml) |
| Sentinel implementation | [`graph/sentinel`](../graph/sentinel) |
| Substreams module | [`indexer/privacy-signals`](../indexer/privacy-signals) |

> **Opaque turns live Graph data into a feedback loop for privacy: deeper
> cover, more diverse routes, and a better moment to settle—on every payment.**
