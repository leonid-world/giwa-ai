# Architecture Decisions

## ADR-001

Date

2026-07-29

Status

Accepted

Decision

Use MetaMask for all Seller, Buyer, and Funder application lifecycle write
transactions.

The test-only MockKRW owner may use local Hardhat administration commands for
explicit owner-balance distribution or additional demo-token issuance. The key
is supplied only through the current terminal and is never handled by the
frontend or backend.

Reason

Never store private keys.

---

## ADR-002

Decision

Wallet Address is mapped to Company.

Reason

Authentication remains Web2.

Blockchain only verifies ownership.

---

## ADR-003

Decision

NFT is initially owned by Financing Contract.

Reason

Remove Seller approve transaction.

---

## ADR-004

Decision

Single Funder model.

Reason

Keep MVP simple.

---

## ADR-005

Decision

Backend stores only transaction metadata.

Reason

Blockchain remains the source of truth.

---

## ADR-006

Date

2026-07-30

Status

Accepted

Decision

MockKRW uses 0 decimals, and one token base unit represents one integer KRW in the MVP.

Reason

The database stores receivable amounts as integer KRW with `DECIMAL(36, 0)`.
Using the same unit onchain prevents accidental 18-decimal scaling differences between
the database, UI, and smart contract.

---

## ADR-007

Date

2026-08-06

Status

Accepted

Decision

Use a separate, pre-funded MockKRWFaucet for testnet reviewer self-service.
Each wallet can claim one fixed amount through MetaMask. The Faucet transfers
existing MockKRW inventory and has no mint authority, backend signer, API, DB
state, or receivable lifecycle journal entry.

Reason

An online reviewer must be able to reach Funding without operator coordination,
while private keys and unlimited token issuance remain outside the public UI.
