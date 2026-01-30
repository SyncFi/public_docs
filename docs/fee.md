# Fees & VIP

## TL;DR (Too Long; Didn't Read)

- SyncFi uses **Hyperliquid builder codes** to charge fees **per order** (within Hyperliquid limits)
- Users must authorize builder fees via on-chain **ApproveBuilderFee**
- Hyperliquid limits: **perps <= 0.1%**, **spot <= 1%**
- VIP fee rate is based on **30-day rolling trading volume**
- Wallet-specific fee handling details are in [Wallets & Fee Handling](wallets)

This page documents Hyperliquid builder codes, allowed fee ranges, and SyncFi VIP fee tiers.

## Hyperliquid builder codes

> Note: “builder” here refers to DeFi builders who build applications on Hyperliquid (such as SyncFi), not block builders in consensus.

Hyperliquid supports **builder codes**, which allow a builder (such as SyncFi) to receive a fee on trades it sends on behalf of a user:

- Builder codes are set **per order**.
- The user must first approve a maximum builder fee for each builder address via the on-chain **ApproveBuilderFee** action.
- Permissions can be updated or revoked by the user at any time.
- Builder codes are processed entirely on-chain as part of Hyperliquid’s fee logic.

### Fee ranges (Hyperliquid limits)

- **Perpetuals (perps):** builder fees can be at most **0.1%**
- **Spot:** builder fees can be at most **1%**

SyncFi’s fee settings always stay within these Hyperliquid constraints.

## Wallets and fee handling

Wallet type affects how builder fee authorization is handled on-chain. See [Wallets & Fee Handling](wallets) for the full explanation.

---

## SyncFi VIP tiers (30-day rolling trading volume)

Notes:
- Higher cumulative trading volume in the last 30 days unlocks **lower fees**
- Current default tier is **VIP1 (Beginner Pioneer)** with a fee rate of **0.20% (2‰)**

### VIP tiers and fee rates

| VIP Tier | Tier Name | 30-Day Cumulative Trading Volume (USDT) | Fee Rate | Discount vs VIP1 |
|---|---|---:|---:|---|
| VIP1 | Beginner Pioneer | 0 <= volume < 10,000 | 0.20% (2‰) | Baseline |
| VIP2 | Steady Follower | 10,000 <= volume < 50,000 | 0.18% (1.8‰) | ~10% |
| VIP3 | Elite Trader | 50,000 <= volume < 200,000 | 0.16% (1.6‰) | ~20% |
| VIP4 | Strategy Master | 200,000 <= volume < 1,000,000 | 0.14% (1.4‰) | ~30% |
| VIP5 | Investment Legend | volume >= 1,000,000 | 0.12% (1.2‰) | ~40% |

### Rule summary

- VIP level is evaluated using a **30-day rolling window** of cumulative trading volume
- Tier upgrades happen **automatically** once thresholds are met (no application required)
- The system **re-evaluates daily** based on the most recent 30-day volume
- Fees are calculated in real time using the **current VIP tier**


