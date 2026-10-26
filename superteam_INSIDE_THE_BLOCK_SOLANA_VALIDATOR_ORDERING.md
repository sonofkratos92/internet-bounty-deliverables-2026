# Inside the Block: How Solana Validators Order Transactions
**Target Bounty**: Superteam Earn — *Inside the Block: How Solana Validators Order Transactions* ($2,500 USDC)
**Bounty URL**: https://earn.superteam.fun/listings/bounty/inside-the-block-how-solana-validators-order-transactions

---

## Executive Summary
Unlike Ethereum's 12-second slot-based mempool auction (PBS / MEV-Boost), Solana uses **continuous Leader-driven Gulf Stream ingress** with a 400ms slot duration (4 ticks per slot, 64 hashes per tick). Yet inside the validator's Transaction Processing Unit (**TPU**), transaction ordering is far from simple FIFO: it is governed by **QUIC stake-weighted QoS (SWQoS)**, **priority-fee-per-Requested-CU scheduling**, **Account Write-Lock Contention Graphs (`ThreadAwareAccountLocks`)**, and **Jito Block Engine / BAM bundle lanes**.

---

## 1. Stage-by-Stage Packet Journey Inside the TPU

```
[Client / RPC]
     │  QUIC Streams (Port 8009 / 8004) — Stake-Weighted QoS (SWQoS)
     ▼
[1. FetchStage (QUIC Streamer)]
     │  Coalesces up to 128-packet batches (PacketBatch)
     ▼
[2. SigVerifyStage]
     │  Ed25519 signature verification + packet deduplication filter
     ▼
[3. BankingStage (Scheduler + 4–6 Worker Threads)]
     │  ┌──────────────────────────────────────────────────────┐
     │  │ CentralScheduler / PrioGraph (Read/Write Lock DAG)   │
     │  │ Priority =TotalFee / (Requested_CU + Overhead_CU)    │
     │  └──────────────────────────────────────────────────────┘
     ▼
[4. SVM (rbpf Execution) + PoH Recorder]
     │  Executes non-conflicting batches in parallel across 4 banking threads
     ▼
[5. BroadcastStage (Turbine Shreds)]
```

---

## 2. How the Agave CentralScheduler Computes Priority

A common misconception is that paying a higher total priority fee guarantees earlier execution. In Agave's `BankingStage`, priority is calculated **per requested Compute Unit (CU)**, not per actual CU consumed:

$$\text{Priority Score} = \frac{\text{Prioritization Fee (micro-lamports)} + \text{Base Fee}}{\text{Requested Compute Unit Limit}}$$

### Critical Implication for Builders & Searchers:
- If a transaction does not include a `ComputeBudgetInstruction::SetComputeUnitLimit`, the scheduler assumes the default ceiling (**200,000 CU per instruction**, up to 1,400,000 CU).
- Over-requesting CUs dilutes your priority score by up to $10\times$ even if your transaction only consumes 20,000 CUs at runtime.

---

## 3. Account Lock Contention & The `PrioGraph` DAG

Solana transactions declare all read and write accounts upfront in the transaction header. Inside `CentralScheduler`:
1. Each incoming transaction is inserted into a **directed acyclic contention graph (`PrioGraph`)** keyed by account pubkey.
2. Two transactions conflict if both touch account $A$ and at least one requests a **write lock** on $A$.
3. If Transaction $T_1$ (high priority) and Transaction $T_2$ (medium priority) both write to the Raydium `AMM Authority` or Orca `Whirlpool` state account, $T_2$ cannot be scheduled onto Worker Thread 2 while $T_1$ holds the write lock on Worker Thread 1.
4. Meanwhile, Transaction $T_3$ (low priority) that writes only to an uncontended user ATA bypasses the hot-account queue and executes immediately on an idle worker thread.

### Block Limits That Gate Inclusion:
- **Max Block Compute Units**: `48,000,000 CU` per slot
- **Max Single-Account Write Compute Units**: `12,000,000 CU` per slot (25% of block capacity — preventing a single hot pool or mint from monopolizing all 4 worker threads)
- **Max Vote Compute Units**: Reserved lane so consensus votes are never starved by DeFi congestion.

---

## 4. Jito-Solana & BAM (Block Assembly Marketplace) Bundle Lanes

On validators running the Jito-Solana / Agave client (~90%+ of mainnet stake):
- **Standard TPU Lane (~80% of slot time)**: Processes QUIC packets via the `CentralScheduler` PrioGraph.
- **Bundle Auction Lane (Jito Block Engine)**: Executes atomic 1–5 transaction bundles sorted by `tip / CU` without revert risk, before or interleaved with standard TPU batches.
