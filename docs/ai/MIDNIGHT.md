# GASOK Midnight PoC

## Scope

The Midnight PoC exists only on the `gasok-midnight` branch. It privately
evaluates caller-supplied mock financial inputs and binds each result to the
canonical Seller or Buyer role context of a specific GASOK receivable. It is
local-only and uses the Midnight `undeployed` network.

`giwa-midnight/` is the dedicated workspace and Git submodule. Its target
structure is `contract/`, `api/`, `cli/`, `attestation-api/`, and
`docker-compose.yml`.

GIWA remains the sole system for receivable tokenization, funding, repayment,
wallet mapping, and asset transactions. Midnight does not replace or deploy a
GIWA Solidity contract.

## Trust Model

The GASOK Attestation API is a local mock provider. It signs demonstration
financial inputs but does not prove that they originated from a bank, tax
authority, accounting firm, or ERP provider. Product copy must call the result
"mock-attested" and must never describe it as bank-verified.

Midnight proves that a registered provider signed the witness data for the
specific encoded GIWA/Midnight context, the signed data was not modified, and
the Compact eligibility circuit executed correctly. It does not prove that the
underlying real-world financial data is truthful.

The Mock Provider reads the configured GIWA Sepolia ReceivableFinance contract
through RPC, confirms the receivable exists, and selects its canonical Seller or
Buyer wallet. This prevents a request from substituting an arbitrary wallet for
that role. It does not prove that the submitted financial inputs belong to that
party. Provider ID `2` additionally requires a two-minute, one-shot EIP-712
authorization from that canonical role wallet before issuing its mock
attestation. Provider ID `1` results predate that gate and remain legacy
role-context-only results. Neither policy is proof of legal-company identity,
financial-data truth, or bank/accounting provenance.

ADR-018 adds a trusted local Proof Bridge so the complete flow can be triggered
from Vue without creating a second Midnight identity. MetaMask authorizes only
the canonical GIWA role through EIP-712. The Bridge, not MetaMask, owns the
existing Midnight development wallet, balance, encrypted private state, proof
orchestration, and transaction submission. This is an explicit local custodial
boundary and must not be presented as a production or self-custody design.

## Private and Public Data

Private inputs are annual revenue, debt ratio, overdue count, PIN-derived
company identity material, attestation signature, the user secret, and witness
randomness. In the ADR-018 route, Vue holds the entered tuple and PIN only in
component memory until one challenge response, then removes them from its
reactive state. The loopback Bridge processes them in memory and reuses the
CLI-compatible encrypted private state at rest. The local Mock Attestation API
must see the raw values to sign them, and the local Proof Server must process
the plaintext witness to generate a proof; both are inside the local PoC trust
boundary and must not persist or log the values. The Proof Server does not
receive the Midnight wallet key.

Raw values must not be stored in Pinia, browser storage, URLs, console logs,
telemetry, MySQL, Midnight public state, committed files, or general application
logs. Bridge cleanup drops transient financial/signature references and removes
them from encrypted state after proof work. The encrypted write is temporary:
cleanup is marked before it begins, attempted after success or failure, retried
once, and rechecked before another prepare may proceed. JavaScript immutable
strings cannot be guaranteed to be zeroized, so this is bounded lifetime,
sanitization, and process isolation, not a perfect memory-erasure claim. A wallet mnemonic is separate
from contract private state: the interactive CLI accepts it locally and shows a
freshly generated mnemonic once without logging it. The Bridge deliberately
reuses the public disposable Local Dev genesis seed from the standalone flow;
it is never sent to Vue and must never be reused on a network or asset with
value.

The Phase 2.5 public map is limited to an opaque receivable-eligibility lookup
key and `{ eligible, providerId, policyVersion }`. The contract also exposes its
admin, registered Provider public-key registry, and sealed GIWA chain and
ReceivableFinance configuration. The public eligibility result entry contains no
raw financial value, signature, receivable ID, role, party wallet, company
commitment, or business number.

After a proof succeeds, the interactive CLI prints or the Bridge returns a
versioned proof capability containing the company commitment, lookup key,
Midnight deployment, GIWA chain and ReceivableFinance address, receivable ID,
role, and canonical party wallet. It contains no PIN, secret, raw financial
value, or provider signature. It is still correlation-sensitive because it
links an otherwise opaque ledger entry to a specific public GIWA party, so it
must be delivered only to the intended verifier and must not be logged or
committed.

For Provider 2, the browser-facing EIP-712 request contains public receivable
context and a salted `attestationRequestCommitment`, not the financial tuple or
hidden salt. In `/midnight/prove`, the request and response move directly
between Vue component memory and the loopback Bridge. The preserved
`/midnight/authorize` tool still supports the manual CLI handoff. Neither path
may place them in logs, URLs, browser storage, MySQL, or Midnight public state.

The company commitment remains derived from a local secret and PIN. It is not a
legal business identity and is deliberately not an unsalted hash of GASOK's
low-entropy 10-digit business number.

## Phase 2.5 Signed Context

The current local Midnight contract address is
`7e3ea9d741ce0f5862db6f46d0ad720be2586cd7d0405ec77e4a0478aa50f4fb`.
It was constructed with the fixed GIWA context below:

- GIWA chain ID: `91342`
- ReceivableFinance:
  `0x0f264334f98BA0d22f7Fc6Bb901a5Fa36158a315`

The prior local deployment
`a8c0c1997c424dd1215d055fb5688200194263c7be5deef8b4e7620d2cdceb2c`
is historical only. Recreating the standalone Node on 2026-08-17 reset the
non-persistent local chain, so the old address and its proof results are no
longer present. Provider 2 registration and one full Seller runtime E2E have
completed on the replacement deployment.

The Mock Provider's Schnorr message contains exactly eight ordered fields:

1. annual revenue in integer KRW
2. debt ratio in basis points
3. overdue count
4. company-commitment hash
5. hash of GIWA chain, ReceivableFinance, receivable ID, role, and canonical wallet
6. hash of the Midnight contract deployment
7. provider ID
8. policy version

Changing the receivable ID, Seller/Buyer role, canonical wallet, GIWA chain,
ReceivableFinance deployment, Midnight deployment, provider ID, policy version,
company secret, or PIN invalidates reuse in another context. The ledger rejects
an already-existing lookup key, so an exact same-context submission is one-shot.
This replay rule is not a freshness policy: the contract has no issued time,
latest-result rule, expiry, revocation, or refresh round yet.

## Provider 2 Role-Wallet Authorization

Provider 2 adds an off-chain issuance gate without changing the Compact
contract or its Schnorr message:

1. The CLI retains the raw mock financial values and hidden random salt and
   requests an authorization challenge.
2. The Provider resolves the canonical Seller/Buyer wallet and returns an exact
   EIP-712 request with a two-minute expiry and salted request commitment.
3. The CLI prints that request for manual paste into the development-only Vue
   `/midnight/authorize` route.
4. Vue strictly validates the fixed context, calls MetaMask for exactly the
   canonical role wallet, checks the typed-data hash and recovered signer, and
   outputs a minified one-line response for paste back into the CLI.
5. On `POST /attest`, the Provider consumes the challenge before validation,
   re-reads the GIWA role, recomputes the private commitment, recovers the EOA
   signer, and only then issues the existing Schnorr attestation.

Compact verifies only the registered Provider's Schnorr signature and ZK policy
execution. It does not independently verify the MetaMask secp256k1 signature.
The two-minute challenge is one-shot transport authorization, not a result
freshness or expiry policy.

## Local Components and Ports

| Component | Purpose | Host endpoint |
| --- | --- | --- |
| Midnight Node | validates Midnight transactions | `http://127.0.0.1:9944` |
| Midnight Indexer | public-state query and subscription | `http://127.0.0.1:8088/api/v4/graphql` |
| Midnight Indexer WS | state subscription | `ws://127.0.0.1:8088/api/v4/graphql/ws` |
| Midnight Proof Server | local ZK-proof generation | `http://127.0.0.1:6300` |
| Mock Attestation API | mock financial-data signing | `http://127.0.0.1:4000` |
| Read-only Midnight API | resolves one exact proof capability against Indexer public state for the dev-only Vue viewer | `http://127.0.0.1:4100` |
| Local Proof Bridge | trusted CLI-compatible private-state/wallet orchestration for `/midnight/prove` | `http://127.0.0.1:4200` |

The proof server is localhost-only. Do not run a second independent container on
port 6300 while Midnight Local Dev is running.

## Required Delivery Order

1. Reproduce the official ZK Loan contract, CLI, and Attestation API unchanged.
2. Compile Compact and build the contract package.
3. Run the local Node, Indexer, and Proof Server.
4. Complete the official CLI flow: fund wallet, deploy, register provider,
   request attestation, generate proof, submit, and query public state.
5. Replace only the loan-domain fields with GASOK financial eligibility fields.
6. Connect the mock provider and test valid and invalid signatures through CLI.
7. Verify the full GASOK proof flow through CLI.
8. Bind Seller and Buyer results to a canonical GIWA receivable and verify the
   bound flow through CLI.
9. Only then add the isolated read-only Vue capability resolver.
10. Add the separate Provider 2 CLI-to-Vue-to-CLI EIP-712 authorization handoff
    without moving private financial values into the browser.
11. Register Provider 2 and verify the real MetaMask authorization plus complete
    local attestation/proof/transaction/Indexer flow end to end.
12. Add the approved local-only Proof Bridge and `/midnight/prove`: prepare the
    challenge, explicitly authorize with MetaMask, attest, prove, submit, poll,
    and resolve the resulting capability through the read API.
13. Treat direct Vue + Lace self-custody and secure multi-user capability
    delivery as later, separate architectures. Add Spring Boot only if an
    already-proven Vue flow needs backend coordination.

The development-only Vue route `/midnight` now resolves the Phase 2.5 result
through `giwa-midnight/api` and the same-origin Vite `/midnight-api` proxy. The
adapter uses the official Indexer public-data provider and generated
`GasokEligibility.ledger()` decoder and has no transaction or private-state
surface. It binds to `127.0.0.1`, uses `Cache-Control: no-store`, and is not a
Spring Boot API.

The adapter is the sole contract-address authority and pins
`7e3ea9d741ce0f5862db6f46d0ad720be2586cd7d0405ec77e4a0478aa50f4fb`.
Vue has no default-contract environment variable. The former anonymous GET
collection is removed. `POST /v1/eligibility-results/resolve` accepts only the
exact version-1 capability object, with no extra fields:

```text
{
  version: 1,
  midnightContractAddress,
  companyCommitment,
  lookupKey,
  giwaChainId,
  receivableFinanceAddress,
  onchainReceivableId,
  subjectRole,
  partyWallet
}
```

It returns:

```text
{
  networkId: "undeployed",
  contractAddress,
  context: { giwaChainId, receivableFinanceAddress, onchainReceivableId,
             subjectRole, partyWallet },
  result: { lookupKey, eligible, providerId, policyVersion }
}
```

The Vue page accepts a manual capability paste, validates basic syntax only,
and shows that one receivable role's result. It does not store the capability in
browser storage, logs, or a URL. Live API smoke resolved the role-bound demo for
receivable `#1` as Seller `eligible=true` and Buyer `eligible=false`, while
tampering returned HTTP 400. These outcomes came from intentionally different
caller-supplied mock inputs; they do not assert the actual financial condition
of either role wallet. Actual development-browser submissions resolved the same
Seller result as `eligible=true` and Buyer result as `eligible=false`; the Buyer
screen correctly explained that ineligibility is a valid proof outcome. This is
browser read verification, not browser attestation or proof submission.

The separate development-only `/midnight/authorize` route performs only the
MetaMask portion of the Provider 2 issuance handoff. It accepts the exact CLI
authorization request, receives no raw financial value or hidden salt, makes no
Attestation, Spring, or Midnight HTTP request, and returns a one-line signature
response for manual CLI paste. MetaMask still handles account/network selection
and EIP-712 signing. This is not Midnight proof submission and is not part of
the GIWA Funding transaction flow.

## ADR-018 Vue Proof Session

The development-only `/midnight/prove` page uses the same proven participant
without exposing its private state to Vue:

1. Vue POSTs one exact private-input object to the Bridge's
   `/v1/proof-sessions/challenge` endpoint.
2. The Bridge prepares the CLI-compatible private witness context and returns a
   random session ID, decimal-string expiry, and exact Provider 2 authorization
   request. Vue immediately clears the financial tuple and PIN.
3. The user explicitly asks MetaMask to sign as the canonical GIWA role wallet.
   Challenge creation never triggers signing automatically.
4. Vue POSTs the versioned authorization response to
   `/v1/proof-sessions/prove`. The Bridge consumes the session and progresses
   through `attesting` and `proving_and_submitting`; after finalization it uses
   `indexing` only as an immediate transition to `complete`.
5. Vue polls `/v1/proof-sessions/status` with the session ID in the JSON body.
   It does not retry proof submission after a timeout or unknown result.
6. `complete` returns only the proof capability. Vue sends that to the existing
   read adapter, which recomputes the lookup key and reads the public Indexer
   result. Only that independently resolved response becomes UI success. The
   Bridge does not query the Indexer per session after finalization; it preserves
   the capability and returns `complete`, while Vue retries only the resolver
   read and never resubmits the proof for delayed public visibility.

`failed`, `expired`, and `cancelled` are terminal. Cancellation is truthful: an
SDK proof/submission already running without abort support remains running and
status-checkable. The Bridge is single-session because its local wallet and
encrypted private state are one mutable participant. The interactive CLI and
Bridge cannot run simultaneously against that database.

Current official Midnight Local Dev supports Lace against the local Node,
Indexer, and Proof Server, and the wallet-connector guide includes an
`undeployed` configuration. Direct Vue + Lace is therefore technically viable
without Preprod. ADR-018 nevertheless uses the trusted local Bridge because the
current proof participant, encrypted private state, wallet balance, and deployed
contract have already been proven through the CLI. The Bridge avoids adding and
migrating a second Midnight identity while the owner learns the end-to-end
runtime. A later direct Lace self-custody path requires a separate replacement
ADR rather than being mixed into this one.

Official references pinned for this decision:

- [Midnight Local Dev Lace setup](https://github.com/midnightntwrk/midnight-local-dev/blob/8b44aabc5ea65e4c5d4cd855017517600bc90e8a/README.md#L76-L89)
- [Local proving roles and endpoints](https://github.com/midnightntwrk/midnight-docs/blob/90da63c74fb92cf156505c682df07dbaba61be62/docs/guides/local-proving.mdx#L267-L325)
- [Wallet connector `undeployed` configuration](https://github.com/midnightntwrk/midnight-docs/blob/90da63c74fb92cf156505c682df07dbaba61be62/docs/guides/react-wallet-connect.mdx#L180-L240)
- [SDK compatibility matrix](https://github.com/midnightntwrk/midnight-docs/blob/90da63c74fb92cf156505c682df07dbaba61be62/docs/relnotes/support-matrix.mdx#L17-L38)

## Initial Eligibility Policy

The initial private-input policy is:

- annual revenue >= 500,000,000 KRW
- debt ratio <= 200%
- overdue count <= 1

Threshold values and policy version are contract configuration, not raw company
financial data. Any policy change after the first working CLI proof requires a
documented contract-version decision.

## Local Security Hardening

- Provider secret keys must be in `1..JubjubOrder-1`; the API does not reduce an
  invalid configured key modulo the order. Compact rejects the Jubjub identity
  public key `(0, 1)` during registration and again during proof verification.
- The Mock Provider signs only for its pinned approved Midnight deployment. It
  accepts uncompressed JSON up to 4,096 bytes and uses bounded HTTP timeouts.
- Provider 2 authorization challenges are random, bounded, two-minute, and
  one-shot. The Provider atomically consumes the challenge on the first
  attestation attempt, rejects zero/expired/replayed or context-mismatched
  material, re-resolves the GIWA role, and recovers the EOA signer before
  Schnorr issuance.
- CLI calls to the Mock Provider are restricted to an HTTP loopback root URL,
  reject redirects, time out after 10 seconds, and cap success/error bodies at
  64 KiB. Remote response bodies and sensitive URL parts are not logged.
- CLI log files use owner-only `0600` permissions. Fresh mnemonics bypass the
  logger and are displayed once only in an interactive terminal.
- The Proof Bridge binds to literal `127.0.0.1:4200`, enables no CORS, and
  accepts only allowlisted local `Origin` and `Host` values,
  `Sec-Fetch-Site: same-origin`, the custom UI header, `application/json`,
  identity encoding, no query string, exact object keys, a 4,096-byte body, and
  bounded header/request timeouts.
- Session IDs are CSPRNG values held only in Bridge memory and sent only in POST
  bodies. One active session, one authorization use, no automatic ambiguous
  retry, and a common CLI/Bridge process lock protect the single local encrypted
  state and wallet. An internal timer expires an unsigned prepared session at
  the Provider deadline, discards its tuple, and frees the slot without relying
  on a later poll. Terminal capability/error/status records are automatically
  purged after 60 seconds by unref timers even if no further request arrives.
- Vue clears raw financial/PIN form state after challenge creation, disables
  devtools while the raw-input route is enabled, and stores none of the proof
  flow in Pinia, browser storage, URLs, logs, or telemetry. Bridge responses and
  errors never reflect input values, bodies, secrets, stacks, or witness data.
- The Bridge attempts to sanitize transient finance, signature, and Provider
  fields after proof work, retries the encrypted cleanup once, and makes a later
  prepare sanitize stale fields before it can proceed. It drops JavaScript
  references but does not claim reliable zeroization of immutable strings.
- A `complete` Bridge status contains the proof capability only. Vue must
  resolve it again through the read adapter/Indexer; the Bridge is not an
  independent public-result authority. Transaction finalization immediately
  preserves and returns the capability; recovery from Indexer delay is
  resolver-only.
- Before port 4200 opens, one 10-second-bounded Indexer preflight validates the
  pinned deployment and seals its GIWA configuration in memory. Challenge
  preparation uses that cache and never queries the Indexer while raw inputs
  exist. The SDK query has no abort signal, so one timed-out startup query may
  remain unresolved internally; the server remains closed and no raw tuple has
  been accepted. The later SDK contract-join watcher is not yet bounded end to
  end; port 4200 still remains closed until it finishes, and a complete join
  deadline is tracked separately.
- The read adapter returns a timeout after 10 seconds and allows only one live
  upstream Indexer query while the SDK operation remains unresolved, preventing
  repeated requests from piling up unabortable work.

## Exclusions

- No Midnight Preprod or Mainnet deployment.
- No React dependency or React conversion.
- No Lace Wallet requirement during CLI Phase 1.
- No claim that the ADR-018 Bridge is self-custodial or required by a Lace
  limitation. Official local Lace support exists; direct Vue + Lace is simply
  not the selected architecture in this phase.
- No raw financial-data persistence in MySQL or public ledger state.
- No real bank, credit-bureau, or accounting-provider claim.
- No change to existing GASOK flows until the proof has succeeded through CLI.
- No claim that Provider 1 legacy results prove wallet control. Only Provider 2
  applies the EIP-712 issuance gate.
- No legal-company identity proof.
- No issued time, freshness, latest-result, expiry, or refresh-round policy.
- No claim that an existing result describes the party's current financial
  condition; the Vue verifier must display this freshness limitation.
- No eligibility-based GIWA Funding gate.
- No general publication of proof capabilities; they are correlation-sensitive.
- No anonymous public-result collection endpoint.

## Current ADR-017 Verification Status

- The complete Attestation API suite passes `72/72`.
- CLI tests pass `60` with `1` optional environment E2E skipped.
- Vue changed-file formatting, ESLint, Oxlint, production build, and development
  build pass.
- Provider 2 is registered on the current replacement deployment and its public
  state reports one Provider. A real Seller MetaMask authorization completed
  the Provider 2 → Schnorr → Midnight proof → transaction → Indexer E2E in
  transaction `00d7ed17d2d109ad0f0490bd6ea116b57745befcb6a7d0662dd922936374657045`
  at block `2854`. The independently decoded public result is `eligible=true`,
  Provider `2`, policy `1`.

## Current ADR-018 Verification Status

- The port-4200 Bridge, session store, runtime, local preflight, and common
  CLI/Bridge process lock are implemented.
- Under Node 22.21.1, the current suite reports 20 files / 242 tests passed with
  1 optional environment file/test skipped, including exact HTTP/session
  security, one-shot state transitions,
  non-reflective failure responses, private-state cleanup behavior, and lock
  safety. CLI typecheck and build pass.
- The installed CLI dependency tree overrides Restify 11's transitive
  `find-my-way` and `send` to `9.8.0` and `1.2.1`; both resolve to those versions
  and `npm audit --audit-level=high` reports zero high-severity vulnerabilities.
- CLI lint is not counted as passed: the workspace currently declares the lint
  script/config but has no installed ESLint binary, so `npm run lint` exits with
  `eslint: command not found`. Restoring the lint dependency and running it is a
  separate remaining verification item.
- The `/midnight/prove` route and its service/composable/view boundaries are
  implemented. Under Node 24.19.0, the focused Vue suite passes 8 files / 36
  tests covering the Bridge wire contract, body size/timeout/abort behavior,
  one-shot and ambiguous-submission recovery, sequential polling, transient
  input cleanup, independent resolution, race/unmount guards, accessible
  eligible/ineligible views, route gating, literal-loopback strict-port proxy
  policy, and both plugin/runtime Vue Devtools disabling.
- Changed-file ESLint, Oxlint, and Prettier checks and the production Vite build
  pass, and `npm audit --audit-level=high` reports zero high-severity
  vulnerabilities. Direct exhaustive unit coverage of the complete pre-existing capability
  and role-authorization modules remains a separate TODO; their dependencies
  are mocked at the focused proof-composable boundary.
- Node 24.19.0 applies to `giwa-ui` verification only. The Midnight CLI and
  Proof Bridge remain pinned and verified on Node 22.21.1.
- The production artifact contains zero proof-route/chunk/API marker matches. A
  live development-browser smoke created the Seller `#1` challenge through Vue
  and the Bridge; all four private form values were then absent from the DOM and
  no synthetic value appeared in captured console output.
- That browser had no MetaMask provider. The smoke therefore does not count as
  EIP-712 signing, proof submission, a Midnight transaction, public resolution,
  or the required real Seller/Buyer browser E2E.
- Docker/real LevelDB Bridge E2E and real Seller/Buyer `/midnight/prove`
  browser-triggered runs are not included in that unit/boundary result and must
  remain visibly pending until executed.
