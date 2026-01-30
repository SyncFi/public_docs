# Position Model (Virtual Positions)

## TL;DR (Too Long; Didn't Read)

- One account can follow **multiple leaders** on the same symbol, but the exchange shows only **one real net position**
- SyncFi uses **virtual positions** (per follower + leader + symbol) so closes are attributed correctly
- We “mark” the exposure per leader; on close we only close the **marked portion**
- Supports **partial close**, **full close**, and **reversal** without accidentally flattening unrelated exposure
- TP/SL uses **reduce-only**; virtual partial closes may require non-reduce-only opposite-side orders in multi-leader netting

## Why we need virtual positions

In our system, **one trading account can follow multiple leaders** and **trade multiple symbols**.  
However, the exchange exposes **only one real net position per account per symbol**.

To correctly attribute PnL and risk per leader, we maintain **virtual positions**:
- A virtual position is an internal ledger slice of the real position
- Each slice is keyed by: **follower account + leader + symbol**
- The sum of all virtual positions on a symbol equals the follower’s real net position on that symbol

## Core idea: “marking” the followed amount

When a follower follows a leader:
- We **record (mark)** how much exposure of that symbol belongs to that leader
- When the leader closes, we close **only the marked amount** for that leader’s virtual position

This is what makes “one account, multiple leaders, same symbol” workable.

---

## Virtual position close mechanics

### 1) Partial close (leader partially closes)

If a leader closes \(for example\) **50%** of their position:
- We compute the follower’s corresponding **virtual close ratio** (e.g. 50%)
- We apply the same ratio to the follower’s **virtual position** for that leader+symbol

**How we execute the real order**
- We place an **opposite-side order** (e.g. close long by selling, close short by buying)
- We typically **cannot rely on `reduceOnly`** for virtual closes in multi-leader scenarios

**Reason (important)**
- The follower’s **real net position** is the sum of multiple leaders’ exposures
- When we try to close *only one leader’s slice*, the real net position may be **smaller than the slice’s absolute size** (because other leaders may offset it)
- Using `reduceOnly` can cause the order to be rejected or partially filled in a way that makes virtual slices impossible to align
- Therefore, for partial virtual close we may need **opposite-side open** behavior to adjust the net position correctly

### 2) Full close (leader fully closes)

If a leader fully closes:
- We close **100%** of that leader’s virtual position
- Execution is done by placing an **opposite-side order** to flatten the corresponding exposure

### 3) Reversal (leader flips direction)

If a leader switches from a positive position to the opposite direction (e.g. long → short):
- Step A: **close** the follower’s existing virtual position for that leader+symbol (to zero)
- Step B: **open** a new virtual position in the new direction, based on the leader’s new target size

This ensures we don’t “mix” the old direction with the new one in the virtual ledger.

---

## Take-profit / Stop-loss orders (Reduce-Only)

For TP/SL on the exchange we use **reduce-only orders** to ensure the order **can only decrease** an existing real position and **never increases** exposure.

Common wording (English):
- **Reduce-only take profit order**
- **Reduce-only stop loss order**
- **Reduce-only limit order** (TP)
- **Reduce-only stop / stop-market order** (SL)

Notes:
- Reduce-only is preferred for TP/SL risk control at the **real position** level.
- For **virtual position** management (multi-leader netting), partial virtual closes may require **non-reduce-only opposite-side orders**, as described above.

