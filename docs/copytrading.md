# Copytrading

## How SyncFi copy trading works

SyncFi triggers trades through **strategies**. To minimize propagation latency, we:
- Maintain our own **Hyperliquid blockchain nodes**
- Listen to **on-chain transaction events**
- Dispatch events through a **message queue** to trigger notifications and trade execution

This architecture reduces end-to-end delay between a leader’s on-chain activity and a follower’s execution.

---

## What is a strategy?

A **strategy** is the core configuration unit for copy trading. It defines the rules and parameters used to trigger and execute trades for a follower, including:
- **Copy rules** (what to copy and when)
- **Risk controls** (limits and safety checks)
- **Execution preferences** (how to open/close)

One follower account can create multiple strategies to follow different leaders or different setups.

---

## Strategy status (Enabled / Disabled)

### Enabled
When a strategy is **Enabled**, it is registered into the system. SyncFi will monitor the target (leader) address activity and execute copy trades according to the strategy settings.

In some cases, the strategy may be enabled but specific opens may fail (i.e. the system **cannot open** a copied position). The most common reason is that the computed order size does not satisfy the DEX minimum order size, including but not limited to:
- The leader’s equity is much larger than the follower’s equity, causing the **proportional sizing** result to be too small
- The **Multiplier** is set too small, causing the computed notional to fall below the minimum order threshold

Recommendation:
- If the follower’s capital is relatively small while the leader’s capital is very large, we recommend using **Fixed Amount** (equal-amount copy). This may not replicate the leader’s strategy precisely, but it helps keep the **long/short direction** consistent and reduces the chance of repeatedly failing to open positions.

### Disabled
When a strategy is switched from **Enabled** to **Disabled**, the system will:
- Immediately close the strategy’s associated **virtual positions** at **market price**
- Stop monitoring the leader address behavior for this strategy

> Important: Enabling/disabling a strategy can trigger executions (including forced market closes) and may result in losses. Please operate with caution.

## Strategy parameters

### Copy Trading Settings

#### Position Mode
Defines how the follower’s position size is derived from the leader:
- **Proportional sizing (equity ratio)**: open size follows the leader’s position ratio. For example, if the leader has 10,000 USDT and opens a 1,000 USDT position (10%), then a follower with 1,000 USDT will open a 100 USDT position (10%).
- **Fixed Amount**: size is a fixed notional/amount per trade

#### Multiplier
A numeric coefficient used when opening copy trades.

- **Precision**: 0.1  
- **Minimum**: 0.1  
- **Default**: 1.0

**Mechanism**

Multiplier scales the follower’s copied opening size relative to the leader’s position ratio.

Example:
- Leader equity: 10,000 USDT
- Leader opens a position with notional: 5,000 USDT
- Follower equity: 1,000 USDT

Then:
- With **Multiplier = 1.0**, follower opens: 500 USDT
- With **Multiplier = 0.1**, follower opens: 50 USDT

Formula (proportional sizing):

Follower Notional = Follower Equity * (Leader Position Notional / Leader Equity) * Multiplier

> Important: If Multiplier is too small, the calculated order size may fall below the DEX minimum order size, and the position may fail to open.

#### Opening Mode
Controls how positions are copied when the strategy is created.

- **Copy All** (copy historical positions)  
  When creating the strategy, copy all of the leader’s existing positions to the follower. New positions opened by the leader will then be followed automatically.

- **New Trades Only**  
  When creating the strategy, do not copy the leader’s existing positions and only follow new trades going forward.

- **Better Price** (preferential execution for historical sync)  
  During copy trading, the system will attempt to replicate the leader’s historical positions at a better price than the leader’s fill price, while new trades from the leader are followed immediately at the normal execution price.

#### Max Leverage
The maximum allowed leverage for the **virtual position** created by this strategy.

- **Precision**: 0.1  
- **Minimum**: 0.1  
- **Default**: 1.0  
- **Maximum supported value**: 40x

If the system estimates that the next order required to follow the leader would cause the strategy’s virtual position leverage to exceed **Max Leverage**, that copy trade will be **canceled**.

> Note: Virtual positions are used to support “one account, multiple leaders, same symbol”. See `position.md` for details.

**How Max Leverage behaves with multiple leaders/strategies**

Max Leverage is enforced at the **strategy virtual position** level, but the follower’s **account-level** leverage on a symbol is effectively the **sum of exposures** across all active strategies (additive).

Example:
- One follower address follows 3 leaders on the same symbol
- Each strategy has **Max Leverage = 1x**
- If all three leaders open positions, the follower’s account-level effective leverage can reach approximately **3x** (1x + 1x + 1x)

#### Take Profit (Optional, account-level)
An optional take profit threshold for the follower’s **real position** on the copy-traded symbol(s).

- **Default**: disabled (-1)
- **Precision**: 0.0 (one decimal place)
- **Minimum**: 0.3% (3‰)

Why >= 0.3% is required:
- SyncFi base fee is **0.20% (2‰)**
- DEX fees (commonly around **0.05%**) plus slippage further reduce realized profit
- If the take profit is below this level, the trade may become economically meaningless and the user may not profit

#### Stop Loss (Optional, account-level)
An optional stop loss threshold for the follower’s **real position** on the copy-traded symbol(s).

- **Default**: disabled (-1)
- **Precision**: 0.001
- **Minimum**: > 0
- **Maximum**: none (based on user risk preference)

Implementation notes:
- For both **take profit** and **stop loss**, the system creates/updates **reduce-only** TP/SL orders on the exchange
- If multiple strategies set TP/SL parameters, the system uses the **most recent** configuration to place/update the account-level TP/SL orders



