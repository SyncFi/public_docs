# Deposit

This page specifies the deposit flow and address design for SyncFi.

**SyncFi** runs on **Arbitrum** and supports deposits in USDC, which are bridged into **Hyperliquid** via Hyperliquid’s official bridge contract.

## Supported asset and network

- **Network**: Arbitrum One
- **Asset**: USDC (standard [EIP‑20] fungible token on Arbitrum)
- **Unit**: USDC (6 decimals)
- **Contract Address (CA)**: `0xaf88d065e77c8cC2239327C5EDb3A432268e5831`

USDC is an [EIP‑20] token. Transaction fees on Arbitrum are always paid in ETH, not in USDC.

## Deposit flow

### 1) User initiates deposit

- User clicks “Wallet Deposit” on the SyncFi UI.
- SyncFi generates and displays a deposit address.

### 2) User sends USDC

- User sends USDC on Arbitrum to the provided deposit address.
- User does not need to fund this address with ETH.

### 3) On-chain monitoring and confirmation

Backend continuously listens for:
- USDC [EIP‑20] Transfer events to the deposit address.
- Balance changes on that address.

Once confirmation conditions are met (e.g., minimal confirmations), the deposit is marked as pending processing.

### 4) Gas distribution and fee deduction

- The deposit address initially holds only USDC and no ETH for gas.
- SyncFi uses a centralized **Gas Distributor Address** to:
  - Send a small amount of ETH to the deposit address to pay for gas.
  - Deduct approximately 1 USD worth (≈ 1 USDT worth) from the user’s deposit to cover gas and operational costs.

### 5) Bridge to Hyperliquid

- After the deposit address is funded with sufficient ETH:
  - SyncFi triggers a transaction from the deposit address to the Hyperliquid bridge contract, transferring the remaining USDC.
- Once the bridge transaction is confirmed:
  - SyncFi credits the user’s balance internally.
  - Funds are available on Hyperliquid for follow-trading.

---

## Deposit address design (security model)

### Risk of funding user-exportable addresses with gas

In SyncFi, the generated address that the user sees is designed to support private key export. This is a user self-custody wallet.  
If SyncFi were to:
- Directly send ETH (gas) to this generated address, and
- Rely on this address to automatically forward USDC to the Hyperliquid bridge,

then the following risks arise:
- An attacker (or the user) who has the private key and a faster Arbitrum node or higher gas price can:
  - Front-run SyncFi’s intended transaction.
  - Sweep both the USDC and the ETH (gas) from that address before the platform’s transaction confirms.

Because the generated address is under user control (private key exportable), it cannot be trusted as a platform-controlled relay address for automated bridging once it holds gas.

### USDC is [EIP‑20], addresses have no native gas

- USDC is an [EIP‑20] token; transfers are contract calls.
- All gas fees on Arbitrum are paid in ETH.
- An address receiving only USDC has no ETH and therefore cannot originate any transaction until ETH is provided.

To enable automated bridging, SyncFi must:
- Fund the address with ETH from a central Gas Distributor Address.
- Use that ETH to pay gas for the transaction to the Hyperliquid bridge contract.
- Deduct the gas cost (about 1 USDT worth) from the deposit.

If this address were a user-exportable generated address, then sending ETH to it would immediately expose that ETH and the USDC to user/attacker control, which is unacceptable for a platform-level automated flow.

Therefore, the deposit address must be isolated from the user-exportable generated address.

### Isolation model

#### Generated Address (user-exportable)

- Private key exportable by the user.
- Treated as a user self-custody wallet.
- SyncFi does not rely on this address for automated gas distribution or bridging.

#### Deposit Address (platform-controlled)

- Used exclusively for the SyncFi deposit pipeline.
- Private key is managed by SyncFi; not exportable by the user.
- Only SyncFi’s backend logic:
  - Receives USDC to this address.
  - Funds it with ETH from the Gas Distributor.
  - Initiates the transfer to the Hyperliquid bridge.

This separation:
- Prevents front-running and asset drain when gas is injected.
- Clearly separates user-controlled keys from platform-controlled operational keys.

---

## Relationship between Deposit Address and Generated Address (HD derivation)

Although they are separated in terms of control and permissions, deposit addresses and generated addresses are derived from the same high-entropy master seed using different derivation paths.

### HD derivation

- SyncFi uses a high-entropy master seed.
- Addresses are derived using hierarchical deterministic (HD) paths similar to [BIP‑32/BIP‑44].
- Different purposes use different derivation paths, for example:

Deposit Address path (example):

```text
[m / 44' / 60' / 0' / 0 / 1 / userIndex]
```

Generated Address path (example):

```text
[m / 44' / 60' / 0' / 0 / 2 / userIndex]
```

Here:
- `1` can represent “deposit address purpose”.
- `2` can represent “generated address (user-exportable) purpose”.
- `userIndex` ties the two addresses to the same logical user in the backend.

### Properties

- **Logical isolation**
  - Different paths → different keys and addresses.
  - Different roles:
    - Deposit Address: platform-controlled, no user export.
    - Generated Address: user-controlled, key export allowed.

- **Operational traceability**
  - SyncFi can deterministically derive both addresses for a user from the same seed plus userIndex.
  - Facilitates reconciliation, auditing, and risk controls.

- **Cryptographic security**
  - Standard HD derivation from a high-entropy master seed.
  - Compromise of one derived key does not reveal the seed or other derived keys.
  - Deposit and generated addresses remain cryptographically independent while being systematically related.


