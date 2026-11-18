# Gozan Launchpad — Ammora on GIWA Sepolia

A token **launchpad** on Ammora's Launch Curve (ALC), built for the Ammora Season 2
Builder Mission ("Ship an exchange or launchpad with the Ammora SDK").

**Live demo:** https://oxdfahmad.github.io/ammora-launchpad/ *(URL set by the deployer)*

## What it does

| Tab | What happens (all through the real `@ammora-protocol/sdk`) |
| --- | --- |
| **Explore** | Lists live launches by scanning `LaunchCreated` events from the launch factory, and lists the enabled Configs from the Config registry. Read-only, no wallet needed. |
| **Create launch** | Plans an **atomic Config + token + first buy** in one transaction, derived from the reviewed Ammora template Config #6. The first buy is quoted with the SDK's segmented-curve math (`@ammora-protocol/math`) so the minimum output is derived from the real curve. Shows the full plan (expected Config ID, predicted token/curve, gas, approval) before any signature. |
| **Trade** | Reads a live curve's state, quotes a buy (WETH → token) or sell (token → WETH) with the SDK, plans the swap with slippage protection, and sends it through the connected wallet. |
| **Graduate** | Shows a curve's graduation progress (quote raised vs threshold) and, once it is **Filled**, migrates it into its canonical AMM pool. |

## How the SDK is used

- `@ammora-protocol/sdk@0.3.9` (+ `@ammora-protocol/math@0.3.0`, `viem@2.33.0`).
- Deployment resolution: `giwaSepoliaAutoRouteDeployment` → `createAmmoraDeploymentSource` → `resolveAmmoraFinalizedHead` → `verifyAmmoraV4CoreAtBlock` (30 contracts verified live at build time).
- Read paths: `getAmmoraV4ConfigAtBlock`, `getAmmoraV4NextConfigId`, `AmmoraV4LaunchCurveClient` (`getStateAtBlock`, `quoteExactInputAtBlock`, `planMigrateAtBlock`), `scanAmmoraLogs` for `LaunchCreated`.
- Launch: `deriveConfigFromTemplate` (template #6, hash-bound) + `quoteALCSegmentedExactInput` (first-buy quote → minimum output) + `createConfigAndPoolWithFirstBuy` (APPROVAL_REQUIRED → exact-amount WETH approval → READY → one atomic transaction). Receipts are parsed with `parseAmmoraV4ConfigCreatedReceipt`, `parseAmmoraV4LaunchCreatedReceipt`, `parseAmmoraV4SwapReceipt`, `parseAmmoraV4MigratedReceipt`.
- Wallet: injected EVM wallet only (MetaMask-style). The app adds/switches to GIWA Sepolia (chain 91342). It never handles, requests or logs a private key or seed phrase.

## Run locally

```bash
npm install
npm run dev        # http://localhost:5173
```

## Build & deploy (static, GitHub Pages)

```bash
npm run build      # outputs dist/
```

Deploy the `dist/` folder to any static host. For GitHub Pages from a project repo, build with the repo path as base:

```bash
VITE_BASE=/ammora-launchpad/ npm run build
```

Then publish `dist/` (e.g. `gh-pages` branch or GitHub Actions).

## What was verified live (no funds used)

- Deployment verified: slot `v4-auto-route-20260917`, protocol V4, block 41617282, factory `0x35353a2Dab7426EcCAe6f5FB267702a542ab68fA`, registry `0x6d043014972b87406C4757a5f2d655A24e0b0aF9`.
- Template #6 enabled, hash `0x6048908774397e0691bb0e19f72dabb790eb0d892d6081d3622b39e7f738e667`.
- Live launches found (most recent curve `0x7Bec0BF8298C74E63e46Fc5B9c629F44b9F0CB22`, Config 52, Active) with a working on-chain quote (0.001 WETH → ~999.8 tokens).
- Atomic first-buy plan validated end-to-end (unsigned preview): `APPROVAL_REQUIRED` with the exact WETH approval to the first-buy executor, expected Config ID 53, est. gas 46,511. Fails closed for an unfunded wallet ("Quote balance is below the atomic first-buy amount").
- First-buy quote math validated against the SDK guide's example: 0.00000001 WETH → 333,310,888,152 raw tokens (guide's example minimum was 330,000,000,000).

## What still needs a funded wallet to test end-to-end

- The actual atomic launch transaction, a live curve swap, and a migration: all three paths are implemented and their plans were validated read-only, but signing/sending them requires a GIWA Sepolia wallet holding test ETH + WETH. The app is built so a user connects their own wallet and the flows run as designed; nothing was signed or submitted during development.

## Testnet only

GIWA Sepolia is a testnet. Nothing in this app is real money. The launchpad launches
9-decimal tokens with a 0.001 WETH graduation threshold by default (editable in the form).
