# Withdraw

## TL;DR (Too Long; Didn't Read)

- After security verification, withdraw **any amount** to **any destination address**
- Withdrawals are sent **directly from Hyperliquid** to your destination address
- To withdraw all while positions are open, **disable all related strategies**; the system closes positions at market first
- Imported wallets connected via **API keys (API wallets)** do **not** support withdrawals (Hyperliquid limitation)
- Withdrawal fee is **$1 per transaction**

## Overview

After passing required security verification, you can withdraw **any amount** to **any destination address** you choose.

The withdrawal transaction is executed **directly from Hyperliquid** to your destination address.

---

## Withdraw all assets (when you still have open positions)

If you want to withdraw **all** assets while you still have open positions:

- Set all copy trading strategies associated with your **generated wallet** to **Disabled**
- The system will immediately close the associated positions **at market price**
- Once positions are closed, the remaining assets can be withdrawn right away

> Note: Imported wallets connected via API keys (API wallets) do not support withdrawals due to Hyperliquid’s mechanism/limitation.

---

## Fees

- Withdrawal fee: **$1 per transaction**


