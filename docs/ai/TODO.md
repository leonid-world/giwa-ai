# TODO

## Done

[x] PoC

[x] Architecture

[x] Smart Contract Design

[x] Login

[x] Wallet Mapping

[x] Receivable CRUD

---

## Stabilization Done

[x] Business Number CHAR(10) Alignment

[x] Business Number UI Formatting

[x] Common Backend Error Response

[x] Spring Security 401/403 Error Separation

[x] Wallet Duplicate Mapping 409 Conflict

[x] Common Frontend API Client

[x] MetaMask Account Selection and Confirmation UX

[x] Buyer Receivable Review and Explicit Attestation UX

[x] Buyer Pre-sign DB/Onchain CREATED Data Comparison

[x] MetaMask Multi-account Registered Signer Selection

[x] Role-specific VERIFIED Lifecycle Copy

[x] Shared Authenticated Layout Current-Account Email Display

[x] Shared Authenticated Navigation and My Information Page

[x] Frontend Visual Polish and Interaction Consistency Pass

[x] Frontend B2B SaaS Visual System and Vertical Blockchain Workflow Timelines

[x] Frontend Public Demo Release Readiness (metadata, favicon, social card, loading/empty states, 404, footer)

[x] Receivable Contract and Lifecycle Transaction Explorer Links

---

## Doing

[x] Buyer Verify Live GIWA Verification

[x] ReceivableFinance lifecycle contract implementation

[x] Seller createReceivable frontend transaction

[x] POST /receivables/{id}/chain-created synchronization

[x] Buyer verifyReceivable frontend transaction

[x] POST /receivables/{id}/verified synchronization and status history

[x] Seller tokenizeReceivable frontend transaction and NFT mint CTA

[x] POST /receivables/{id}/tokenized synchronization and status history

[x] RPC-authoritative token ID synchronization

[x] Cross-reload MetaMask replacement discovery and recovery

[x] Onchain-success/backend-failure retry UX

[x] Server-journal tokenization recovery gate and backend-only manual synchronization UX

[x] Third-party Funding opportunity discovery and role-safe visibility

[x] Funder mKRW balance/allowance preflight and explicit approval transaction

[x] Funder fundReceivable transaction, receipt recovery, and backend-only retry UX

[x] POST /receivables/{id}/funded synchronization and status history

[x] RPC-authoritative ReceivableFunded, mKRW Transfer, and NFT Transfer verification

[x] Buyer Repayment opportunity discovery and FUNDED receivable selection

[x] Buyer mKRW balance/allowance preflight and explicit face-value approval transaction

[x] Buyer repayReceivable transaction, receipt recovery, and backend-only retry UX

[x] POST /receivables/{id}/repaid synchronization and status history

[x] RPC-authoritative ReceivableRepaid and mKRW Transfer verification

[x] Local contract compile and Hardhat lifecycle/rollback tests

[x] Backend tests and build

[x] Frontend build/lint

[x] Reproducible Hardhat GIWA Sepolia compile, deployment, and Blockscout verification tooling

[x] Recover the successful ReceivableFinance deployment after immediate RPC visibility lag without redeploying

[x] Hardhat MockKRW Owner Balance Transfer and Additional Test Mint Operations

[x] MockKRW Owner Hardhat Operations User Guide

[x] MockKRW Owner Operations Confirmed-block RPC State Retry and Safe Warning

[x] Execute a real Owner-to-Funder 10,000 mKRW Hardhat transfer on GIWA Sepolia

[x] Document MetaMask MockKRW Import, Current-contract Selection, and Balance Refresh

[x] Document GIWA Sepolia MetaMask Custom Network Fields in the MockKRW Operations Guide

[x] Pre-funded MockKRWFaucet contract PoC (fixed one-time wallet claim, depletion protection, owner recovery)

[x] Safe Faucet-only Hardhat deployment command with separate metadata and duplicate-deployment recovery

[ ] Apply `.codex/migrations/20260730_receivable_chain_metadata_uniques.sql` to the existing MySQL database after backup and duplicate preflight

[ ] Confirm `blockchain_transactions` exists in local/Railway MySQL; run `.codex/migrations/20260730_blockchain_transactions.sql` only when absent

[ ] On an existing journal table, apply `.codex/migrations/20260730_blockchain_transaction_rpc_verification.sql` only when all four RPC proof columns are absent

[ ] Apply `.codex/migrations/20260730_blockchain_transaction_verification_version.sql` only when `verification_version` is absent; do not run either ALTER after the current create-table migration

[x] Deploy MockKRW and ReceivableFinance to GIWA Sepolia

[x] Configure frontend chain, explorer, RPC, and contract addresses

[x] Execute real Seller createReceivable and Buyer verifyReceivable transactions

[x] Deploy frontend to Vercel

[x] Add Railway Java 17 Docker build and runtime

[x] Support Railway PORT, MySQL variables, Vercel CORS, and GET /health

[x] Deploy backend to Railway and verify GET /health

[x] Set Vercel VITE_API_URL to the Railway public domain

[x] Normalize VITE_API_URL trailing slashes before appending API paths

[ ] Configure Railway MySQL references, JWT secret, and Vercel CORS origin

[ ] Initialize the fresh Railway MySQL schema

[x] Redeploy Vercel with the normalized API URL and verify frontend API calls

[ ] Redeploy Vercel with the public-demo metadata, SPA rewrite, and release-quality UI states

[ ] Set Railway GIWA_MOCK_KRW_ADDRESS and deploy the Funding backend

[ ] Redeploy Vercel with the Funding page

[ ] Deploy the Repayment backend to Railway

[ ] Redeploy Vercel with the Repayment page

---

## Next

[x] Deploy MockKRWFaucet against the existing MockKRW without redeploying MockKRW or ReceivableFinance (source verification optional)

[x] Record the MockKRWFaucet address, receipt, block, compiler settings, MockKRW link, and fixed claim amount in separate deployment metadata

[x] Pre-fund MockKRWFaucet with 600,000,000 existing owner mKRW inventory (60 fixed claims)

[x] Add the conditional "데모 mKRW 충전하기" flow to the Funding page and refresh readiness after a verified claim

[x] Share wallet-level Faucet claim recovery between Funding and Repayment without duplicating pending transaction safety logic

[x] Add the conditional Buyer "데모 mKRW 충전하기" flow to the Repayment page using faceValue readiness

[x] Add GIWA Sepolia native ETH balance guidance and the official test-ETH faucet link for claim/approve/fund/repay gas

[ ] Execute a real Buyer Faucet claim and confirm claim -> approval -> repayment remains three explicit actions

[ ] Set Vercel VITE_MOCK_KRW_FAUCET_ADDRESS and redeploy the Funding/Repayment frontend

[x] Blockchain Transaction Journal (PENDING / CONFIRMED / FAILED)

[x] Backend RPC Receipt/Event Verification Before Blockchain State Synchronization

[x] Verify/correct journal chain ID from the backend RPC network

[x] Recover a MetaMask replacement after reload when only the original tx hash is known

[ ] Upgrade the local contract test toolchain to remove Hardhat 2 / solc dev-only audit advisories

[ ] Production hardening: add per-receivable intent leases for tokenize, fund, and repay when simultaneous pre-hash submissions across different browsers must be prevented

[x] Tokenize

[x] Execute a real Seller tokenizeReceivable transaction on GIWA Sepolia and verify DB TOKENIZED/tokenId/tokenizeTxHash

[x] Funding

[x] Execute a real Funder approve + fundReceivable transaction on GIWA Sepolia and verify DB FUNDED/funder/fundingTxHash

[x] Confirm the Seller received the Funding mKRW on GIWA Sepolia

[x] Repay implementation

[x] Execute a real Buyer approve + repayReceivable transaction on GIWA Sepolia and verify DB REPAID/repayTxHash

[ ] Confirm the current NFT owner received the full faceValue mKRW on GIWA Sepolia

[x] Deploy and verify replacement MockKRW and ReceivableFinance with Hardhat

[x] Update tracked frontend and backend local configuration defaults to the verified replacement address pair

[ ] Update Railway and Vercel runtime contract address pairs and run a fresh CREATED-to-REPAID demo on the replacement deployment

---

## Deployment Verified

Railway is live and returns the configured Vercel origin for a normal preflight.
Vercel has been redeployed with the frontend API URL normalization fix, and the
browser API flow is working without CORS errors.

## Smart Contract Pre-Submission Validation

### P0 — End-to-End Happy Path

- [x] Add Hardhat tests for the complete receivable lifecycle:
  1. Seller creates a receivable
  2. Buyer verifies the receivable
  3. Seller tokenizes the receivable
  4. Funder approves MockKRW
  5. Funder funds the receivable
  6. Buyer approves MockKRW
  7. Buyer repays the receivable
- [x] Verify final receivable status is `REPAID`.
- [x] Verify the seller receives `fundingAmount`.
- [x] Verify the current NFT owner receives `faceValue` on repayment.
- [x] Verify the funder becomes the NFT owner after funding.
- [x] Verify all lifecycle events are emitted with correct arguments.

### P0 — Access Control and State Transitions

- [x] Verify only the registered buyer can call `verifyReceivable`.
- [x] Verify only the seller can call `tokenizeReceivable`.
- [x] Verify the seller and buyer cannot fund their own receivable.
- [x] Verify only the buyer can call `repayReceivable`.
- [x] Verify lifecycle functions revert when called in the wrong status.
- [x] Verify nonexistent receivable IDs revert with `ReceivableNotFound`.

### P0 — ERC20 Funding Requirements

- [x] Verify funding fails when the funder has insufficient MockKRW balance.
- [x] Verify funding fails when ERC20 allowance is insufficient.
- [x] Verify repayment fails when the buyer has insufficient MockKRW balance.
- [x] Verify repayment fails when ERC20 allowance is insufficient.

### P1 — Input Validation

- [x] Verify buyer cannot be the zero address.
- [x] Verify buyer cannot equal seller.
- [x] Verify face value and funding amount cannot be zero.
- [x] Verify funding amount cannot exceed face value.
- [x] Verify maturity date must be greater than issue date.

### MIDNIGHT

#### Current request-bound v2

- [x] Approve ADR-021: Funder supplies public policy criteria while the selected
  Seller/Buyer supplies private caller-provided mock facts and consent
- [x] Bind request ID, intended Funder wallet, thresholds, expiry, GIWA
  receivable/role/wallet, deployment, Provider, and freshness through Compact,
  Provider signature, capability, Spring, and Read API
- [x] Remove product JSON/PIN UX; generate the request-scoped pseudonym nonce
  inside the Bridge and retain `/midnight/legacy/*` only for v1 diagnostics
- [x] Clamp v2 role-wallet authorization to
  `min(issuedAt + 120 seconds, policyValidUntil)`, require an effective 1..120
  second TTL, and reject an already-expired policy as
  `409 / POLICY_REQUEST_EXPIRED`
- [x] Deploy local Compact evaluation v2 at
  `12caaf76aef1de1c584b67462018810f6e4e7eb2535e136f560cb621e24a3f36`
  and register deterministic Provider 2 in transaction
  `006abe69d8ba934519e19c4490ce77be724f75aae1bcb4e6b4fcd720258aa10601`
  at block `25714`, preserving the old v1 contract
- [x] Add authenticated Spring proof-request coordination with
  `REQUESTED -> SUBMITTED -> COMPLETED`, subject denial, expiry, failure,
  exact actor/context checks, and no raw-financial persistence
- [x] Encrypt the capability in MySQL with external
  `MIDNIGHT_CAPABILITY_ENCRYPTION_KEY`, versioned AES-256-GCM envelope/AAD, and
  HMAC idempotency fingerprint
- [x] Classify Read API/Indexer not-found as retryable while preserving
  `SUBMITTED`; classify permanent invalid capability as `FAILED`, purge its
  envelope, and release the active marker
- [x] Add an encrypted owner-only local Bridge outbox, request/session
  reservation, restart recovery, and ACK after Spring `SUBMITTED` or idempotent
  `COMPLETED` so a finalized capability is not lost or reproved during delivery
  failure
- [x] Release an expired `awaiting_authorization` reservation during recovery so
  the same tab can safely return to idle, while retaining ambiguous `proving`
  reservations through `validUntil` to prevent duplicate submission
- [x] Normalize the v2 capability's Midnight contract address as bare lowercase
  64-hex across Bridge, Vue, Spring, and Read API while keeping the EIP-712
  typed-data field `0x`-prefixed as `bytes32`
- [x] Verify current v2 boundaries: Compact `39/39`, Provider `87/87`, Read API
  `60/60`, CLI `156` passed + `1` optional environment skip, Vue `141/141`,
  Spring full Gradle `86/86` (focused Midnight `19/19`), plus CLI
  typecheck/build, Vue lint/build, and Spring `bootJar`
- [x] Enforce one active unexpired request per Funder, receivable, and role,
  including a completed result through `validUntil`
- [ ] Add policy templates plus per-Funder/subject query budgets, cooldowns,
  audit visibility, and abuse controls; current active uniqueness only reduces
  simple adaptive threshold probing
- [ ] Add versioned Spring envelope key rotation/reencryption or a documented
  drain/expiry ceremony before changing `MIDNIGHT_CAPABILITY_ENCRYPTION_KEY`
- [ ] Design and migrate independent per-company Midnight wallets, encrypted
  private states, and company secrets before remote multi-user use
- [ ] Restart all changed local processes and execute a fresh live v2 E2E:
  Funder request -> Seller and Buyer consent/proof -> Spring `SUBMITTED` ->
  Bridge ACK/restart recovery -> retryable Indexer read -> Funder `COMPLETED`
- [ ] Record a live negative-criteria result and confirm the UI states only
  “requested criteria not satisfied,” never bank verification, Funding
  rejection, or automatic GIWA gate

#### Historical v1 implementation record

The checklist below records ADR-008 through ADR-020's official-CLI,
fixed-policy, PIN, and manual capability learning path. Open v1 items do not
override the current v2 product requirements above.

- [x] Define `giwa-midnight/` as the dedicated Git-submodule workspace
- [x] Document local-only Midnight trust boundary, network, components, and data classification
- [x] Add `giwa-midnight` Git submodule from `leonid-world/giwa-midnight`
- [x] Initialize the Node 22 Midnight workspace and install official SDK dependencies
- [x] Official ZK Loan contract compiles
- [x] Local Midnight services are healthy
- [ ] Design and approve explicit persistent storage and recovery for the local
  Midnight Node and Indexer before treating contract addresses as durable across
  container recreation
- [ ] Add a bounded end-to-end CLI/Bridge join deadline so a nonexistent or
  newly unreachable deployment does not wait indefinitely in
  `watchForDeployTxData` and instead reports a clear `NOT_FOUND`. The Bridge's
  10-second startup preflight validates the address first but does not itself
  bound every later SDK join watcher
- [ ] Preserve and display actionable `Wallet.Sync` root causes, including the
  Node/Indexer endpoint and transport failure, instead of `[object Object]`
- [x] Official CLI synchronizes its wallet and deploys a contract on the local network
- [x] Preserve existing contract-scoped CLI private state when joining after a
  restart; create a fresh participant state only when none exists locally
- [x] Restore the replacement deployment's original encrypted admin private
  state after the pre-fix Join overwrite; verify the recovered derived key
  against public `contractAdmin` before and after the official provider write,
  without logging or persisting the plaintext secret
- [x] Mock provider is registered
- [x] Official loan proof succeeds and the public contract state is queryable
- [ ] Rotate the local Midnight storage password and recreate the encrypted CLI
  private-state DB when ready; the former `.env.example` value exists in the
  current submodule Git history, so decide separately whether to rewrite that history
- [x] Financial fields replace credit fields
- [x] GASOK eligible and ineligible proofs succeed via CLI and expose no raw financial values in public state
- [x] Add the initial dev-only Vue `/midnight` page and local read-only API for
  CLI-proven public eligibility results
- [x] Verify the initial Vue read side against the live local Indexer with no raw
  financial values exposed
- [x] Seal GIWA chain `91342` and ReceivableFinance
  `0x0f264334f98BA0d22f7Fc6Bb901a5Fa36158a315` into the new local Compact
  deployment
- [x] Resolve the canonical Seller/Buyer wallet for a uint256 receivable ID
  through GIWA RPC in the Mock Attestation Provider
- [x] Bind the provider signature to eight fields: private policy inputs,
  company-commitment hash, GIWA receivable subject, Midnight deployment,
  provider, and policy version
- [x] Store only an opaque lookup key with eligibility, provider ID, and policy
  version; reject an exact same-key replay
- [x] Verify receivable `#1` through CLI end to end with a separate eligible
  Seller result and ineligible Buyer result
- [x] Emit the correlation-sensitive proof capability only to the CLI terminal
  for intentional verifier sharing
- [x] Reject zero/out-of-range Provider secrets and the Jubjub identity public
  key in both Provider registration and proof verification
- [x] Pin the Mock Provider and read adapter to the approved local Midnight
  deployment, and reject another address before GIWA RPC or signing
- [x] Bound local Attestation requests/responses and timeouts; restrict CLI
  Provider URLs to a redirect-free loopback root
- [x] Keep fresh wallet mnemonics out of file logs and enforce owner-only `0600`
  CLI log permissions
- [x] Bound unresolved read-adapter Indexer work to one in-flight SDK query
- [x] Add the Provider 2 two-minute, one-shot EIP-712 issuance gate: CLI
  challenge/response handoff, dev-only `/midnight/authorize` MetaMask signing,
  and Mock Provider canonical EOA recovery before Schnorr attestation
- [x] Keep raw financial values and the hidden request salt out of Vue; exchange
  only the exact salted authorization request and minified one-line response
- [x] Register Provider 2 on the current replacement deployment and confirm the
  public contract state reports one registered Provider
- [x] Execute a real Seller MetaMask authorization plus local attestation, proof
  generation, Midnight submission, and independent Indexer result verification
- [ ] Decide and implement independent per-company Seller/Buyer Midnight
  participants, wallets, encrypted private states, and `companySecret` values.
  The current actor-aware Vue flow still demonstrates every role through one
  shared Bridge identity, so same-PIN capabilities can be cross-correlatable
- [ ] Define refresh rounds, allowed replacement behavior, freshness/latest
  selection, expiry, and revocation semantics before allowing another result for
  the same receivable subject
- [x] Supersede the v1 lost-capability limitation with a request-bound encrypted
  local outbox: reserve before proving, persist before `complete`, recover by
  request ID after reload/restart, and delete only after Spring delivery ACK
- [ ] If cross-receivable reuse of one company financial attestation is desired,
  approve a separate ADR for a time-bounded/revocable company credential plus a
  fresh per-receivable-role ZK presentation/nullifier. Keep the existing
  receivable/role binding until that replacement protocol and independent
  per-company private state are designed
- [x] Replace the normal v1 clipboard/file handoff with authenticated v2 Spring
  delivery bound to the intended Funder and an encrypted capability envelope;
  keep the local artifact only in explicitly labelled legacy diagnostics
- [ ] Replace the local PoC key/env and single-instance delivery assumptions
  with a production KMS, authenticated remote Bridge/subject identity,
  retention/deletion policy, revocation, and audit before multi-user release
- [x] Update the existing Vue viewer for Phase 2.5 capability-based exact
  resolution without publicly listing receivable-party correlations
- [ ] Add direct exhaustive frontend unit tests for the complete pre-existing
  capability schema/response module and role-authorization schema/real-signer
  module. The focused proof-flow suite covers its mocked integration boundary,
  timeout/abort, races, same-origin configuration, and both route flags
- [x] Approve ADR-018's trusted local Proof Bridge to reuse the already-proven
  CLI participant, encrypted private state, wallet balance, Provider 2 flow,
  Proof Server, and current contract
- [x] Correct the former Lace assumption: official Midnight Local Dev supports
  Lace on local `undeployed`; the Bridge is a minimal-change custodial PoC
  choice, not a technical requirement
- [x] Implement the port-4200 loopback Proof Bridge with CSPRNG body-only
  sessions, one active proof, one-shot authorization, no ambiguous auto-retry,
  bounded safe HTTP behavior, and a common CLI/Bridge private-state process lock
- [x] Add an internal authorization-deadline timer so an unsigned prepared tuple
  is discarded and its active slot is released even when no later poll arrives
- [x] Automatically purge terminal capability/error/status records after 60
  seconds with unref timers, without relying on lazy sweep or a later request
- [x] Bound the startup Indexer preflight to 10 seconds, keep port 4200 closed on
  failure, seal validated GIWA configuration in memory, and remove per-challenge
  Indexer queries while raw inputs exist; record the SDK's non-abortable single
  timed-out-startup-query limitation
- [x] Attempt encrypted-state sanitization after every Bridge success/failure,
  retry cleanup once, and require stale-witness sanitization before another
  prepare; disclose that dropping JavaScript references is not guaranteed
  memory zeroization
- [x] Immediately preserve and return a finalized proof capability without a
  per-session Indexer query; permit resolver-only retry and never proof
  resubmission for delayed public visibility
- [x] Add the separately gated dev-only `/midnight/prove` route; clear raw
  values/PIN after challenge creation, require an explicit MetaMask action, poll
  status, and independently resolve the completed capability via read API/Indexer
- [x] Replace free-form proof-subject entry with authenticated receivable
  selection: display DB/onchain/NFT IDs separately, derive the synchronized
  onchain ID and Seller/Buyer role, and block Funder/unrelated issuance
- [x] Separate the actor UX: Seller/Buyer start proof issuance from Receivables
  with their canonical role wallet; Funder opens `/midnight`, never signs for a
  party, and verifies only an intentionally delivered capability
- [x] Remove `/midnight/authorize` links from product-facing verifier/prover
  pages; retain that dev-only route only for direct CLI learning/diagnostic use
- [x] Bind Funder capability verification to a selected DB receivable and role;
  reject an onchain ID, ReceivableFinance, role, or party-wallet mismatch before
  the read adapter call and clear capability memory when context changes
- [x] Replace the copy/paste-only primary handoff with explicit issuer clipboard
  copy or local-file export plus Funder clipboard/file import; keep raw one-line
  JSON for advanced diagnostics, preserve context/reset cleanup, and warn that
  correlation-sensitive OS artifacts may remain after Vue memory is cleared
- [x] Make private-input guidance human-readable without narrowing Compact:
  comma-formatted integer-KRW revenue, percentage-to-bps conversion, full
  overdue/PIN uint ranges, temporary CSPRNG PIN, and explicit valid-ineligible
  behavior outside policy thresholds
- [x] Disable both plugin and runtime Vue Devtools while the raw-input proof
  route is enabled and verify no proof-flow value enters Pinia, storage, URL,
  logs, telemetry, Spring, or MySQL
- [x] Add Bridge session, HTTP security, one-shot/expiry/cancellation,
  private-state cleanup, and common process-lock tests; keep Docker/LevelDB live
  E2E separate from the non-environment suite
- [ ] Restore the CLI workspace ESLint binary/dependency and run its existing
  lint script; current CLI typecheck/build and 125-test pass result plus 1
  optional environment test skip are verified, but `npm run lint`
  currently reports `eslint: command not found`
- [x] Override and lock Restify 11's transitive `find-my-way`/`send` to
  `9.8.0`/`1.2.1`; confirm the installed tree and zero high-severity npm audit
  findings
- [x] Complete focused Vue route/service/composable tests for success, failure, expiry,
  cancellation, races, unmount, timeout, strict response matching, and
  independent final resolution; current Node 24.19.0 result is 14 files / 103
  tests plus full ESLint/Oxlint, changed-file Prettier, production build, and zero
  high-severity `npm audit` findings
- [x] Distinguish a stopped port-4200 Bridge from a malformed successful wire
  response: map Vite's non-JSON `502` proxy response to the safe local-service
  unavailable error, preserve ambiguous prove status-only recovery, and keep a
  successful non-JSON response invalid
- [x] Preserve only fixed Provider/GIWA operational errors through the
  CLI/Bridge/Vue boundary (`GIWA_RECEIVABLE_NOT_FOUND`,
  `GIWA_RPC_UNAVAILABLE`, `ROLE_WALLET_MISMATCH`) without reflecting Provider
  response bodies or request values
- [x] Normalize Compact's exact lookup-key duplicate to
  `ELIGIBILITY_RESULT_ALREADY_EXISTS`; show existing-capability reuse guidance,
  forbid PIN-change bypass language, and disclose that a lost capability cannot
  be recovered in the current MVP
- [x] Diagnose the 2026-08-19 DB `#4` / onchain `#1` Seller failure as an
  already-existing exact result rather than Docker, Provider, Proof Server,
  Node, or Bridge unavailability
- [x] Add `midnight/LOCAL_POC_RUNBOOK.md` with five copy-paste terminal blocks,
  readiness checks, DB/onchain/NFT identity guidance, Seller/Buyer issuance and
  Funder verification, input units/PIN meaning, safe shutdown, CLI/Bridge
  locking rules, and separate 502 diagnosis for a stopped Bridge versus the
  Provider/GIWA-RPC Challenge path
- [x] Replace the ephemeral Provider 2 with deterministic local development
  config `PROVIDER_SECRET_KEY=2`, register its public key once on the v2
  deployment, and pin Provider/Bridge preflight to that key
- [x] Verify the production artifact excludes the proof route registration,
  proof view/service chunk, and proof API marker, and smoke the live Seller `#1`
  challenge through Vue → Bridge with
  all four private values absent from the post-challenge DOM and captured console
- [ ] Remove the dead `midnight-prove` route-name string left in the production
  Receivables asset by the dev-only CTA, if complete compile-time elimination is
  desired. The route/chunk/API are already absent; do not present this dead
  string as a security boundary
- [ ] Repeat the browser flow with an available MetaMask provider; the in-app
  browser smoke stopped before EIP-712 signing and is not a proof/transaction E2E
- [ ] Execute real Seller and Buyer `/midnight/prove` local E2E runs before
  marking browser-triggered proof submission complete
- [ ] Treat direct Vue + Lace as a later self-custody replacement requiring a
  separate identity/private-state migration ADR, not a parallel hidden path
- [x] Supersede the former no-Spring v1 limitation with ADR-021's minimal
  authenticated request coordination and encrypted capability custody; keep
  raw facts outside Spring/MySQL and keep eligibility outside the Funding gate

Phase 1 complete: the workspace pins one physical
`@midnight-ntwrk/onchain-runtime-v3@3.0.0` instance, matching the official ZK
Loan lockfile and compatibility matrix. This fixed the duplicate-WASM-class
`StateValue` failure. The official CLI deployed the contract, registered Mock
Provider 1, fetched a mock attestation, generated and submitted the loan proof,
and queried the public contract state on the local `undeployed` network.

Phase 2 CLI complete: the Compact circuit privately evaluates integer KRW
annual revenue, debt ratio in basis points, and overdue count. The local CLI E2E
recorded both `eligible=true` at the exact policy boundary and `eligible=false`
one KRW below the revenue boundary. Each result entry contained only a
pseudonymous commitment, eligibility, Mock Provider ID, and policy version;
public admin and Provider-registry control state remained separate.

The initial Phase 3A list viewer is retired. The current read side accepts one
intentionally shared Proof capability in the development-only Vue `/midnight`
page and sends it to the adapter's exact
`POST /v1/eligibility-results/resolve` endpoint. The adapter is the sole
authority for the pinned Midnight contract, does not expose an anonymous result
list, and returns only the matching receivable, role, canonical party wallet,
and minimal proof result. The page does not submit proofs or change Funding,
Spring Boot, MySQL, MetaMask, or GIWA contracts.

Phase 2.5 binding was completed on the now-superseded local contract
`a8c0c1997c424dd1215d055fb5688200194263c7be5deef8b4e7620d2cdceb2c`.
That deployment separated Seller and Buyer results for one canonical GIWA
receivable context.

The Mock Provider resolves that context through GIWA RPC, the eight-field
signature prevents cross-context reuse, and the ledger publishes only an opaque
key plus the minimal result. Receivable `#1` produced Seller `true` and Buyer
`false` through the CLI from deliberately different caller-supplied mock inputs.
This proves role-context separation, not either party's actual finances. Live
adapter smoke tests resolved those same role-labeled outcomes and rejected a
tampered capability with HTTP 400. Actual development-browser submissions also
resolved Seller `true` and Buyer `false`, while Node 22 lint/build checks passed.
That is only the browser read path.

The standalone chain was reset when its Node container was recreated on
2026-08-17, so those results and capabilities remain historical evidence only.
The current replacement deployment is
`7e3ea9d741ce0f5862db6f46d0ad720be2586cd7d0405ec77e4a0478aa50f4fb`;
Provider 2 registration and the ADR-017 real Seller runtime E2E were completed
there. ADR-018 real Seller/Buyer browser-triggered runs remain pending.

ADR-017 code complete: Provider 2 now has a separate two-step issuance gate.
The CLI retains private values and a hidden salt, hands a two-minute typed
request to the dev-only Vue `/midnight/authorize` tool, accepts its one-line
MetaMask response, and sends it back to the Mock Provider. The Provider consumes
the challenge once, rechecks the GIWA role and private commitment, and recovers
the canonical EOA before issuing the unchanged Schnorr attestation. Compact and
the contract logic are unchanged and do not independently verify EIP-712;
Provider 1 results remain legacy. The local deployment address changed only
because the standalone chain was recreated.

The complete Attestation API suite passes `74/74`. CLI tests pass `125` with `1`
optional environment E2E skipped, and UI lint/build checks pass. Provider 2
registration and a real Seller MetaMask-to-Midnight local runtime E2E are
complete on the current replacement deployment. ADR-018 is accepted for the
trusted local Bridge and `/midnight/prove` flow. Secure multi-user capability
delivery/access, independent actors, public-result freshness/expiry, direct Lace
self-custody, and backend coordination remain later work.
