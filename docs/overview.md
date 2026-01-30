# SyncFi: Universal Copy Trading Platform

## TL;DR (Too Long; Didn't Read)

- **Universal copy trading** across digital asset markets (starting with Hyperliquid)
- **Safer multi-leader copying** via virtual positions (avoid "close everything" mistakes)
- **Low latency execution** via on-chain P2P monitoring + event-driven, in-memory propagation
- **No subscriptions**: fees handled per order via Hyperliquid builder codes
- **Safer funds & full control** with multiple wallet options (export key / import key / API keys)
- **Smarter leader selection**: evaluates copyability so smaller accounts can actually follow

## Simple Flow, Smarter Follow

---

## Introduction

SyncFi is the world's first **universal copy trading ecosystem** that enables users to replicate successful strategies across all digital asset markets. We transform complex trading into accessible, intelligent investment opportunities through our "Simple Flow, Smarter Follow" philosophy.

### Multi-Market Coverage
- **Current**: Hyperliquid (crypto derivatives & spot trading)
- **Expanding**: Polymarket (prediction markets), Aster (DEX), Smart Money tracking
- **Vision**: Copy trade everything - cryptocurrencies, options, prediction markets, DeFi strategies

### Core Innovation
Moving beyond simple trade copying to **meaningful strategy replication** - helping users participate in value-driven investment approaches rather than random speculation.

---

## Why we built SyncFi (and why it's different)

After reviewing many copy-trading products, we found that many platforms interpret "copy trading" as simply copying trade signals, while overlooking a critical responsibility: **protecting followers from execution and position-management risks**.

### The problem with “naive” copy trading
On many platforms, when a leader closes a trade, the follower system may close the follower’s **entire real position** on that symbol. This is especially problematic when one follower account is following multiple leaders on the same symbol. Closing everything based on a single leader’s action is **irresponsible** and can lead to significant losses.

### Our approach: virtual positions + strategy-aware execution
SyncFi introduces **virtual positions** and an algorithmic position ledger:
- We track positions per **leader + symbol** internally, even though the exchange only exposes a single net position per symbol.
- When a leader closes, we close **only the portion** of the follower exposure attributed to that leader.

This makes multi-leader copy trading safer and more accurate. See [Position Model](position.md) for details.

### Reject high latency: built for fast signal propagation
Many copy-trading platforms suffer from slow pipelines. From receiving a leader signal to actually triggering execution can take **2–5 minutes**. With that kind of delay, market conditions may have already changed dramatically, and the copied “strategy” often becomes invalid.

SyncFi is designed with a full low-latency architecture:
- We upgrade signal intake from traditional API polling/webhooks to **on-chain P2P broadcast** monitoring
- Signals propagate through **high-speed in-memory channels** inside our system
- The entire flow is **event-driven**, so trade triggers are handled as system events rather than slow batch processes

In short: we optimize for fast, reliable propagation and **refuse high-latency copy trading**.

### Flexible Web3 pricing: no subscriptions
Many copy-trading systems charge users via closed private channels or monthly subscriptions. We believe Web3 products should be more flexible and transparent.

SyncFi uses **Hyperliquid builder codes** to collect a reasonable commission **automatically per order** (within Hyperliquid limits). This keeps the user experience simple:
- No subscription payments
- No recurring billing setup
- Fees are handled directly by Hyperliquid as part of the on-chain trade flow

See [Fees & VIP](fee.md) for details.

### Safer funds: wallet isolation and full user control
Compared to many platforms, SyncFi is designed to keep user funds safer and more flexible in day-to-day use (depositing, withdrawing, and position control).

We support multiple wallet options to isolate funds and match different user preferences, including:
- Wallets created by SyncFi with **private key export** (so users can self-custody)
- **Private key import** for users who want maximum control and portability
- **API key import** for users who prefer connecting your existing Hyperliquid account without sharing private keys

Users can move funds at any time and keep full control over their wallets and positions. See [Wallets & Fee Handling](wallets.md) for details.

### Optimized trade details to improve outcomes
Beyond accounting correctness, we continuously refine strategy parameters and execution details to:
- Improve the probability that followers can successfully open/close positions
- Reduce unnecessary losses caused by poor sizing, forced closures, or misaligned execution
- Help users capture more meaningful profit from copy trading rather than noise

---

## Key Advantages

### 1. Universal Asset Access(On going)
**The only platform where you can copy trade everything**
- Multi-asset coverage from crypto to prediction markets
- Cross-chain strategy execution
- Smart money and institutional movement tracking

### 2. Intelligent Strategy Curation
**Algo-powered selection of sustainable strategies**
- Risk-adjusted performance analytics
- Focus on long-term value creation
- Strategy transparency and education
- AI-Powerd selection of sustainable strategies will online soon

**Smarter leader selection for your account size**
Many platforms only rank leaders by metrics like PnL or ROI. In practice, many top whale accounts often trade with a relatively low position-to-equity ratio, meaning copied trade sizes for smaller followers can become too small to meet DEX minimum order size constraints.

SyncFi uses our own selection rules to evaluate **copyability** (including position-to-equity ratio and execution feasibility) and helps users find leaders that better match their available capital, so followers are less likely to miss opens due to sizing limitations.

### 3. Advanced Risk Management
**Safe and optimized execution**
- Dynamic position sizing based on risk tolerance
- Real-time portfolio monitoring
- Automated slippage protection

### 4. Value Investment Focus
**Meaningful trading, not gambling**
- Emphasis on sustainable wealth building
- Educational integration with strategy providers
- Community-driven learning approach

---

## 🔮 Vision

### Democratizing Sophisticated Trading
**Making institutional-grade strategies accessible to everyone**

We envision a future where:
- **Universal Access**: Every tradeable digital asset available for copy trading
- **Smart Money Intelligence**: Real-time tracking and replication of institutional movements
- **Educational Evolution**: Transforming followers into informed investors
- **Global Strategy Network**: Connecting the world's best traders with retail investors

### The Future Ecosystem
**"Copy Trade Everything, Everywhere"**
- Total market coverage across all digital assets
- AI-driven strategy optimization and prediction
- Cross-platform intelligence aggregation
- Sustainable value creation over speculation

