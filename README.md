
# sBTC-HashPort - Cross-Chain Atomic Swap Smart Contract

This Clarity smart contract enables **secure, decentralized atomic swaps** of fungible tokens between different blockchain networks (e.g., Stacks ↔ Ethereum). It uses **hash time-locked contracts (HTLCs)** to ensure that swaps are trustless, permissionless, and either fully completed or fully refunded.

---

## 📜 Features

* 🔒 **Hash-lock Mechanism**: Swaps are locked using SHA-256 hashes.
* ⏳ **Time-locks**: Swaps automatically expire and become refundable after a set number of blocks.
* 🧾 **Token-agnostic**: Supports any fungible token contract that follows the `ft-trait`.
* 🧍 **Two-step participation**: The swap initiator starts the swap, and a participant claims it by revealing a secret.
* 🔁 **Refund support**: Initiators can reclaim their tokens if the participant fails to redeem before expiration.
* 🔗 **Cross-chain validation**: Validates target blockchain and wallet format for added security.

---

## 🧩 How It Works

1. **Initialization**

   * The initiator locks tokens in the contract.
   * A hash-lock, duration, and destination blockchain address are provided.
   * Tokens are transferred from the sender to the contract.

2. **Participation**

   * The counterparty registers as a participant to the swap.
   * The hash-lock must match, but the preimage is not revealed yet.

3. **Redemption**

   * The participant redeems tokens by revealing the preimage of the hash.
   * The contract verifies the preimage, and transfers tokens to the participant.

4. **Refund**

   * If the participant fails to redeem in time, the initiator can claim a refund.

---

## 📦 Data Structures

### Atomic Swap Record (`atomic-swaps`)

```clojure
{
  atomic-swap-identifier: (buff 32),
  swap-initiator: principal,
  swap-participant: (optional principal),
  token-contract-principal: principal,
  token-amount: uint,
  atomic-hash-lock: (buff 32),
  swap-expiration-height: uint,
  swap-current-status: "active" | "participated" | "redeemed" | "refunded",
  destination-blockchain: (string-ascii 8),
  destination-wallet-address: (string-ascii 42)
}
```

---

## 🚀 Public Functions

### `initialize-atomic-swap`

Initializes a new atomic swap.

```clojure
(token-contract <ft-trait>)
(token-amount uint)
(atomic-hash-lock (buff 32))
(swap-duration-blocks uint)
(destination-blockchain (string-ascii 8))
(destination-wallet-address (string-ascii 42))
```

---

### `register-swap-participant`

Registers the caller as the participant of a specific swap.

```clojure
(atomic-swap-identifier (buff 32))
```

---

### `redeem-atomic-swap`

Redeems a swap by revealing the correct preimage for the hash-lock.

```clojure
(atomic-swap-identifier (buff 32))
(hash-preimage (buff 32))
(token-contract <ft-trait>)
```

---

### `process-swap-refund`

Allows the initiator to reclaim tokens after swap expiration.

```clojure
(atomic-swap-identifier (buff 32))
(token-contract <ft-trait>)
```

---

## 👀 Read-Only Functions

### `get-atomic-swap-details`

Fetches all details about a swap using its identifier.

---

### `verify-hash-preimage`

Checks whether a given preimage matches a stored SHA-256 hash.

---

## 🧪 Validation

The contract performs thorough validation checks, including:

* Token contract interface compliance
* Non-zero and within-bounds token amount
* Proper hash-lock format and uniqueness
* Blockchain name and wallet address format
* Valid expiration block height

---

## ❗ Error Codes

| Error Code                         | Meaning                            |
| ---------------------------------- | ---------------------------------- |
| `ERROR-SWAP-EXPIRED`               | Swap has already expired           |
| `ERROR-SWAP-NOT-FOUND`             | Swap ID does not exist             |
| `ERROR-UNAUTHORIZED-ACCESS`        | Caller is not authorized           |
| `ERROR-SWAP-ALREADY-FINALIZED`     | Already redeemed or refunded       |
| `ERROR-INVALID-TOKEN-AMOUNT`       | Invalid token amount specified     |
| `ERROR-INSUFFICIENT-TOKEN-BALANCE` | Insufficient balance               |
| `ERROR-INVALID-CONTRACT`           | Provided token contract is invalid |
| `ERROR-INVALID-HASH`               | Hash-lock is malformed             |
| `ERROR-INVALID-BLOCKCHAIN`         | Blockchain name is not supported   |
| `ERROR-INVALID-ADDRESS`            | Invalid destination wallet address |

---
