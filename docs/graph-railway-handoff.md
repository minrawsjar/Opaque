# Opaque × The Graph — Deployment Handoff

This is the remaining deployment work for the Graph submission. It is isolated
from Opaque’s wallet, mesh, CRE, contracts, and payment services.

## Goal

Deploy **Privacy Sentinel** as a public Railway service. It composes Opaque’s
live Arc privacy state with an independent Arbitrum USDC context:

```text
Arc Studio = operational privacy source of truth
Arbitrum USDC Graph source = independent contextual signal
Graph Client = composes both
Privacy Sentinel = exposes the combined public context
```

The Arbitrum sidecar is intentional: it gives the Graph submission a genuine
cross-source composition and public settlement-liveness context. It never
authorizes, routes, delays, signs, or settles an Opaque payment.

## 1. Preserve the two-source composition

Keep these components together:

| Location | Change |
|---|---|
| `graph/companion/.graphclientrc.yml` | Keep both the live Opaque Studio / Arc source and the pinned Graph Network Arbitrum USDC source. |
| `graph/companion/documents/privacy-context.graphql` | Keep both `OpaquePublicContext` and `SettlementPublicContext` operations. |
| `graph/src/companion-context.ts` + `graph/sentinel/live-context.ts` | Keep Arc privacy readiness and independent settlement-context status separate in the typed output. |
| `graph/sentinel/http.ts` | Keep `/healthz` source status reporting for both providers. |
| `graph/test/*companion*` + `graph/test/sentinel-http.test.ts` | Preserve tests for current, stale, and unavailable sidecar states. |
| `graph/sentinel/Dockerfile` | Build the two-source Graph Client at startup from the Railway Gateway key. |

Then run:

```bash
cd graph
bun run companion:build:live
bun test
bun run typecheck
```

## 2. Create the Railway service

Create a **new service** from this repository. Do not attach it to any existing
Opaque payment service.

| Railway setting | Value |
|---|---|
| Builder | Dockerfile |
| Dockerfile path | `graph/sentinel/Dockerfile` |
| Build context | repository root |
| Healthcheck path | `/healthz` |
| Public networking | Enabled |

Set exactly one service variable through Railway’s secret UI:

```text
GRAPH_GATEWAY_API_KEY=<Graph Gateway API key>
```

This key is only for the independent Arbitrum Graph Network source. Do not add
RPC keys, wallet keys, `MESH_MASTER`, `CREDENTIAL_MAC`, CRE keys, or any
payment-stack secret. Railway provides `PORT` automatically.

## 3. Deploy and verify

Wait until the build is successful. Startup performs two actions:

```text
build Graph Client from the injected Gateway key
             ↓
start Privacy Sentinel HTTP server
```

After Railway assigns a public domain, run:

```bash
curl -sS https://<sentinel-domain>/healthz
```

Expected shape:

```json
{
  "service": "opaque-privacy-sentinel",
  "status": "ok",
  "opaqueStudio": "public",
  "graphNetwork": "configured",
  "mcp": "/mcp"
}
```

`graphNetwork` must be `configured`; `opaqueStudio` remains the operational
source of truth. If the sidecar is temporarily unavailable, Sentinel should
report that context as `UNKNOWN` while Arc privacy context remains usable.

The judge-facing endpoint is:

```text
https://<sentinel-domain>/mcp
```

## 4. Prove the MCP interface

Use any Streamable HTTP MCP client, or send an initialize request:

```bash
curl -sS https://<sentinel-domain>/mcp \
  -X POST \
  -H 'accept: application/json, text/event-stream' \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-11-25","capabilities":{},"clientInfo":{"name":"opaque-demo","version":"1.0"}}}'
```

Capture one screenshot or terminal output of `/healthz` and one MCP response
for the submission/demo.

## 5. Finish the submission README

In [`partner-docs/the-graph.md`](../partner-docs/the-graph.md), replace every
occurrence of:

```text
https://<sentinel-domain>
```

with the Railway public domain. Confirm that these links work:

```text
https://<sentinel-domain>/healthz
https://<sentinel-domain>/mcp
```

## 6. Optional Agent0 registration

This is a bonus, not a dependency for the Graph composition demonstration.

1. Replace `mcpEndpoint` in `graph/sentinel/agent0-registration.json` with the
   final `https://<sentinel-domain>/mcp` URL.
2. Register the descriptor through the intended ERC-8004 / Base Sepolia flow.
3. Add the resulting registration URL/transaction to the partner README only
   after it is confirmed on-chain.

## 7. Optional Substreams publication

The reusable EVM package is complete at
[`indexer/privacy-signals`](../indexer/privacy-signals). If Substreams CLI and
a target provider are available:

```bash
cd indexer/privacy-signals
substreams pack substreams.yaml
```

Publish only after supplying your own target-provider credentials. Add a
public package link to the README only after publication succeeds.

## Done criteria

- Railway deployment is green.
- `/healthz` reports `opaqueStudio: public`.
- `/mcp` completes MCP initialization.
- The public Railway domain replaces README placeholders.
- The Studio link and public Sentinel link both work from an unauthenticated
  browser/session.
