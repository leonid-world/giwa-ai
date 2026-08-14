# Current Context

Project

GIWA Hackathon

Current Stage

End-to-End MVP Lifecycle Complete

Submission Deployment Preparation

Current Focus

Replacement Contract Address Rollout and Fresh Demo Lifecycle

Midnight PoC Context

The `gasok-midnight` branch additionally contains a local-only Midnight privacy
PoC. `giwa-midnight/` is an initialized Git submodule workspace backed by
`https://github.com/leonid-world/giwa-midnight.git`; it contains Node 22
workspace metadata and the official Midnight SDK dependency lockfile. Phase 1
must reproduce the official ZK Loan Compact/CLI/Attestation flow unchanged on
the `undeployed` network before any GASOK financial-field or Vue work. Node
24.19.0, Docker 29.6.2, and Compact 0.5.1 are installed. Use Node 22.21.1 for
the official API and CLI runtime through `nvm use 22`; Compact compilation has
also been verified with Node 24. The official Proof Server is running locally.

Phase 1 progress: the unmodified official ZK Loan `contract/` source has been
imported from `midnightntwrk/example-zkloan`. With Node 24.19.0 and Compact
0.5.1, its eight Compact circuits compiled successfully and generated
`contract/src/managed/zkloan-credit-scorer/{contract,keys,zkir,compiler}`.
The official Midnight Local Dev Node (`127.0.0.1:9944`), Indexer
(`127.0.0.1:8088`), and Proof Server (`127.0.0.1:6300`) are running and healthy
through the official standalone Docker Compose definition.
The official mock Attestation API is running on `127.0.0.1:4000` under Node
22.21.1 and its health endpoint returns provider ID 1. Node 24.19.0 cannot run
this official Restify/SPDY dependency path because it no longer exposes
`http_parser`; use Node 22 for the official API and CLI runtime, while Compact
compilation remains verified on Node 24. The contract simulator passes 61/61
tests. Phase 1 is complete: the official CLI synchronized its wallet, received
local NIGHT/DUST, deployed the contract, registered Mock Provider 1, fetched a
mock attestation, generated and submitted a loan proof, and queried the public
contract state on the local `undeployed` network.

The initial Provider registration failure, `expected instance of StateValue`,
was caused by two physical WASM runtime copies after the custom workspace lock
resolved `@midnight-ntwrk/onchain-runtime-v3` to both 3.1.0 and 3.0.0. The
official ZK Loan lockfile uses one 3.0.0 instance. The workspace therefore pins
and hoists exactly one 3.0.0 runtime through a direct dependency plus npm
override/resolution. Do not remove that compatibility pin while Midnight.js
remains on 4.1.1.

The tracked CLI `.env.example` previously contained a non-empty storage
password and the value is present in the current submodule Git history. The
working template is blank now. Treat the former password as exposed; rotate the
ignored local `.env` value and recreate the local encrypted private-state DB
together when the current disposable Phase 1 state is no longer needed. Do not
delete or rewrite either one automatically.

Phase 2 GASOK CLI proof flow is complete. The transformed Compact contract uses
`annualRevenueKrw: Uint<64>`, `debtRatioBps: Uint<32>`, and
`overdueCount: Uint<16>` as private witness values and applies policy version 1:
revenue at least 500,000,000 KRW, debt ratio at most 20,000 basis points
(200.00%), and overdue count at most 1. The official loan amount, tier,
authorized amount, response flow, blacklist, and loan/PIN-migration ledger data
were removed because they have no approved GASOK meaning.

The local Mock Attestation API accepts decimal strings, validates Compact
integer ranges, signs the three values plus a pseudonymous commitment hash,
does not echo raw values, and binds only to `127.0.0.1`. Contract tests pass
18/18 and API tests pass 16/16. A local CLI E2E deployed the GASOK contract,
registered Mock Provider 1, submitted valid eligible and ineligible proofs, and
queried two public results containing only commitment, eligibility, Provider ID,
and policy version.

Phase 3A public-result viewing is complete. `giwa-midnight/api` is a
localhost-only read adapter on `127.0.0.1:4100`; it fixes the network to
`undeployed`, queries the local Indexer through the official provider, decodes
state with the generated `GasokEligibility.ledger()`, and returns only the four
approved public result fields. `giwa-ui` adds a development-only authenticated
`/midnight` learning page, isolated service/composable, Vite proxy, navigation,
and Dashboard entry. A live browser check displayed the two existing eligible
and ineligible results. The page does not handle raw financial values, wallets,
proofs, attestations, or Funding decisions, and no Spring Boot integration has
begun.

Full browser proof submission is not implemented. The official ZK Loan UI is
Preprod-only because Lace cannot balance or sign for local `undeployed`; Preprod
is prohibited here. The next Phase 3B step therefore requires approval of a
local-wallet approach or a Node bridge and its new trusted-signer boundary.

Database Contract

business_number = CHAR(10), digits only

UI Contract

business number display = 000-00-00000

Authenticated pages use a shared navigation header. The global account summary
shows only the current login email.

The dedicated My Information page shows the login email and company wallet
connection state/address. It does not expose internal user/company IDs.

All active frontend routes use the same minimalist B2B SaaS visual baseline,
1200px content width, 8px spacing scale, one green brand color, neutral surfaces,
subtle borders, form focus state, and button interaction hierarchy. Blockchain
lifecycle actions are presented as vertical timelines. This is a presentation
layer only; route structure and business flows are unchanged.

Error Contract

Common JSON error response with stable error codes

Backend implementation

- GlobalExceptionHandler
- SecurityErrorHandler
- ApiException
- BlockchainTransactionController
- BlockchainTransactionService
- BlockchainTransactionMapper
- BlockchainTransactionVerifier
- GiwaJsonRpcClient
- BlockchainRpcProperties
- BlockchainTransactionFailureRecorder

Frontend implementation

- src/services/api.js
- ApiError
- AuthStore, WalletStore, ReceivableStore use apiRequest
- src/assets/base.css and src/assets/main.css
- App-level authenticated navigation header
- src/views/ProfileView.vue
- src/contracts/ReceivableFinance.abi.json
- src/contracts/MockKRW.abi.json
- src/contracts/addresses.js
- src/services/web3/provider.js
- src/services/web3/receivableContract.js
- src/services/web3/mockKrwFaucet.js
- src/composables/useMockKrwFaucetClaim.js
- src/services/blockchainTransactions.js
- src/views/FundingView.vue
- src/views/RepaymentView.vue
- src/views/NotFoundView.vue
- `@lucide/vue` icon components
- public/favicon-64.png, public/apple-touch-icon.png, public/og.png, and public/og-saas.png
- public/robots.txt and public/sitemap.xml
- vercel.json SPA history fallback

Authenticated layout

- Every route with `meta.requiresAuth` displays a compact shared navigation
  header with Dashboard, Receivables, Funding, Repayment, and My Information.
- The minimum account identity is the authenticated email from `/auth/me`.
- Internal user/company IDs, business number, and wallet address are not shown in
  the shared header.
- `/profile` reuses `/auth/me` and `/wallet/me` to show only the login email,
  company wallet connection state, and full wallet address.
- A missing company wallet is a normal profile state; wallet lookup errors other
  than 404 remain visible and retryable.
- App startup loads the current user for authenticated pages, including a direct
  dashboard refresh.
- Concurrent page/layout calls share one in-flight `/auth/me` request.

Frontend visual polish

- `main.js` loads the project baseline stylesheet. It applies the light theme,
  one primary brand color, neutral canvas/surfaces, typography, 8px spacing
  tokens, box sizing, and minimum app height.
- The removed starter resets no longer override semantic heading and `strong`
  weights or introduce an unsupported automatic dark theme.
- The authenticated shell, Login, Dashboard, Profile, Receivables, Funding,
  Repayment, and 404 routes use Lucide icons instead of decorative emoji and
  retain visible text for accessibility.
- Redundant nested cards, gradients, and decorative shadows were removed. The
  remaining form and workspace boundaries use 1px neutral borders and 8px/12px
  radii so the product keeps the same layout with less visual noise.
- Receivables, Funding, and Repayment workspaces align to the shared 1200px
  content width. Their list panes and detail panes share one flat workspace
  boundary rather than appearing as unrelated cards.
- Receivables now presents CREATED, VERIFIED, TOKENIZED, FUNDED, and REPAID as a
  five-step vertical lifecycle timeline. Funding and Repayment present approval
  and execution as two-step vertical transaction timelines.
- Existing lifecycle guards, journal recovery, MetaMask actions, API payloads,
  disabled states, and synchronization handlers are unchanged; the timeline
  state is display-only and reuses the existing conditions.
- Interactive list rows expose selected, hover, focus-visible, and
  `aria-pressed` states without changing selection behavior.
- Mobile rules reduce outer and panel padding, wrap header actions, and preserve
  the existing single-column workflows.

Frontend public demo release quality

- The base document uses Korean language metadata, the GIWA product title and
  description, browser theme color, canonical URL, Open Graph fields, and
  Twitter Card fields.
- The favicon uses a lightweight 64px derivative of the existing GIWA
  receivable/NFT mark. A 180px Apple touch icon and a minimalist single-brand
  1200x630 social preview card are public assets.
- `robots.txt` allows the public demo entry point and references a sitemap that
  lists only the public root URL. Authenticated routes and the not-found route
  switch the runtime robots directive to `noindex, nofollow`.
- Every route has a product-specific browser title. Unknown client routes render
  a branded 404 page rather than an empty RouterView.
- Vercel rewrites history-mode paths to `index.html`, so direct visits and
  refreshes can reach Vue Router and the application 404 page.
- The shared shell keeps the existing header and route layout while adding a
  compact GIWA demo footer and a keyboard skip link.
- Dashboard, Profile, Receivables, Funding, and Repayment now render loading
  before data, confirmed empty state only after a successful read, and explicit
  retry/failure guidance where applicable. Login retains its existing
  submit-loading state.
- These changes are release/UI state only. Authentication, wallet mapping,
  receivable lifecycle, API payloads, and Web3 transaction behavior are
  unchanged.

Receivable explorer links

- The receivable detail panel keeps the full contract address and lifecycle
  transaction hashes visible.
- Contract, create, verify, tokenize, funding, and repayment metadata expose
  button-style links to the configured GIWA explorer in a new tab.
- Explorer buttons are hidden when the metadata or `VITE_GIWA_EXPLORER_URL` is
  missing.

Receivable onchain flow

DB CREATED

→ Buyer can review Seller, Buyer, amounts, dates, wallets, and document hash

→ Seller MetaMask createReceivable

→ ReceivableCreated event ID

→ POST /receivables/{id}/chain-created

→ Buyer explicitly accepts the displayed debt terms

→ Frontend reads getReceivable and compares DB data with onchain CREATED data

→ Buyer MetaMask verifyReceivable

→ POST /receivables/{id}/verified

→ DB VERIFIED

→ Seller reads and compares the VERIFIED onchain receivable

→ Seller MetaMask tokenizeReceivable

→ ReceivableTokenized event and escrow NFT mint

→ POST /receivables/{id}/tokenized with txHash only

→ Backend stores RPC-verified tokenId and DB TOKENIZED

→ Unrelated authenticated companies discover TOKENIZED funding opportunities

→ Funder verifies DB/onchain terms, payment token, NFT escrow, mKRW balance, and allowance

→ Funder explicitly approves the exact funding amount of mKRW

→ Funder separately confirms fundReceivable

→ ReceivableFunded + mKRW Transfer + escrow NFT Transfer

→ POST /receivables/{id}/funded with txHash only

→ Backend stores RPC-verified Funder metadata and DB FUNDED

→ Buyer reviews the full face value, current NFT owner, mKRW balance, and allowance

→ 잔액 부족 시 Buyer가 사전 예치된 데모 mKRW를 지갑당 1회 충전

→ Buyer explicitly approves the exact face value of mKRW

→ Buyer separately confirms repayReceivable

→ ReceivableRepaid + Buyer-to-current-owner mKRW Transfer

→ POST /receivables/{id}/repaid with txHash only

→ Backend stores the RPC-verified repayment hash and DB REPAID

Synchronization contract

- `chain-created` does not change the DB status.
- `verified` changes CREATED to VERIFIED and writes status history.
- `tokenized` changes VERIFIED to TOKENIZED, stores the RPC-verified token ID and
  tokenize transaction hash, and writes status history.
- `funded` changes TOKENIZED to FUNDED, stores the authenticated Funder company
  and wallet, configured MockKRW address, funding transaction hash, and status history.
- `repaid` changes FUNDED to REPAID, stores the repayment transaction hash, and
  writes Buyer status history.
- All five synchronization APIs are idempotent for the same blockchain metadata.
- Different metadata returns 409 BLOCKCHAIN_METADATA_CONFLICT.
- `(contract_address, onchain_receivable_id)`, create tx hash, and verify tx hash
  have same-stage database uniqueness constraints.
- Service checks reject a transaction hash already stored in any current lifecycle
  tx column, and `blockchain_transactions.tx_hash` now enforces race-safe global
  uniqueness across lifecycle stages.
- Frontend stores txHash in per-company browser local storage immediately after
  submission, before waiting for a receipt.
- A reload resumes the existing receipt check and event parsing instead of
  submitting a duplicate contract transaction.
- Confirmed event data remains stored until backend synchronization succeeds.
- Retry calls only the backend synchronization API; it never repeats the confirmed contract call.
- When browser recovery data is missing, the Seller UI reconciles DB `VERIFIED`
  receivables with the server transaction journal before exposing the mint action.
- An existing CONFIRMED tokenization journal is recovered through the idempotent
  backend synchronization API without opening MetaMask again.
- If a server-recovered CONFIRMED synchronization fails or its journal state
  changes, retry first re-reads the journal. It never silently downgrades the same
  button into a MetaMask receipt-recovery action.
- An existing PENDING tokenization journal blocks a new mint and resumes the
  original transaction receipt check. A journal lookup failure also keeps minting
  locked until the Seller explicitly retries the check.
- If the browser has a submitted tokenization hash that has not reached the server
  journal yet, preserve its nonce/calldata/replacement metadata ahead of a
  different server PENDING row. The server row remains available for the next
  reconciliation.
- The recovery candidate order is DB `TOKENIZED`, any TOKENIZE `CONFIRMED`
  transaction, TOKENIZE `PENDING`, and finally `FAILED`; a later failed attempt
  never hides an earlier confirmed mint.
- The journal gate detects already-submitted hashes. Two browsers that click in the
  same pre-submission window can both pass the read check before either hash is
  journaled; the contract prevents a second NFT mint, but a server-side
  per-receivable submission lease is future hardening if failed gas must also be
  prevented.
- Frontend role buttons use authenticated companyId.
- A Buyer-owned CREATED receivable is selected first when available; otherwise the
  first visible receivable is selected, so pending Buyer work is not hidden behind
  an additional list click.
- A Buyer review panel is visible for every Buyer-owned CREATED receivable, including
  while Seller chain creation is still pending.
- VERIFIED workflow guidance is role-specific: Seller sees Buyer verification
  completion, while Buyer sees its receivable verification completion. Internal
  TODO wording is not exposed in the user interface.
- The review panel provides a state refresh action so Buyer can pick up Seller chain
  creation without leaving the page; refreshing resets the local attestation checkbox.
- The review panel shows the parties, registered wallets, amounts, dates, document
  hash, and an explicit debt-attestation checkbox.
- The checkbox is a local signing guard, not a separate backend approval state.
- Immediately before `verifyReceivable`, the frontend reads `getReceivable` and
  requires the onchain ID, Seller, Buyer, amounts, dates, document hash, and CREATED
  status to match the backend response.
- Receipt event parsing accepts logs emitted only by the configured
  ReceivableFinance contract.
- Frontend preflight comparison remains a Buyer UX and accidental-mismatch guard.
  Backend RPC receipt/event verification is the authoritative synchronization guard.
- Contract calls request the receivable's registered wallet explicitly with
  `BrowserProvider.getSigner(expectedAddress)` instead of using the first permitted
  MetaMask account.
- If the registered wallet is not currently permitted for the site, the frontend
  opens MetaMask account permissions once and checks again before returning a
  WALLET_MISMATCH that includes the expected and first permitted addresses.
- Every write still verifies the resolved signer address against the receivable
  wallet immediately before signing.
- uint256 event values remain bigint/string and are never converted to JavaScript Number.

Blockchain transaction journal

- MetaMask hash submission creates a backend PENDING row before receipt waiting.
- The backend derives company, stored receivable wallet, chain ID, and function name
  instead of trusting those client fields.
- CREATE_RECEIVABLE, VERIFY_RECEIVABLE, TOKENIZE_RECEIVABLE, FUND_RECEIVABLE,
  and REPAY_RECEIVABLE are supported.
- REPAY_RECEIVABLE requires the registered Buyer while the DB receivable is
  FUNDED.
- Transaction hashes and addresses are normalized to lowercase.
- `blockchain_transactions.tx_hash` provides race-safe global uniqueness.
- Identical create/confirm/fail retries are idempotent; conflicting reuse returns
  `409 / BLOCKCHAIN_TRANSACTION_CONFLICT`.
- The frontend-confirmed block/gas fields are format-checked hints only. The backend
  fetches the transaction, receipt, latest block, and canonical receipt block from
  `GIWA_RPC_URL` before writing CONFIRMED.
- RPC verification requires the configured chain and ReceivableFinance address,
  expected signer/target, zero native value, exact ABI selector/arguments, a
  successful canonical receipt, minimum confirmations, and exactly one expected
  lifecycle event.
- CREATE validates every onchain receivable field and binds the emitted receivable
  ID; VERIFY binds the receivable ID and Buyer; TOKENIZE binds the receivable ID,
  token ID, custodian, and zero-address ERC-721 mint Transfer. FUND binds the
  receivable ID, token ID, Funder, Seller, funding amount, MockKRW payment
  Transfer, and escrow-to-Funder ERC-721 Transfer. REPAY binds the receivable ID,
  token ID, Buyer, current NFT-owner recipient, face value, and the matching
  Buyer-to-recipient MockKRW Transfer.
- The RPC-derived chain ID replaces the provisional wallet-mapping chain snapshot.
  Block number/hash, gas used, effective gas price, event IDs, and `rpc_verified_at`
  are stored as the verification proof summary.
- `verification_version` provides optimistic CAS ordering for success and failure
  results. A stale concurrent RPC result cannot overwrite a newer result.
- Only the submitting company can update a journal row; related companies can list
  the receivable journal.
- New chain-created, verified, tokenized, funded, and repaid database
  synchronization requires a matching RPC-verified CONFIRMED journal whose
  emitted receivable ID matches the requested onchain ID.
- Legacy CONFIRMED rows with no `rpc_verified_at` are verified and backfilled before
  they can authorize a new lifecycle synchronization.
- Even an already RPC-verified CONFIRMED row is checked again immediately before
  the first receivable lifecycle write, so a post-confirmation reorg cannot reuse a
  stale proof. A newly canonical placement refreshes the stored proof summary.
- Existing already-synchronized metadata keeps legacy-safe idempotent retry behavior.
- A 60-second receipt timeout remains PENDING.
- A successful replacement uses the replacement hash and marks the original FAILED
  with `TRANSACTION_REPLACED`; local storage retains both hashes until recovery is
  complete.
- Legacy browser records created before the journal integration backfill the
  PENDING/CONFIRMED journal before lifecycle synchronization. If exact lifecycle
  metadata is already present in the database, the stale browser record is cleared
  without attempting an invalid state transition.
- A cancelled replacement is retained locally until the original transaction can
  be marked FAILED, so a temporary journal API failure remains retryable.
- RPC unavailability, missing receipts, insufficient confirmations, and possible
  reorgs remain retryable without changing PENDING to FAILED.
- After a coherent canonical proof reaches the configured confirmation depth, a
  reverted receipt or deterministic signer/target/calldata/event mismatch marks
  the unsynchronized journal row FAILED and returns a stable error code.
- A lifecycle success refresh holds the journal row through the receivable write;
  a late failure cannot mark a hash FAILED after that hash is already synchronized.
- If concurrent verification changes the proof first, the caller receives
  `BLOCKCHAIN_VERIFICATION_RETRY_REQUIRED` and keeps its local recovery record.
- A client synchronization payload whose claimed event ID differs from the valid
  RPC proof uses `BLOCKCHAIN_SYNCHRONIZATION_EVENT_MISMATCH`; it does not corrupt
  the confirmed journal.
- New submissions persist a pre-send scan block and public transaction metadata.
- Reload recovery uses ethers replacement detection while the original transaction
  remains RPC-readable.
- If the original transaction is no longer returned by RPC, recovery scans bounded
  canonical block ranges with a persisted cursor. Stored metadata requires the same
  sender and nonce; legacy hash-only records require a unique exact lifecycle call.
- Exact-intent repricing resumes under the replacement hash. Changed calls and
  cancellations are terminal; missing or ambiguous candidates remain retryable
  without resubmitting a contract call.

Funding flow

- `GET /receivables/funding-opportunities` exposes only TOKENIZED, unassigned
  receivables to companies that are neither Seller nor Buyer.
- Before funding, a candidate can read the receivable detail but can list only its
  own FUND_RECEIVABLE journal rows. Seller, Buyer, and the assigned Funder retain
  full related-journal visibility.
- The Funding page validates `paymentToken()`, MockKRW decimals 0, the complete
  DB/onchain receivable terms, TOKENIZED status, token ID, and NFT escrow ownership.
- The page displays the exact funding amount, mKRW balance, and allowance.
- When that balance is insufficient, the page separately reads the configured
  MockKRWFaucet without weakening or replacing the normal Funding preflight.
- The testnet recharge CTA requires deployed Faucet/token bytecode, the expected
  MockKRW link and zero-decimal model, an eligible wallet, enough Faucet
  inventory, enough native gas, and a fixed claim that makes the wallet balance
  sufficient for the selected receivable.
- A claim is signed by the registered Funder MetaMask wallet. The client verifies
  the exact `Claimed` and MockKRW `Transfer(Faucet, Funder, claimAmount)` events,
  then refreshes the existing balance and allowance readiness.
- Submitted claim hashes are stored per wallet until a receipt/state check proves
  success or failure. Reload and temporary GIWA RPC lag cannot expose a duplicate
  claim action while the original result is uncertain.
- Funding and Repayment share the same wallet-keyed Faucet claim composable and
  browser record. A claim submitted on one page cannot be submitted again from
  the other page while its outcome is uncertain.
- Funding opportunity detail, journal, readiness, and Faucet reads share a
  selection generation guard, so a slower response for a previously selected
  receivable cannot overwrite the current receivable UI.
- Approval and funding are separate user actions. Approval never automatically
  opens the fundReceivable signature request.
- Approval is not a receivable lifecycle journal row. Its successful receipt,
  exact Approval event, and refreshed allowance are the client-side readiness
  proof; the final funding receipt proves the actual token movement.
- FUND_RECEIVABLE uses the existing PENDING/CONFIRMED/FAILED journal and
  per-company reload/replacement recovery.
- A confirmed funding whose DB synchronization failed is retried through
  `POST /receivables/{id}/funded` without submitting another MetaMask transaction.
- Backend confirmation requires the exact fundReceivable calldata, Funder signer,
  ReceivableFunded event, MockKRW Transfer from Funder to Seller, and ERC-721
  Transfer from ReceivableFinance escrow to Funder.
- The backend writes FUNDED only when the RPC-derived event token ID matches the
  stored token ID. The client supplies only the funding transaction hash.

Repayment flow

- `/repayment` lists receivables for which the authenticated company is the Buyer
  and the DB status is FUNDED.
- The page validates the configured Finance and MockKRW addresses,
  `paymentToken()`, MockKRW decimals 0, complete DB/onchain terms, FUNDED status,
  token ID, stored Funder, and the current NFT owner.
- The current NFT owner is the repayment recipient. It may differ from the
  original Funder after an ERC-721 transfer and must never be inferred from the
  stored Funder wallet.
- The page displays face value, maturity date, Buyer wallet, current NFT owner,
  Buyer mKRW balance, and allowance. The current contract does not enforce a
  maturity-time gate, so maturity is informational and the frontend does not add
  a client-only block.
- When the Buyer balance is below `faceValue`, the Repayment page conditionally
  validates the same pre-funded Faucet and allows one claim only when the fixed
  amount makes the selected repayment affordable.
- The Buyer claim uses the registered Buyer MetaMask wallet, verifies exactly one
  `Claimed` and one Faucet-to-Buyer MockKRW `Transfer`, and refreshes the existing
  repayment balance/allowance readiness. It never automatically approves or repays.
- Repayment detail, journal, readiness, and Faucet reads use a selection generation
  guard so a slower response for a previously selected receivable cannot overwrite
  the current Buyer workflow.
- Approval and repayment are separate user actions. Approval uses the exact face
  value and never automatically opens the repayReceivable signature request.
- REPAY_RECEIVABLE uses the shared journal and per-company localStorage recovery.
  A CONFIRMED repayment takes priority over PENDING or FAILED attempts.
- Two tabs can still pass the pre-hash journal check before either transaction is
  recorded. The contract prevents a second state transition, while a server-side
  per-receivable intent lease remains future hardening to prevent reverted gas
  for tokenize, fund, and repay submissions.
- A confirmed onchain repayment whose DB synchronization failed exposes only
  `POST /receivables/{id}/repaid` retry and never submits another MetaMask
  repayment.
- Receivable management redirects shared Funding/Repayment recovery records to
  their dedicated workflow pages so the generic lifecycle UI cannot invoke the
  wrong synchronization handler.
- Frontend receipt validation requires exactly one ReceivableRepaid and one
  MockKRW Transfer from Buyer to the current NFT owner for the full face value,
  plus receipt-block REPAID status and unchanged NFT ownership.
- Backend RPC verification binds the exact repayReceivable calldata, Buyer signer,
  ReceivableRepaid IDs/Buyer/recipient/face value, and the matching MockKRW
  Transfer before writing DB REPAID.
- Repayment reuses the existing `repay_tx_hash`, status-history, and transaction
  journal columns; no schema migration is required.

Wallet UX

MetaMask account selection → address confirmation → company mapping

Reconnecting the same mapped address refreshes the stored wallet chain ID.

Duplicate wallet

409 WALLET_ALREADY_MAPPED → show conflict → select another MetaMask account

Guardrail for next work

- New backend errors that require dedicated UX must use a stable ApiException code.
- New frontend API calls must use src/services/api.js.
- Do not convert all errors to 403.
- Keep 401 authentication, 403 authorization, and 409 resource conflict distinct.

Deployment

Hardhat

GIWA Sepolia

Deployment status

- Hardhat compile, test, deployment, and verification now share Solidity
  `0.8.24+commit.e11b9ed9`, optimizer enabled with 200 runs, viaIR disabled, and
  EVM version Paris.
- The pinned local `solc` package supplies the compiler, and Hardhat artifacts
  preserve the Standard JSON input used for Blockscout verification.
- `npm run deploy:giwa` deploys MockKRW and then ReceivableFinance, verifies
  `paymentToken()`, and writes public metadata to
  `giwa-contrract/deployment/giwa-testnet.json`.
- `npm run verify:giwa` verifies both contracts against the GIWA Blockscout API;
  `npm run deployment:env` prints the matching frontend/backend environment
  values.
- `npm run mkrw:transfer -- <recipient> <amount>` distributes existing owner
  balance without increasing supply, while
  `npm run mkrw:mint -- <recipient> <amount>` explicitly creates additional
  test-only supply. Both Hardhat tasks attach to the recorded deployment and
  validate chain, code, zero decimals, current onchain owner, balances, event,
  and post-state before reporting success.
- `giwa-contrract/MKRW_OPERATIONS.md` is the Korean operator guide for securely
  loading the temporary Owner key, choosing transfer versus mint, checking the
  explorer result, pre-funding the deployed Faucet, cleaning the shell variable,
  and resolving common errors.
- After a successful receipt and exact Transfer event, the owner tasks read
  balances and total supply at the confirmed block with bounded retries. Public
  RPC post-receipt state lag produces a success warning instead of a false
  transaction failure; the submitted transaction must never be sent again.
- Owner-to-Funder transfer
  `0x276dc7572aa09e47b2cc55e1b75ae4e111cfbe1cb45f89ef73985a4716032ba0`
  succeeded on GIWA Sepolia for `10,000 mKRW`. RPC verification confirmed status
  1, the exact Transfer event, Owner balance `999,990,000`, Funder balance
  `10,000`, and unchanged total supply `1,000,000,000`.
- A subsequent user-executed `100,000 mKRW` transfer increased the same Funder
  wallet's RPC-verified current balance to `110,000 mKRW`.
- `MockKRWFaucet` provides a self-service demo-token path for online reviewers.
  The owner pre-funds the Faucet with existing mKRW, and each wallet may claim one
  constructor-configured fixed amount. Claims transfer existing inventory, so
  they do not increase `totalSupply`.
- The Faucet validates the payment-token contract address and claim amount at
  deployment, rejects duplicate claims, reports depleted inventory without
  consuming claim eligibility, and lets only the owner recover remaining demo
  inventory when the Faucet is retired. It does not receive MockKRW ownership or
  mint permission.
- The GIWA Sepolia Faucet is deployed at
  `0xa451FA95c3E2Efd771f6Ba556daBBf36f888ef2E` with fixed claim amount
  `10,000,000 mKRW`. Deployment transaction
  `0xcd71ff3c79c17bfc4627a22c87a72139c30cd5a69cf686025744f5b30f8cc633`
  was confirmed in block `32667847` and recorded separately from the verified
  MockKRW/ReceivableFinance pair.
- `npm run deploy:faucet:giwa` now provides a Faucet-only Hardhat deployment
  path. It reads the existing pair metadata without modifying it, validates the
  GIWA chain, recorded deployer, live MockKRW identity/owner/zero decimals, and
  native gas balance, then deploys a default `10,000,000 mKRW` fixed claim unless
  `MKRW_FAUCET_CLAIM_AMOUNT` is explicitly set.
- A successful Faucet receipt is written separately to
  `deployment/giwa-testnet-faucet.json` before post-deployment RPC reads. Rerunning
  the command recovers and validates that address without submitting a duplicate
  deployment when GIWA public-RPC visibility is delayed.
- The Funding and Repayment UIs read `VITE_MOCK_KRW_FAUCET_ADDRESS` only after
  detecting an insufficient actor balance. A missing/bad Faucet configuration
  remains local to that callout and never blocks a wallet that already has enough
  mKRW.
- The implemented UX is: insufficient mKRW balance -> registered Funder or Buyer
  MetaMask wallet calls `claim()` -> receipt and exact token transfer are checked
  -> the existing workflow readiness is refreshed -> approval and funding/repayment
  remain separate explicit actions.
- Claim submission and recovery never call the backend or add a receivable
  lifecycle journal row. A per-wallet browser record blocks duplicate submission
  across reloads until the existing receipt or onchain claim state is reconciled.
- The Owner has pre-funded the Faucet with `600,000,000 mKRW`, enough for sixty
  fixed `10,000,000 mKRW` claims. Total supply was not increased. The private key
  remains only in the user's terminal and must never be copied into chat or a
  repository file.
- The Faucet solves only demo mKRW distribution. Reviewers still require GIWA
  Sepolia native ETH for the claim, approve, and fund/repay transactions; gas
  sponsorship remains outside the MVP PoC.
- `MKRW_OPERATIONS.md` documents MetaMask balance visibility: use GIWA Sepolia,
  activate the exact recipient account, import the current deployment's MockKRW
  address with symbol `mKRW` and decimals `0`, distinguish old deployments with
  the same symbol, and refresh the network without resubmitting a transfer.
- The same guide records the complete MetaMask custom-network fields for GIWA
  Sepolia: RPC `https://sepolia-rpc.giwa.io`, chain ID `91342`, native symbol
  `ETH`, and explorer `https://sepolia-explorer.giwa.io`.
- Verification rechecks the live chain, both contracts' deployed code, recorded
  compiler settings, and the ReceivableFinance-to-MockKRW payment token link
  before submitting source metadata to Blockscout.
- The verified replacement deployment is MockKRW
  `0x5cD8a99Dcf5Fa00fb4fD9873b41A15F9C13C9d3F` and ReceivableFinance
  `0x0f264334f98BA0d22f7Fc6Bb901a5Fa36158a315`.
- The first Finance post-deployment `paymentToken()` read briefly returned empty
  data from the public RPC even though the creation receipt succeeded. Explorer
  and RPC recovery confirmed the existing deployment, and both contracts were
  verified without redeploying.
- Deployment and verification now retry contract-code/state reads during public
  RPC visibility lag, while completed receipt metadata is persisted before the
  post-deployment read.
- The deployer private key is accepted only through the current terminal's
  `DEPLOYER_PRIVATE_KEY`; it is never stored in the repository.
- Hardhat local EVM tests pass for the full CREATED→REPAID lifecycle, permissions,
  current NFT-owner repayment, and ERC-20 failure rollback.
- `npm audit --omit=dev` reports zero non-development dependency vulnerabilities.
- The Hardhat 2 / solc development-only dependency tree still has audit advisories
  without a non-breaking fix; it is used only with trusted local contract sources
  and remains a toolchain-upgrade TODO.
- Existing MySQL must receive the non-destructive
  `.codex/migrations/20260730_receivable_chain_metadata_uniques.sql` migration.
- Existing journal tables with none of the four RPC proof columns must receive
  `.codex/migrations/20260730_blockchain_transaction_rpc_verification.sql`, then
  tables without `verification_version` must receive
  `.codex/migrations/20260730_blockchain_transaction_verification_version.sql`
  before the backend is deployed. A table created by the current create-table
  migration already contains all five and must not run either ALTER.
- Never rerun the destructive `.codex/schema.sql` against the populated local database.
- User-confirmed MockKRW and ReceivableFinance deployment to GIWA Sepolia is complete.
- User-confirmed real Seller createReceivable and Buyer verifyReceivable transactions
  succeeded.
- User-confirmed real Seller tokenizeReceivable transaction, NFT mint, and backend
  TOKENIZED synchronization succeeded.
- User-confirmed real Funder approval, fundReceivable transaction, DB FUNDED
  synchronization, and Seller mKRW receipt succeeded.
- User-confirmed real Buyer approval, repayReceivable transaction, and DB REPAID
  synchronization succeeded. The current NFT owner's exact face-value balance
  increase remains to be confirmed.
- The original Remix deployment remains historical. The replacement Hardhat
  deployment is complete and both contracts are verified on Blockscout.
- Existing DB chain metadata, NFTs, MockKRW balances, allowances, and journal
  proofs remain bound to the original contract pair. Never rewrite those rows to
  the replacement addresses; use a fresh demo database or new lifecycle data.
- The Vercel frontend deployment is live.
- The Repayment backend/frontend update still requires Railway and Vercel
  redeployment before production-origin validation.
- Public replacement contract addresses, transaction hashes, blocks, deployer,
  and compiler settings are recorded in
  `giwa-contrract/deployment/giwa-testnet.json`.
- Frontend development/production/example env files and the backend
  `application.yml` fallback addresses now use the verified replacement pair.
- Vercel `VITE_*` and Railway `GIWA_*` dashboard variables still override those
  files and must be updated together before their next deployments.
- Railway backend deployment uses the `giwa-api/Dockerfile` Java 17 build instead of
  relying on Railpack Java auto-detection.
- Backend runtime supports Railway `PORT`, Railway MySQL variables, exact
  comma-separated Vercel CORS origins, and public `GET /health`.
- `GET /health` is a liveness endpoint and does not validate MySQL connectivity or
  schema readiness.
- Blank legacy `DB_*` variables must not remain on Railway because they override
  the corresponding `MYSQL*` fallback values.
- Railway is deployed at `https://giwa-api-production.up.railway.app`, and
  `GET /health` returns `{"status":"UP"}`.
- A preflight from `https://giwa-ui.vercel.app` to `/auth/login` returns the correct
  CORS allow-origin header.
- The deployed frontend previously appended `/auth/login` to a slash-terminated
  `VITE_API_URL`, producing `//auth/login`; Railway returned 400 without CORS headers,
  so the browser surfaced it as a CORS failure.
- The frontend now strips trailing slashes from `VITE_API_URL`, and the production
  environment value is stored without a trailing slash.
- Vercel has been redeployed, and browser API calls work without CORS errors.
- The public-demo metadata, Vercel SPA rewrite, 404, footer, and loading/empty
  state update still requires a Vercel redeployment.
- Railway RPC verification requires `GIWA_RPC_URL`, `GIWA_CHAIN_ID`,
  `GIWA_RECEIVABLE_FINANCE_ADDRESS`, and `GIWA_MOCK_KRW_ADDRESS`; timeout and
  confirmation depth are configurable with `GIWA_RPC_TIMEOUT_MS` and
  `GIWA_MIN_CONFIRMATIONS`.

Smart contract pre-submission validation

- `npm test` compiles Solidity 0.8.24 and passes 18 explicit Hardhat scenarios.
- Six Faucet scenarios cover constructor validation, fixed pre-funded transfer
  without supply growth, one claim per wallet, independent wallets, depleted
  inventory and refill recovery, and owner-only inventory withdrawal.
- The additional MockKRW scenario distinguishes initial-supply owner transfer
  from owner-only minting and checks the corresponding total-supply behavior.
- The complete CREATED→VERIFIED→TOKENIZED→FUNDED→REPAID lifecycle and all five
  lifecycle event argument sets are verified.
- Seller funding proceeds, NFT escrow/ownership transfer, repayment to the current
  NFT owner, and final REPAID status are verified.
- Role restrictions, wrong-state calls, and nonexistent IDs assert exact Solidity
  custom errors and arguments.
- Funding and repayment independently test insufficient ERC-20 balance and
  insufficient allowance, including state, balance, and NFT ownership rollback.
- Buyer, amount, and date input boundaries are verified.
- No change to the deployed `ReceivableFinance` or `MockKRW` core contracts was
  required; `MockKRWFaucet` is an additive testnet-only contract.

Remaining Features

- Update Railway/Vercel runtime address pairs and run a fresh lifecycle.
- Confirm the current NFT owner received the full face value on repayment.
- Redeploy Vercel with the Faucet-enabled Funding and Repayment bundle and its
  `VITE_MOCK_KRW_FAUCET_ADDRESS` value.
- Execute one real Buyer Faucet claim and confirm the claim, approval, and
  repayment remain three explicit MetaMask actions.
