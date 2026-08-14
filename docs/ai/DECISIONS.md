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

---

## ADR-008

Date

2026-08-09

Status

Accepted

Decision

Use `giwa-midnight/` as a dedicated Git submodule workspace for the local-only
Midnight PoC. The workspace is backed by
`https://github.com/leonid-world/giwa-midnight.git` and contains the Compact
contract, shared API boundary, CLI, mock Attestation API, and local Docker
configuration.

Reason

The PoC needs an independently versioned Midnight toolchain without coupling its
Node/TypeScript dependencies or generated artifacts to the Vue, Spring Boot, or
GIWA Solidity submodules.

---

## ADR-009

Date

2026-08-09

Status

Accepted

Decision

Use the official Midnight ZK Loan tutorial as the Phase 1 baseline and prove it
through its CLI before changing it to the GASOK domain.

Reason

It supplies the required Compact contract, Schnorr attestation flow, private
witness implementation, CLI orchestration, provider registration, and public
state inspection in one official end-to-end reference.

---

## ADR-010

Date

2026-08-09

Status

Accepted

Decision

Run this PoC only on the local Midnight `undeployed` network, using a local
Node, Indexer, Proof Server, and mock Attestation API. Do not deploy a Midnight
contract to Preprod or Mainnet.

Reason

The PoC is a reproducible local privacy demonstration, not a production
financial-verification system.

---

## ADR-011

Date

2026-08-09

Status

Accepted

Decision

Keep the existing Vue application, Spring Boot backend, and GIWA Solidity
contracts intact. Midnight adds only a financial-eligibility result; Vue
integration begins only after the official and GASOK CLI proof flows succeed.

Reason

GIWA remains responsible for tokenization, funding, repayment, wallets, and
asset movement. Keeping the proof system isolated prevents an experimental local
PoC from changing production funding architecture.

---

## ADR-012

Date

2026-08-09

Status

Accepted

Decision

Raw financial values, attestation signatures, witness secrets, mnemonics, and
provider private keys remain local private data. Public state and application
display may expose only eligibility output, provider ID, policy version, and a
pseudonymous commitment. The initial provider is explicitly mock-attested.

Reason

This preserves the PoC privacy boundary and avoids representing a local mock
signature as independent real-world financial verification.

---

## ADR-013

Date

2026-08-10

Status

Accepted

Decision

Phase 2 exposes only a pseudonymous verification commitment, binary funding
eligibility, Mock Provider ID, and policy version. Policy version 1 privately
checks annual revenue of at least 500,000,000 KRW, debt ratio of at most 200.00%,
and overdue count of at most 1. Do not publish raw values or add risk tiers,
maximum funding ratios, issued/expiry timestamps, a real-company binding, or a
GIWA funding gate until their policies and trust sources are separately approved.

Reason

Only the binary three-condition policy is currently defined. Additional public
categories would reveal more about private financial ranges and would invent
financial and time semantics that the PoC has not specified. CLI-only Phase 2
also has no approved binding between its local secret-derived commitment and a
GASOK business identity.

---

## ADR-014

Date

2026-08-12

Status

Accepted

Decision

Implement Phase 3A as a development-only, wallet-free public-result viewer in
the existing Vue application. A localhost-only `giwa-midnight/api` adapter uses
the official Indexer provider and generated Compact ledger decoder and returns
only the approved public result DTO. Keep the page independent of Funding,
Spring Boot, MySQL, MetaMask, and GASOK company/receivable identities. Defer
browser proof submission until a local `undeployed` signer architecture is
separately approved; do not use Preprod as a workaround.

Reason

The CLI flow already proves both eligibility outcomes, so public state can be
observed without any private input or wallet. The official ZK Loan browser path
depends on Lace and is Preprod-only because Lace cannot sign the local chain.
An honest read-only stage makes the runtime observable for learning without
inventing a browser proof flow or adding an unreviewed trusted signer service.
