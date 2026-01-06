# SOL Top Holder

<img src="soltophold.png" alt="SOL Top Holder Logo" width="180" />


## Overview

SOL Top Holder is a Solana-native on-chain ranking and reward distribution system.
The protocol continuously observes token holder balances, deterministically ranks wallets in real time, and redistributes rewards to the top-ranked participants at a fixed interval.

The system does not rely on staking, locking, claiming, or user-initiated transactions.
Participation is implicit and purely balance-based.

At each distribution boundary, the protocol identifies the top N holders by live balance and executes a reward transfer to those wallets.
If a wallet is no longer ranked within the top set at the boundary, it does not receive that cycle’s rewards.

This design prioritizes sustained on-chain position over momentary balance spikes and is intentionally optimized for Solana’s high-throughput execution model.

---

## Core Design Principles

### Continuous Ranking

Traditional incentive systems rely on discrete snapshots or staking windows.
SOL Top Holder instead operates as a continuous ranking engine.
Wallet rankings are recalculated every distribution interval using live on-chain state.

This eliminates the need for registration periods, opt-in actions, or manual claims.

### Time-Weighted Participation

Rewards are not based on a single snapshot.
They are accumulated through repeated inclusion in the top ranking set over time.
The earlier and longer a wallet maintains a top position, the more reward cycles it participates in.

### Deterministic Execution

All ranking and distribution behavior is deterministic given on-chain state at the boundary.
There are no subjective inputs, governance overrides, or discretionary actions.

### Adversarial Tolerance

The system is designed for high-churn, adversarial environments.
Momentary balance manipulation does not guarantee sustained rewards.
Only balances present at the execution boundary are considered.

---

## High-Level System Architecture

The protocol is composed of on-chain and off-chain components.

On-chain:
- Solana program responsible for deterministic reward distribution
- Token Program integration for balance reads and transfers

Off-chain:
- Indexer that observes token accounts
- Ranking engine that computes holder order
- Execution coordinator that triggers distribution instructions

### Data Flow

1. Token accounts are scanned from the Solana ledger
2. Wallet balances are aggregated
3. Wallets are sorted by balance
4. Top N wallets are selected
5. Distribution instruction is executed

### Architecture Diagram

```
[ Token Mint ]
      |
      v
[ Token Accounts ]
      |
      v
[ Indexer ]
      |
      v
[ Ranking Engine ]
      |
      v
[ Solana Program ]
      |
      v
[ Reward Transfers ]
```

---

## Holder Ranking Mechanics

### Balance Collection

Balances are read directly from SPL token accounts associated with the mint.
Only settled on-chain balances are considered.

### Sorting

Wallets are sorted by balance in descending order.
The highest balance receives rank 1.

### Tie Resolution

If balances are equal, ordering is resolved deterministically using wallet public keys.
This guarantees consistent ranking across executions.

### Rank Transitions

Rank membership is evaluated independently at every interval.
A wallet entering the top set begins receiving rewards immediately.
A wallet exiting the set stops receiving rewards at the next boundary.

### Pseudocode

```
accounts = fetch_token_accounts(mint)
balances = aggregate_balances(accounts)

sorted_wallets = sort(
  balances,
  by = [balance DESC, pubkey ASC]
)

top_wallets = sorted_wallets[0:N]
```

---

## Distribution Engine

### Interval

Distributions occur at a fixed interval of 3 minutes.
This interval is chosen to balance responsiveness with execution stability.

### Reward Pool

Rewards are sourced from protocol-defined mechanisms.
The distribution logic itself is agnostic to the source of funds.

### Allocation

Rewards are distributed evenly across the top N wallets unless configured otherwise.

### Execution

Distribution is executed via a Solana program instruction.
All transfers occur atomically within the same transaction.

### Example Calculation

```
reward_pool = 1000 tokens
top_n = 10

reward_per_wallet = reward_pool / top_n
```

---

## Time and Fairness Model

The system rewards duration, not prediction.
Maintaining rank across multiple intervals compounds rewards naturally.

Early entry provides more distribution opportunities without requiring lockups.
This aligns incentives toward sustained holding rather than short-lived spikes.

---

## Edge Cases and Adversarial Behavior

### Flash Balance Movements

Temporary balance increases that do not persist until the boundary do not receive rewards.

### Wallet Cycling

Rapid wallet rotation does not bypass ranking logic.
Each wallet is evaluated independently per interval.

### RPC Inconsistencies

The system relies on finalized state.
Transient RPC discrepancies do not affect deterministic execution.

---

## Solana-Specific Considerations

### Account Model

Solana’s explicit account model allows efficient balance reads without contract storage iteration.

### Compute Limits

Ranking is performed off-chain to avoid compute saturation.
On-chain execution is limited to deterministic transfers.

### Execution Frequency

Solana’s throughput enables frequent reward cycles without congestion.

---

## Security Model

The protocol assumes:
- Correct execution of the Solana runtime
- Accurate token account state

The protocol does not assume:
- Trusted operators
- Admin intervention
- Off-chain custody

There are no upgrade hooks or discretionary controls.

---

## Protocol Properties

- Permissionless participation
- No custody
- No staking contracts
- No claim functions
- Continuous incentive alignment

---

## Example Execution Walkthrough

Assume 12 wallets hold the token.

At T0:
- Wallets A through J are top 10
- Rewards distributed

At T1:
- Wallet K enters top 10
- Wallet J exits

Wallet K begins receiving rewards immediately.
Wallet J stops at T1.

This process repeats every interval.

---

## Limitations and Tradeoffs

- Ranking size is fixed
- High-rank concentration is intentional
- Not designed for long-tail distribution
- Off-chain indexing is required

These tradeoffs are explicit and intentional.

---

## Future Extensions

- Configurable ranking sizes
- Weighted distributions
- Multi-token support
- Composable integrations
- DAO-controlled parameters

---

## Disclaimer

This protocol is experimental.
Behavior is deterministic but not guaranteed to be optimal under all market conditions.
Use at your own discretion.
