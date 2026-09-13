<div align="center">

# Opaque × Arc

### Programmable private USDC settlement, built natively on Arc.

[![Network](https://img.shields.io/badge/network-Arc%20testnet-FF7A18?style=flat-square)](https://arc.network/)
[![Asset](https://img.shields.io/badge/asset-USDC-2775CA?style=flat-square)](https://www.circle.com/usdc)
[![Account](https://img.shields.io/badge/account-ERC--4337%20%2B%20FORS%2BC-FF7A18?style=flat-square)](../packages/pq-wallet/)
[![Bridge](https://img.shields.io/badge/bridge-CCTP%20V2-2775CA?style=flat-square)](https://www.circle.com/cctp)

**Arc is Opaque’s settlement layer: the place where private notes become a real USDC payment under enforceable conditions.**

</div>

![Opaque confidential settlement on Arc](../diagrams/cre-arc.svg)

> **Track:** Best DeFi/Onchain Finance Application, Arc

## Why Opaque is native to Arc

Opaque is not a wallet deployed onto Arc. It is a private USDC settlement flow designed around what Arc makes possible:

| Arc primitive | What Opaque does with it | Why it is load-bearing |
|---|---|---|
| **USDC as money and gas** | The same asset funds the account, creates notes, pays for UserOperations, and reaches the recipient. | A post-quantum account needs no ECDSA wallet, native token, or separate gas balance. |
| **Programmable settlement** | A pool releases fixed-denomination USDC only after proof, nullifier, recipient, and confidential-policy checks pass. | Privacy is an enforceable payment condition, not an off-chain promise. |
| **ERC-4337** | FORS+C authorizes a smart account through PQKeyRegistry and PQValidator. | The account is controlled by a hash-based key rather than a conventional wallet signer. |
| **CCTP V2** | USDC moves from Sepolia to the same Arc account that creates private notes. | A user can enter the private payment rail with native Circle liquidity. |

## One conditional private USDC payment

~~~text
1. Fund a PQ ERC-4337 account with USDC on Arc or through CCTP V2.
2. Deposit USDC into fixed-denomination pools; each deposit creates a private note.
3. Locally form an 8-member same-denomination ring and seal the payment terms.
4. Chainlink CRE checks the sealed privacy threshold, deadline, and credential.
5. The attester verifies the proof and provides a post-quantum authorization.
6. Arc enforces real ring members, one unused nullifier, denomination, and recipient.
7. The pool transfers USDC to the recipient.
~~~

The deposit is attributable by design. The later private spend does **not** name which deposited note paid.

## The programmable-money mechanics

| Mechanic | Arc-enforced or Arc-observed behaviour |
|---|---|
| **Fixed denominations** | A 20 USDC note only hides among other 20 USDC notes. Amount privacy comes from interchangeable buckets, not an opaque balance. |
| **One-time spending** | The pool marks a fresh nullifier before moving USDC, preventing a note from being released twice. |
| **Conditional release** | CRE may release as soon as public privacy conditions meet the payer’s sealed threshold; otherwise it settles at the payer’s latest-settlement deadline. |
| **Recipient binding** | The final recipient must match the released payment terms before settlement proceeds. |
| **PQ key lifecycle** | The registry tracks FORS+C use and accepts rotation only under the current post-quantum authority. |
| **Relay health telemetry** | RelayDirectory emits health and identity events that The Graph turns into privacy coordination signals. |

## Settlement contracts

All contracts are live on Arc testnet. The deployment JSON is the complete source of truth.

| Contract | Address | Settlement role |
|---|---|---|
| **PQKeyRegistry** | [0x6eb5…8e8f](https://testnet.arcscan.app/address/0x6eb5b42373191121d31dfc4b5c8571c4eaf58e8f) | PQ key registration, use budget, and rotation |
| **PQAccountFactory** | [0x13be…b214](https://testnet.arcscan.app/address/0x13beaec42922e3f63fa0dbe5bba270edf46ab214) | Counterfactual ERC-4337 account deployment |
| **PQAccount implementation** | [0xecce…5012](https://testnet.arcscan.app/address/0xeccec6b1e6a2e5367902675c49e577633f705012) | UserOperation execution under PQ authority |
| **PQValidator** | [0xfad5…6ea2](https://testnet.arcscan.app/address/0xfad5b4149489eaf9bbe402eca4b26f9284046ea2) | FORS+C validation for EntryPoint v0.7 |
| **RelayDirectory** | [0xcf58…6653](https://testnet.arcscan.app/address/0xcf588b5b8ab2fa11ccf28a5c0631da4269a36653) | Relay announcements and health reporting |
| **EntryPoint v0.7** | 0x0000000071727de22e5e9d8baf0edac6f37da032 | ERC-4337 execution entrypoint |

## CCTP V2 entry path

Opaque uses CCTP V2 for USDC onboarding from Ethereum Sepolia.

~~~text
approve + depositForBurn on Sepolia, domain 0 → Arc domain 26
  → maxFee set to twice Circle’s current quote
  → poll Circle attestation until final
  → receiveMessage on Arc
  → USDC reaches the same account that deposits private notes
~~~

| Contract | Address |
|---|---|
| TokenMessengerV2, Sepolia | 0x8fe6b999dc680ccfdd5bf7eb0974218be2542daa |
| MessageTransmitterV2, Arc | 0xe737e5cebeeba77efe34d4aa090756590b1ce275 |
| USDC, Sepolia | 0x1c7d4b196cb0c7b01d743fbc6116a902379c7238 |

## Engineering evidence

| What we learned | Response in Opaque |
|---|---|
| Native USDC and ERC-20 USDC use different decimals. | All amount handling carries explicit units; native gas uses 18 decimals while pooled ERC-20 USDC uses 6. |
| Public RPC log windows and bursts are bounded. | Pool scanning pages logs in 10,000-block windows and falls back across Circle, QuickNode, and Blockdaemon RPCs. |
| A public RPC is not a bundler. | UserOperations use Pimlico’s keyless Arc endpoint for EntryPoint v0.7. |
| Small operations must stay economically viable. | First registry/verifier/pool deployment dry-run cost about 0.17 USDC; a relay health report costs about 0.0018 USDC. |

## Scope choices

- **CCTP V2, not Gateway:** CCTP provides the working Sepolia-to-Arc entry route. Gateway remains a future unified-liquidity integration.
- **Opaque PQ account, not Circle Wallets:** Circle Wallets use conventional elliptic-curve signers; Opaque needs the account authority itself to be post-quantum.
- **One stablecoin, not FX:** A single USDC asset and fixed buckets protect amount anonymity. StableFX and App Kits do not improve this V1 flow.

## Source evidence

| File | What it proves |
|---|---|
| [PrivatePool.sol](../contracts/src/opaque/pool/PrivatePool.sol) | USDC note custody and fixed-denomination pool rules |
| [AttestedRingVerifier.sol](../contracts/src/opaque/pool/AttestedRingVerifier.sol) | Post-quantum attestation gate for an eligible ring spend |
| [PQKeyRegistry.sol](../contracts/src/opaque/wallet/PQKeyRegistry.sol) | PQ key lifecycle and bounded authorization |
| [PQAccount.sol](../contracts/src/opaque/wallet/PQAccount.sol) | ERC-4337 smart account under PQ authority |
| [pq-wallet-chain.ts](../backend/chain/pq-wallet-chain.ts) | Account deployment and UserOperations on Arc |
| [pool.ts](../backend/chain/pool.ts) | Pool interaction and RPC fallback |
| [bridge-sepolia.ts](../backend/chain/bridge-sepolia.ts) | CCTP V2 bridging flow |

## Product feedback for Arc + Circle

**What worked well**

- Arc testnet already exposes the ERC-4337 EntryPoint, so Opaque needed no account-abstraction infrastructure deployment
- USDC gas removes an onboarding asset: the account is funded and pays with the same asset it sends.
- CCTP V2 was smooth and helpful


**What would make this easier**

- Publish public RPC log-range and burst limits
- Document a recommended ERC-4337 UserOperation bundler for Arc testnet alongside the public RPC

## Next on Arc

1. Deploy the same private-settlement rail to Arc mainnet after hardening the backend for production
2. Add Gateway funding, so a note can be created from a USDC balance on any supported chain.
3. Let recipients use Gateway to receive USDC on their preferred chain after private Arc settlement.

> **Why Arc:** private payments need a stable unit, cheap programmable settlement, and gas an account can pay without a second token. USDC on Arc supplies all three.
