# Discover a Leader

## TL;DR (Too Long; Didn't Read)

- The Discover page provides two leader sources: **Hot Addresses** and **Hyperliquid Official Leaderboard**
- Hot Addresses are curated by SyncFi to improve **copyability** for different account sizes
- Hyperliquid leaderboard has strict eligibility rules; smaller accounts are often not a good match
- Both lists support filters like capital range, time window, ROI, PnL, and historical leverage

---

## Two leader lists on Discover

SyncFi provides two types of leader lists:

1) **Hot Addresses** (SyncFi curated)  
2) **Hyperliquid Official Leaderboard** (Hyperliquid curated)

---

## What’s the difference?

### 1) Hot Addresses (SyncFi curated)

Hot Addresses are selected by SyncFi using dedicated screening rules to find Hyperliquid addresses that are more likely to be valuable *and* practical to follow.

Key idea:
- Users can filter for leaders that better match their **capital size**, reducing the chance of “can’t open” outcomes caused by minimum order size constraints.

For how the Hot Addresses screening works, see [Hot Address Spec](hot-address-spec.md).

### 2) Hyperliquid Official Leaderboard

Hyperliquid provides an official leaderboard list. Their rule (quoted) is:

> Excludes accounts with less than 100k USDC account value and less than 10M USDC trading volume.  
> ROI = PNL / max(100, starting account value + maximum net deposits) for the time window.

Because the threshold is high, if a user’s account size is far below that level, we generally **do not recommend** using the official leaderboard as the primary source for copy trading.

---

## Filters supported (both lists)

Both leader sources support a set of filters so users can discover leaders based on their needs, such as:
- **Capital size / tier**
- **Time window**
- **ROI**
- **PnL**
- **Historical leverage**



