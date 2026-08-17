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
provider private keys remain local private data. A Phase 2 eligibility-result
entry and its application display may expose only eligibility output, provider
ID, policy version, and a pseudonymous commitment. The contract's public admin
and registered Provider public-key registry remain public control-plane state.
The initial provider is explicitly mock-attested.

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

Each Phase 2 eligibility-result entry exposes only a pseudonymous verification
commitment, binary funding eligibility, Mock Provider ID, and policy version;
the public contract admin and Provider registry remain separate control-plane
state. Policy version 1 privately
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
used Lace/Preprod in the reference version evaluated at the time. An honest
read-only stage makes the runtime observable for learning without inventing a
browser proof flow or adding an unreviewed trusted signer service.
ADR-016 later replaces this anonymous list DTO with exact capability resolution;
the wallet-free and no-submission boundaries remain unchanged.

Runtime correction (2026-08-17): current official Midnight Local Dev and wallet
connector documentation supports Lace on the local `undeployed` environment.
The earlier reason must not be read as a continuing platform limitation.
ADR-018 supersedes only that factual premise and approves a local Bridge for
reuse of the current proven CLI identity/state, while leaving the historical
Phase 3A read-only decision intact.

---

## ADR-015

Date

2026-08-15

Status

Accepted

Decision

Add a CLI-first Phase 2.5 receivable-subject binding before any further Vue
proof-submission work. Construct the local Midnight contract with sealed GIWA
chain `91342` and ReceivableFinance
`0x0f264334f98BA0d22f7Fc6Bb901a5Fa36158a315`. For each proof, the Mock
Attestation Provider reads the specified receivable from that GIWA deployment
and resolves the canonical `SELLER` or `BUYER` wallet.

The provider signs an eight-field message containing the three private financial
values, company-commitment hash, GIWA receivable-subject binding hash, Midnight
deployment hash, provider ID, and policy version. Compact recomputes the context,
checks the signature and private policy, rejects an already-present exact lookup
key, and publishes only the opaque key with eligibility, provider ID, and policy
version. The CLI emits a versioned, correlation-sensitive capability so an
intended verifier can associate that opaque key with the selected GIWA
receivable party.

This is a role-context binding, not an ownership or provenance claim. The Mock
Provider labels caller-supplied demonstration inputs with the canonical GIWA
role wallet read from RPC; it does not yet prove that those values belong to the
role wallet. The security baseline therefore also requires canonical non-zero
Provider secrets, Compact rejection of the Jubjub identity public key, a single
approved Midnight deployment at the Provider and read adapter, bounded local
HTTP bodies/timeouts, owner-only CLI logs, and no mnemonic logging.

This decision does not add EIP-712 wallet-control authorization, legal-company
identity, bank verification, attestation freshness, latest-result or expiry
semantics, a Funding gate, browser submission, or Spring Boot coordination.
Those require separate decisions. Secure capability delivery/access, independent
Seller and Buyer actors/private states, and refresh-round policy are also
deferred.

Reason

The former secret-and-PIN-only result answered whether an anonymous local
witness met the policy, which was insufficient for a Funder evaluating the two
parties to a specific receivable. Domain separation across GIWA chain,
ReceivableFinance, uint256 receivable ID, role, canonical wallet, and Midnight
deployment prevents cross-receivable, cross-role, and cross-deployment signature
reuse while keeping those correlation fields out of the public result value.
The capability makes disclosure intentional instead of publishing every binding
directly on the Midnight ledger.

---

## ADR-016

Date

2026-08-15

Status

Accepted

Decision

Replace the anonymous Phase 3A result-list endpoint and Vue list with exact,
capability-based read resolution. The localhost-only adapter is the sole
Midnight contract-address authority. At acceptance, it pinned
`a8c0c1997c424dd1215d055fb5688200194263c7be5deef8b4e7620d2cdceb2c`.
Vue has no independent contract-address configuration. The capability carries
its deployment address, and the adapter approves or rejects it.

Runtime note (2026-08-17): the accepted address above is historical because the
disposable local Node was recreated and the deployment became `NOT_FOUND`. The
same single-authority design now pins replacement deployment
`7e3ea9d741ce0f5862db6f46d0ad720be2586cd7d0405ec77e4a0478aa50f4fb`;
this runtime replacement does not change the ADR.

Expose only `POST /v1/eligibility-results/resolve`. Its body must be the exact
version-1 proof-capability object with no extra fields. The adapter recomputes
the GIWA binding hash, Midnight deployment hash, and lookup key, checks the
pinned GIWA/contract context, and reads only that map entry. Do not expose the
former anonymous GET collection. Although the operation uses POST to keep the
correlation-sensitive capability out of URLs, it remains read-only and returns
`Cache-Control: no-store`.

The existing Vue application remains development-only for this feature. It
accepts a manually pasted capability, performs only safe basic syntax checks,
POSTs through the same-origin `/midnight-api` proxy, and displays the exact
receivable, Seller/Buyer role, canonical party wallet, eligibility, provider,
and policy version. It must not persist the capability in browser storage,
logs, or URLs and must not claim wallet ownership, legal-company identity, bank
verification, data truth, or Funding approval.

This decision does not implement secure capability delivery or verifier access
control, EIP-712 wallet-control authorization, independent Seller/Buyer actors,
freshness/expiry/refresh semantics, browser attestation/proof submission,
Spring Boot coordination, or a Funding gate.

Reason

Publishing every opaque result allowed observers to enumerate outcomes without
possessing the intentional disclosure context. Exact resolution preserves the
learning-oriented read path while requiring the verifier to present the
capability and while keeping authoritative hash recomputation outside Vue.
Making the adapter the only deployment-address authority avoids configuration
drift between the UI and resolver.

Validation distinguished the layers: live API smoke resolved receivable `#1`
as Seller `eligible=true` and Buyer `eligible=false`, and rejected tampering
with HTTP 400. Actual development-browser submissions resolved both capabilities
and displayed the same outcomes. This validates capability-based browser reads;
it is not browser attestation, proof generation, or transaction submission.

---

## ADR-017

Date

2026-08-15

Status

Accepted

Decision

Require an EIP-712 role-wallet authorization before the local Mock Attestation
Provider issues a new GASOK financial attestation. The Provider first resolves
the canonical Seller or Buyer wallet from the fixed GIWA ReceivableFinance
deployment, then issues a cryptographically random, two-minute, in-memory
challenge. Challenges are bounded, expire, and are removed atomically on the
first attestation attempt so EIP-712 signatures cannot be replayed.

The EIP-712 domain uses the user-readable GASOK mock-attestation name, schema
version, and GIWA chain ID. It does not set `verifyingContract`, because the
signature verifier is the off-chain Mock Provider rather than the
ReceivableFinance contract. The signed message explicitly includes the fixed
ReceivableFinance address, canonical role wallet, uint256 receivable ID, role,
approved Midnight deployment, Provider ID, financial-policy version, challenge
times, and a salted `attestationRequestCommitment`. That commitment binds the
three private mock financial inputs and company-commitment hash without sending
their raw values or the random salt to Vue.

Keep private financial input in the CLI. The CLI prints only the versioned
authorization request to its interactive terminal. A separate development-only
Vue `/midnight/authorize` tool validates the exact schema, selects the canonical
GIWA role account through MetaMask, signs it, verifies the recovered signer, and
returns a versioned authorization response for the user to paste back into the
CLI. Neither side stores the challenge, typed data, signature, salt, or raw
financial values in logs, URLs, browser storage, MySQL, or Midnight public
state.

Reserve Mock Provider ID `2` for this EIP-712-authorized issuance policy. Results
from Provider ID `1` are legacy role-context-only results and must not be
described as wallet-authorized. The Compact contract and eight-field Schnorr
message remain unchanged: a new result proves that registered Provider `2`
signed the private policy inputs after applying its off-chain authorization
policy, but Compact does not independently verify the secp256k1 signature.

This authorization proves control of the canonical GIWA role wallet at issuance
time. It does not prove legal-company identity, financial-data truth, bank or
accounting provenance, current eligibility, result freshness, revocation,
Funding approval, or secure capability delivery. The first implementation
supports MetaMask EOA signatures only; ERC-1271 contract-wallet verification is
deferred.

Reason

Phase 2.5 bound an attestation to the wallet address stored in a GIWA receivable
but did not require the holder of that wallet to approve the request. EIP-712
makes the role, receivable, deployment, Provider policy, and one-time purpose
visible in MetaMask while allowing the Provider to recover the signer before it
creates the Schnorr attestation. A hidden random salt keeps the browser-facing
request commitment from becoming a practical dictionary oracle for the small
private financial tuple. A distinct Provider ID preserves honest interpretation
of already-recorded legacy results without changing or migrating the local
Midnight ledger schema.

---

## ADR-018

Date

2026-08-17

Status

Accepted

Decision

Implement the first complete Vue-driven local proof flow with a trusted Node.js
Proof Bridge inside the `giwa-midnight/cli` workspace. Bind it only to
`127.0.0.1:4200`, expose it to development Vue only through the same-origin
`/midnight-proof` Vite proxy, and add a separate development-only
`/midnight/prove` route. Preserve the manual `/midnight` capability verifier and
`/midnight/authorize` CLI handoff as learning and diagnostic tools.

The Bridge reuses exactly the current local CLI participant: its encrypted
contract private state, Midnight development wallet and balance, approved
Provider 2 flow, Proof Server, and current deployed contract. It is therefore a
trusted custodial local process. MetaMask does not become a Midnight wallet; it
signs only the EIP-712 authorization proving control of the canonical GIWA
Seller/Buyer role wallet. The Bridge obtains the Mock Provider's Schnorr
attestation, supplies the plaintext witness to the local Proof Server, signs and
submits the Midnight transaction with its development wallet, and returns a
correlation-sensitive proof capability. The Proof Server never receives the
wallet key.

Vue may hold the raw financial tuple and PIN only in `/midnight/prove` component
memory until challenge creation. It sends them once to the loopback Bridge and
must immediately remove them from its reactive form state after the challenge
response. Neither Vue nor Bridge may log them, place them in Pinia, browser
storage, a URL, telemetry, Spring Boot, MySQL, or public Midnight state. Bridge
transient witness and Provider-signature fields are written only to encrypted
local private state for the proof call. Cleanup is marked before the write
awaits, attempted after success or failure, and retried once idempotently. A
later prepare sanitizes any stale transient fields and cannot proceed unless
that write succeeds. A finalized proof capability is preserved even if a
cleanup write reports failure. Because JavaScript strings cannot be reliably
zeroized, this is bounded-reference lifetime and sanitization rather than a
promise of perfect process-memory erasure.

Use memory-only CSPRNG session IDs in JSON bodies, never URL paths or queries.
Allow one active prepared/running session, one authorization use, and no
automatic retry after an ambiguous submission. The session states are
`awaiting_authorization`, `attesting`, `proving_and_submitting`, `indexing`,
`complete`, `failed`, `expired`, and `cancelled`. Vue polls status. A complete
session returns the exact proof capability but no trusted eligibility shortcut;
Vue must pass that capability to the independent port-4100 resolver and show the
Indexer-decoded result. Once the transaction finalizes, the Bridge preserves the
capability and immediately returns `complete` without a per-session Indexer
query. Vue retries only the resolver and never submits another proof for delayed
public visibility. A common fail-fast process lock prevents the interactive CLI
and Bridge from mutating the same encrypted LevelDB state concurrently.
An internal expiry timer consumes an unsigned `awaiting_authorization` session,
drops its prepared tuple, and releases the active slot at `expiresAt` without
requiring a status poll or another client request. Every terminal record,
including its capability or safe error, is retained for 60 seconds and then
purged by an unref timer without waiting for another request.

Before accepting HTTP input, the Bridge performs one 10-second-bounded Indexer
preflight, validates the pinned contract and Provider, and seals the GIWA
configuration in memory. Per-challenge preparation uses that cache, so raw
tuples are never held while an Indexer query is attempted. The SDK does not
offer cancellation: one timed-out startup query can remain internally pending,
but the server stays closed and no proof input has been accepted.

The HTTP surface is exact-body, JSON-only, no-CORS, loopback-only, body-size and
timeout bounded, and restricted to allowlisted local Origin/Host values,
`Sec-Fetch-Site: same-origin`, plus a custom UI request header. It returns
no-store, non-reflective safe errors and
does not expose request bodies, stacks, seeds, witness values, private state, or
Provider material. A proof already inside the non-abortable SDK call is not
reported as cancelled; clients continue polling its one status instead.

This Bridge is chosen because it minimizes change and preserves the identity and
state already proven through the required CLI-first sequence, not because Lace
is unable to use `undeployed`. The current official
[Local Dev README](https://github.com/midnightntwrk/midnight-local-dev/blob/8b44aabc5ea65e4c5d4cd855017517600bc90e8a/README.md#L76-L89),
[local proving guide](https://github.com/midnightntwrk/midnight-docs/blob/90da63c74fb92cf156505c682df07dbaba61be62/docs/guides/local-proving.mdx#L267-L325),
and
[wallet-connector `undeployed` example](https://github.com/midnightntwrk/midnight-docs/blob/90da63c74fb92cf156505c682df07dbaba61be62/docs/guides/react-wallet-connect.mdx#L180-L240)
show a supported local Lace path. Direct Vue + Lace is a viable later
self-custody replacement, but it requires a separate ADR for DApp Connector
integration and participant/private-state migration and must not be mixed with
the Bridge implicitly.

No part of this decision adds React, Spring Boot coordination, a GIWA Funding
gate, Preprod, Mainnet, financial-data truth, legal-company identity, bank or
accounting verification, freshness, revocation, or production security.

Reason

The CLI, Provider 2, Compact circuit, Proof Server, wallet, and current
deployment have already completed one real local end-to-end proof. Reusing that
known participant through a narrowly bound development Bridge lets the project
owner observe the complete browser-triggered sequence without changing the
existing Vue/Spring/GIWA architecture or introducing a second Midnight identity
at the same time. Explicit custody and session controls keep that educational
shortcut honest and bounded.
