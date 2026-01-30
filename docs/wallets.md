# Wallets & Fee Handling

## TL;DR (Too Long; Didn't Read)

- SyncFi supports **2 wallet types**: generated (exportable key) and imported by API keys
- Users can **self-custody**, move funds anytime, and keep full control of wallets and positions
- Hyperliquid builder fees require **ApproveBuilderFee** authorization (on-chain)
- For API-keys wallets, approval must be signed by the **main wallet** (not an API sub-wallet)
- Without (or after revoking) approval, SyncFi may be unable to trade/copy trade on that wallet

This page explains wallet types supported by SyncFi and how fee handling (Hyperliquid builder fees) works for each.

## Wallet types

SyncFi supports two types of trading wallets:

- **Generated Wallets**: wallets created by SyncFi (you can export the private key and self-custody)
- **Imported Wallets (API Keys)**: connect your existing Hyperliquid account via API keys (no private key required)

Both wallet types can be used for strategies and copy trading. However, fee handling and builder codes differ because of Hyperliquid’s on-chain permission model.

---

## Generated Wallets

### What is a Generated Wallet?

A Generated Wallet is a Hyperliquid-compatible wallet that is:
- Created by SyncFi
- Programmatically controlled by SyncFi systems
- Used exclusively for trading via SyncFi strategy and copy-trading infrastructure

### How fees work with Generated Wallets

Because SyncFi controls the main wallet:

- **One-time on-chain builder approval**
  - SyncFi sends an **ApproveBuilderFee** transaction from the wallet’s main address.
  - This authorizes SyncFi’s builder address to charge a builder fee up to a predefined maximum rate (within Hyperliquid limits).

- **Automatic builder fees per order**
  - For each order SyncFi places, it attaches Hyperliquid builder parameters such as:

```json
{
  "b": "builderAddress",
  "f": "feeValue"
}
```

  - As long as `f <= maxFee` set in `ApproveBuilderFee`, Hyperliquid distributes the configured fee share to SyncFi’s builder address.

- **On-chain transparency**
  - All builder fees are processed by Hyperliquid on-chain.
  - Users can verify them via Hyperliquid info APIs and historical data.

### Advantages of Generated Wallets

- No manual approval: SyncFi handles ApproveBuilderFee during wallet setup.
- Full feature support: all strategies and advanced features are available.
- Stable fee model: fees are consistently applied according to SyncFi pricing.

---

## Imported Wallets (API Keys)

### What is an Imported Wallet (API Keys)?

An Imported Wallet (API Keys) is your existing Hyperliquid account that you connect to SyncFi by providing:
- Hyperliquid API key & secret
- Optional trading permissions configured on Hyperliquid

SyncFi does not hold your primary private keys. It only uses API credentials to place and manage orders on your behalf.

**Important recommendation**  
If you plan to import a Hyperliquid API wallet, SyncFi recommends using the same address to log in to SyncFi as you use on Hyperliquid. This helps keep positions, permissions, and billing logic consistent.

### Hyperliquid constraint: Main wallet vs API wallet

Hyperliquid’s builder system requires:
- **ApproveBuilderFee must be signed by your main wallet**
- It cannot be signed by an agent or API sub-wallet

As a result:
- Even with trading API access, SyncFi cannot set or change your builder fee authorization by itself.
- You must perform `ApproveBuilderFee` on-chain from your main Hyperliquid wallet.

### How fees work with Imported Wallets

To enable SyncFi to trade and charge builder fees on an Imported Wallet, there are two steps:

**Stage 1 — One-time on-chain approval (required)**  
You must:
- Use your main Hyperliquid wallet (the owner of the account).
- Send an `ApproveBuilderFee` transaction specifying:
  - `builder`: SyncFi’s builder address
  - `maxFee`: the maximum fee (in tenths of basis points) you authorize per order

**Note**  
When you perform this step through SyncFi, the platform will prompt your wallet for a signature before the import is completed. Make sure the account selected in your wallet is the same address as your main Hyperliquid account; otherwise, the approval will not apply to the correct Hyperliquid user and SyncFi will refuse the import request.

**Stage 2 — Per-order builder fee usage (automatic)**  
After `ApproveBuilderFee` is completed:
- For all orders sent via your Imported Wallet, SyncFi attaches builder parameters such as:

```json
{
  "b": "builderAddress",
  "f": "feePerOrder"
}
```

- As long as `f <= maxFee` that you approved:
  - Hyperliquid deducts the builder portion from your collateral/quote asset.
  - Hyperliquid routes this amount to SyncFi’s builder account.

### If you do not approve (or you revoke approval)

If you either:
- Do not perform `ApproveBuilderFee`, or
- Manually revoke or reduce SyncFi’s builder approval on-chain after importing,

then SyncFi cannot:
- Import your Hyperliquid API wallet
- Use your Hyperliquid API wallet to place or manage orders on Hyperliquid
- Execute copy trading or strategies for that Imported Wallet

**Important note**  
If you revoke `ApproveBuilderFee` or reduce the fee rate SyncFi relies on after setting up an Imported Wallet, SyncFi will no longer be able to place trades or follow strategies for you on Hyperliquid. To continue copy trading, you must:
- Manually restore `ApproveBuilderFee`, or
- Re-import the wallet with proper approval, or
- Use a Generated Wallet.

