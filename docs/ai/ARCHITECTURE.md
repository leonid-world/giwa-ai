# Architecture

## Network Boundary

GIWA and Midnight are separate networks with separate responsibilities. The
Midnight PoC is local-only on `undeployed`; GIWA remains the existing testnet
integration. No Midnight deployment reaches Preprod or Mainnet.

```text
GASOK Vue (existing giwa-ui)
  ├─ Existing GIWA lifecycle UI ──────────────────────────────┐
  └─ Dev-only Midnight tools                                   │
     ├─ /midnight/prove ── raw input, explicit MetaMask EIP-712│
     │                         │                                 │
     │                         ▼                                 │
     │   Proof Bridge (127.0.0.1:4200, trusted local process)   │
     │     ├─ encrypted private state + Midnight dev wallet     │
     │     ├─ Mock Provider 2 attestation                       │
     │     └─ proof generation + transaction submission        │
     ├─ /midnight/authorize ← preserved manual CLI handoff      │
     └─ /midnight verifier ── capability ────────────────┐      │
                                                           │      │
GASOK Spring Boot (existing giwa-api)                      │      │
  ├─ auth / business logic / MySQL                          │      │
  └─ GIWA RPC verification and transaction journal          │      │
                                                           │      │
GIWA Sepolia                                               │      │
  ├─ ReceivableFinance / MockKRW / NFT settlement ◀────────┘      │
  └─ canonical Seller/Buyer read ───────────────────────┐          │
                                                        │          │
Midnight PoC (giwa-midnight, local-only) ◀──────────────┼──────────┘
  ├─ CLI and Proof Bridge encrypted private state        │
  ├─ Mock Attestation API (127.0.0.1:4000) ◀─────────────┘
  ├─ Proof Server (127.0.0.1:6300; witness, no wallet key)
  ├─ Midnight Node (127.0.0.1:9944)
  ├─ Indexer (127.0.0.1:8088) → minimal public result
  └─ Read-only API (127.0.0.1:4100) → exact capability resolver
```

## Component Ownership

| Component | Owns | Must not own |
| --- | --- | --- |
| Vue | Dev-only `/midnight/prove` form, transient raw input/PIN memory until challenge creation, explicit canonical-role MetaMask EIP-712 signing, proof-session polling, and independent capability resolution; preserved manual `/midnight` and `/midnight/authorize` tools | raw-input persistence or logging, hidden salt, Midnight wallet/private-key custody, direct private-state access, direct Proof Server/Node calls, Funding enforcement, GIWA architecture changes |
| Spring Boot | authentication, MySQL, REST API, GIWA receipt/event verification and journal | Attestation-provider signing in the initial PoC, Midnight transaction signing |
| GIWA contracts | receivable ownership, tokenization, funding, repayment, current-NFT-owner settlement | financial eligibility proof or private financial data |
| Midnight contract | sealed GIWA deployment configuration, eight-field provider-signature verification, private eligibility proof, one-shot opaque-key result | raw financial values, GIWA assets, GIWA lifecycle state, independent GIWA RPC reads |
| Attestation API | canonical Seller/Buyer role-context resolution from GIWA RPC; bounded two-minute one-shot EIP-712 challenges, EOA signer recovery, and Provider 2 mock financial-input/context signing | bank/accounting-provider claim, legal-company identity proof, financial-data truth, MySQL persistence |
| CLI | deployment, provider registration, manual proof submission, local encrypted private state, two-step Provider 2 authorization handoff, and correlation-sensitive proof-capability output | MetaMask key custody, Vue replacement, production wallet flow, secure verifier delivery channel |
| Local Proof Bridge | one localhost-only proof session at a time; reuse of the current CLI encrypted state and development wallet; Provider 2 orchestration; proof generation and Midnight submission; transient capability return | remote/multi-user service, production signer, raw-input persistence/logging, Funding/Spring/MySQL responsibilities, automatic ambiguous retry |
| Proof Server | local proof generation from the plaintext witness supplied by the Bridge/CLI | wallet key, transaction signing, public remote exposure, or persisted raw financial data |
| Indexer | query of public Midnight result | private witness/state query |
| Read-only Midnight API | one pinned Midnight deployment, exact `POST /v1/eligibility-results/resolve`, capability/key recomputation, local Indexer lookup, and generated Compact-ledger decoding for Vue | anonymous result listing, wallet, proof, attestation, mutation, Spring/MySQL responsibilities, or verifier authentication |

## Data Classification

| Data | Location |
| --- | --- |
| annual revenue, debt ratio, overdue count, PIN, signatures, witness secret | `/midnight/prove` component memory only until challenge creation; then trusted loopback Bridge memory and encrypted CLI-compatible private state. The Mock Attestation API and Proof Server process required plaintext transiently without persistence or logging |
| EIP-712 authorization request and response | direct `/midnight/prove` sessions keep them in component/Bridge memory; the preserved `/midnight/authorize` tool still supports manual CLI copy/paste. Both forms contain a salted request commitment rather than raw financial values or the hidden salt |
| Midnight wallet mnemonic/seed | interactive CLI mnemonics are supplied locally or shown once and never logged; ADR-018 reuses the public disposable Local Dev genesis seed already used by the standalone flow and must never use it on a network or asset with value |
| contract admin, registered Provider public keys, sealed GIWA chain/address, opaque receivable-eligibility lookup key, provider ID, policy version, eligibility | Midnight public state and Indexer |
| company commitment plus GIWA receivable ID/role/wallet and lookup key | CLI/Bridge proof capability; not raw financial data, but correlation-sensitive; transiently pasted or returned to Vue and POSTed to the local adapter without browser-storage, log, or URL persistence |
| Proof-session ID and state | cryptographically random, memory-only Bridge record; sent only in request bodies, one-shot, short-lived, and automatically discarded by a 60-second terminal timer or process restart |
| account/company identity, receivable lifecycle, GIWA transaction proof summaries | existing Spring Boot/MySQL and GIWA chain according to existing rules |

## Phase 2.5 Receivable-Subject Binding

The local Compact deployment is fixed to GIWA chain `91342` and
ReceivableFinance `0x0f264334f98BA0d22f7Fc6Bb901a5Fa36158a315`. For each
request, the Mock Attestation API reads `getReceivable(id)` through GIWA RPC and
selects the canonical Seller or Buyer wallet for the requested role. The
Midnight contract itself does not call GIWA RPC.

The provider signs exactly eight fields, in protocol order:

1. annual revenue
2. debt ratio
3. overdue count
4. pseudonymous company-commitment hash
5. GIWA binding hash over chain, ReceivableFinance, receivable ID, role, and wallet
6. Midnight deployment hash
7. provider ID
8. policy version

The circuit recomputes those hashes, verifies the signature, evaluates the
private policy, and writes only
`opaque lookup key -> { eligible, providerId, policyVersion }`. Seller and Buyer
therefore have different lookup keys even for the same receivable; their result
values depend on the caller-supplied mock inputs and may be equal or different.
An existing key is rejected, so the exact same proof context is one-shot.

After submission, the CLI prints a proof capability containing the lookup data
needed to correlate that opaque Midnight entry with one GIWA receivable party.
The capability contains no PIN, company secret, raw financial value, or provider
signature, but possession reveals that correlation and must be treated as
privacy-sensitive.

The read-only adapter is the sole configured Midnight-address authority and is
pinned to
`7e3ea9d741ce0f5862db6f46d0ad720be2586cd7d0405ec77e4a0478aa50f4fb`.
It accepts only the exact version-1 capability object at
`POST /v1/eligibility-results/resolve`, recomputes the binding, deployment, and
lookup key, and reads that single map entry. The former anonymous
`GET .../eligibility-results` list is no longer exposed. The dev-only Vue page
pastes the capability through the same-origin `/midnight-api` proxy and shows
the exact receivable ID, Seller/Buyer role, canonical party wallet, eligibility,
provider ID, and policy version.

## ADR-017 Provider 2 Issuance Authorization

Provider ID `2` is reserved for the EIP-712-authorized mock issuance policy.
The CLI keeps the private financial tuple and hidden random salt, asks the local
Provider for a two-minute challenge, and prints only the typed authorization
request. The development-only Vue `/midnight/authorize` tool validates the
exact fixed GIWA/Midnight/Provider context, selects
`message.partyWallet` through MetaMask, signs the EIP-712 message, verifies its
hash and recovered signer locally, and returns minified one-line JSON for manual
paste back into the CLI.

On the first attestation attempt, the Provider atomically consumes the
challenge, re-reads the canonical role wallet from GIWA RPC, recomputes the
salted request commitment, and independently recovers the EOA signer. Only then
does it issue its existing Schnorr attestation. Raw financial values, the hidden
salt, PIN, company secret, Provider signature, and Midnight private state never
enter Vue.

This is an off-chain issuance gate. It does not require a change to the Compact
contract, eight-field Schnorr message, public ledger schema, or sealed GIWA
configuration. Compact proves the registered Provider 2 Schnorr
signature and private policy execution; Midnight does not independently verify
the secp256k1 EIP-712 signature. Provider ID `1` entries remain honest legacy
role-context-only results and must not be described as wallet-authorized.

## ADR-018 Local Vue Proof Bridge

The approved Phase 3B path adds a trusted Node.js Proof Bridge inside the
`giwa-midnight/cli` workspace and a development-only Vue `/midnight/prove`
route. This choice reuses the current, runtime-proven CLI participant identity,
encrypted private state, wallet balance, Provider 2 flow, and deployed Compact
contract. It is not a claim that Lace is unable to use local `undeployed`.

Current official Midnight Local Dev documentation supports Lace against the
local Node, Indexer, and Proof Server endpoints (`9944`, `8088`, and `6300`),
and the official wallet-connector guide includes an `undeployed` configuration.
A direct Vue + Lace self-custody architecture is therefore viable later. It is
not mixed into this PoC because the current Vue application has no Midnight
DApp Connector, Lace participant state, wallet balance, or migration from the
already-proven CLI identity. See the pinned
[official Local Dev README](https://github.com/midnightntwrk/midnight-local-dev/blob/8b44aabc5ea65e4c5d4cd855017517600bc90e8a/README.md#L76-L89)
and
[wallet-connector `undeployed` example](https://github.com/midnightntwrk/midnight-docs/blob/90da63c74fb92cf156505c682df07dbaba61be62/docs/guides/react-wallet-connect.mdx#L180-L240).

The Bridge is explicitly custodial inside this disposable local PoC. Vue sends
the private mock tuple and PIN to loopback memory to prepare a Provider 2
challenge, then immediately clears those form values. MetaMask authorizes only
the public GIWA role context. The Bridge holds the local Midnight development
wallet and encrypted private state, receives the resulting authorization,
obtains the Provider signature, asks the Proof Server to process the plaintext
witness, and submits the Midnight transaction. The Proof Server does not receive
the wallet key. The public ledger remains limited to the opaque key and minimal
result.

Proof sessions are CSPRNG-identified, body-only, memory-only, short-lived,
single-active, and one-shot. Submission begins only after an explicit MetaMask
action. The frontend polls status and never automatically retries an ambiguous
transaction. A complete session returns a capability, which Vue must resolve
again through the independent port-4100 read adapter and Indexer; it must not
trust a Bridge-supplied eligibility boolean. Once the transaction is finalized,
the Bridge immediately preserves that capability and completes without a
per-session Indexer query. The browser retries resolver reads only and never
submits a second proof for delayed public visibility. CLI and Bridge share a
fail-fast process lock around the encrypted state database.

Before port 4200 opens, the Bridge performs a 10-second-bounded Indexer
preflight, validates the pinned contract and Provider, and seals the GIWA
configuration in memory. Each challenge uses that validated cache instead of
querying the Indexer while raw inputs exist. The current SDK query has no abort
signal, so one timed-out startup query may remain internally unresolved; the
server stays closed and no raw tuple has been accepted in that condition.

This decision preserves the existing `/midnight` capability verifier and
`/midnight/authorize` manual learning tool. It does not add Spring Boot, MySQL,
a GIWA Funding gate, React, Preprod, or Mainnet. A later move to direct Lace
self-custody would require a separate ADR covering identity/state migration and
would replace, not silently coexist with, the custodial Bridge architecture.

## Integration Sequence

1. `giwa-midnight` CLI proves the official ZK Loan example locally.
2. The same CLI proves the GASOK financial-eligibility contract.
3. Phase 2.5 binds separate Seller and Buyer proofs to a canonical GIWA
   receivable context and verifies them end to end through the CLI.
4. The Phase 3A read side now resolves one manually pasted Phase 2.5 capability
   without publicly enumerating results. This is verification, not a secure
   delivery channel or proof-submission UI.
5. ADR-017 adds the separate CLI-to-Vue-to-CLI EIP-712 authorization handoff for
   Provider 2. The current replacement deployment completed a real Seller
   MetaMask authorization followed by Provider attestation, CLI proof creation,
   Midnight submission, and Indexer result verification. At that ADR-017 stage,
   Vue remained only the authorization bridge; ADR-018 adds the separate
   Bridge-backed proof route without changing the manual tool.
6. ADR-018 adds the development-only `/midnight/prove` route and trusted
   loopback Proof Bridge. The Bridge reuses the proven CLI wallet/private state;
   Vue explicitly authorizes with MetaMask, polls proof submission, and
   independently resolves the returned capability through the read adapter.
7. Direct Vue + Lace is a viable later self-custody replacement, not a blocker
   or a parallel implementation. Secure capability delivery/access remains a
   separate product concern beyond this local single-user session.
8. Spring Boot integration is considered only when required by the proven Vue
   flow. Existing GIWA lifecycle APIs and Solidity contracts remain unchanged.

Phase 2.5 and ADR-017 still do not enforce a GIWA funding gate. Provider 2 proves
control of the canonical role wallet only at mock-attestation issuance time; it
does not prove that the private values belong to that wallet, legal-company
identity, bank verification, accounting provenance, or data truth. The
two-minute expiry bounds the authorization lifetime, while atomic one-shot
challenge consumption prevents replay. Neither makes the result fresh: the
ledger still has no issued time, latest-result rule, expiry, revocation, or
refresh round. Independent Seller/Buyer Midnight identities, lifecycle policy,
secure verifier capability delivery, any direct Lace migration, and later
backend coordination remain separate work.

## Local PoC Security Controls

- Provider secrets must be canonical non-zero Jubjub scalars. The Compact
  registry and verification circuit reject the identity public key `(0, 1)` so
  it cannot be used to forge a signature for an arbitrary message.
- The Mock Attestation API pins one approved Midnight contract before it reads
  GIWA or signs, accepts JSON bodies up to 4,096 bytes, and applies local HTTP
  request/header timeouts.
- Provider 2 challenges are cryptographically random, bounded in memory,
  limited to two minutes, and consumed before validation on the first
  attestation attempt. The Provider rejects expired, zero, replayed, or
  context-mismatched authorization material and recovers the canonical EOA
  signer off-chain.
- The CLI accepts only an HTTP loopback Attestation API root URL, rejects
  redirects, bounds the full response to 64 KiB, and applies a 10-second
  deadline without reflecting remote bodies or sensitive URL data into logs.
- CLI log files are owner-only (`0600`). A freshly generated wallet mnemonic is
  shown once on the interactive terminal and never sent to the file logger.
- The Proof Bridge binds to literal loopback port `4200`, accepts only the
  configured local development Origin/Host allowlists,
  `Sec-Fetch-Site: same-origin`, and custom UI header, enables no CORS, requires
  exact JSON/body-only session requests, and returns no-store safe errors
  without request-value, body, stack, secret, or witness reflection.
- One active proof session and a common CLI/Bridge process lock prevent
  concurrent mutation of the same encrypted private state. Session IDs are
  cryptographically random, memory-only, one-shot, and never placed in URLs. An
  internal deadline timer discards an unsigned prepared tuple and frees the
  active slot even when the client never polls again. Complete/failed/expired/
  cancelled records, including capability or safe error data, are automatically
  purged after 60 seconds without relying on another request.
- Vue removes raw financial fields and PIN after challenge creation and never
  writes them, authorization material, session IDs, or capabilities to Pinia,
  browser storage, URLs, logs, telemetry, Spring Boot, or MySQL. Devtools are
  disabled while the raw-input proof route is enabled.
- The Bridge temporarily writes financial and Provider-signature witness fields
  to its encrypted local private state for proof generation. Cleanup is marked
  before the write awaits, is attempted after success or failure, and is retried
  once idempotently; the next prepare also sanitizes stale transient fields and
  cannot continue unless that write succeeds. A finalized capability is not
  discarded merely because cleanup storage reports an error. JavaScript
  immutable strings cannot be guaranteed to be zeroized, so the local-only
  trust model relies on dropping references, bounded lifetime, sanitization,
  and process isolation rather than claiming perfect memory erasure.
- The read adapter permits one unresolved Indexer query at a time. Because the
  current Midnight SDK query has no abort signal, this bound prevents timed-out
  requests from accumulating orphaned upstream operations.
