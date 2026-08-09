# Frontend

## Pages

Login

Signup

Dashboard

Wallet

Receivables

Funding

Repayment

My Information

---

## Web3

ethers.js v6

BrowserProvider

Signer

MetaMask

---

## Stores

AuthStore

WalletStore

ReceivableStore

---

## Rules

State-changing transactions are signed only by MetaMask.

Frontend never stores private keys.

Visual consistency

- Preserve the existing light green product theme and route/component structure.
- `main.js` imports `assets/main.css`, which applies the project baseline from
  `base.css`. The baseline is light-only and limited to box sizing, body/app
  dimensions, typography, background, and native font inheritance.
- Do not restore the Vue starter automatic dark theme or universal
  `font-weight: normal`/margin reset; active screens use explicit scoped spacing
  and semantic type hierarchy.
- Authenticated workflow pages align to a maximum 1180px content width.
- Primary, secondary, list-row, and explorer actions keep distinct hover,
  disabled, and focus-visible states.
- Inputs and textareas keep the existing shape and receive consistent hover and
  green focus-ring feedback.
- Panels use the same subtle border, 16px-scale radius, and restrained shadow.
- Responsive rules preserve the current flows while reducing padding and
  wrapping actions at narrow widths.

Public demo release quality

- `index.html` defines the Korean document language, product title and
  description, browser theme color, canonical production URL, Open Graph
  metadata, and Twitter Card metadata.
- Public assets include an optimized favicon, Apple touch icon, dedicated
  1200x630 GIWA social card, `robots.txt`, and a root-only `sitemap.xml`.
- Authenticated pages and the client-side not-found route are marked
  `noindex, nofollow` at runtime.
- Router metadata sets a distinct browser title for Login, Dashboard,
  Receivables, Funding, Repayment, My Information, and 404.
- Unknown routes render the branded 404 page. `vercel.json` rewrites history-mode
  paths to `index.html` so direct links reach Vue Router.
- The shared shell adds only a compact demo footer and keyboard skip link; the
  existing header, route layout, and business flows remain unchanged.
- Data screens render in the order loading, error, data, and confirmed empty
  state. Empty copy is never shown before the initial API request completes.

Authenticated layout

- `App.vue` shows a compact navigation header on every route with
  `meta.requiresAuth`.
- The header links to Dashboard, Receivables, Funding, Repayment, and My
  Information.
- Display only the authenticated email in the global account summary. Do not
  expose internal user/company IDs, business number, or wallet address there.
- Direct refresh of an authenticated page loads `/auth/me` before replacing the
  loading label, and concurrent layout/page loads reuse the same request.
- Long emails use ellipsis. On mobile, the account summary remains on the first
  row and the menu is horizontally scrollable on the second row.

My Information

- `/profile` requires authentication and reuses AuthStore and WalletStore.
- Show the login email, company wallet connection state, and full wallet address
  only. Current APIs do not expose a user name or company name, so the frontend
  must not infer them.
- Treat `/wallet/me` 404 as a normal disconnected state. Keep other wallet lookup
  errors visible with a retry action.
- Provide wallet-address copy, Dashboard wallet management, and logout actions.
- Logout clears both AuthStore and WalletStore before routing to Login.

Contract configuration

- `src/contracts/addresses.js` reads only Vite environment variables.
- `src/contracts/ReceivableFinance.abi.json` contains the receivable lifecycle ABI.
- `src/contracts/MockKRW.abi.json` contains the payment-token ABI used by Funding
  and Repayment.
- `src/contracts/MockKRWFaucet.abi.json` contains only the read and claim surface
  used by the optional testnet demo-token flow.
- `src/composables/useMockKrwFaucetClaim.js` owns the shared wallet-keyed pending
  claim persistence, receipt/nonce recovery, uncertainty lock, and Faucet state
  used by both Funding and Repayment.
- Missing chain ID, RPC URL, ReceivableFinance address, or MockKRW address fails
  with a user-safe configuration message.
- A missing or invalid `VITE_MOCK_KRW_FAUCET_ADDRESS` disables only the demo
  recharge callout. It must never block a Funding or Repayment wallet that already
  has enough mKRW from its normal approval and lifecycle action.
- Placeholder or zero contract addresses are never treated as deployed contracts.
- Vercel must set `VITE_API_URL` to the Railway backend public HTTPS origin and
  rebuild the frontend because Vite variables are embedded at build time.
- `api.js` strips trailing slashes from `VITE_API_URL`, so API paths never become
  double-slash URLs that fail Railway preflight handling.

Receivable chain creation

1. Register the receivable in the backend.
2. Seller explicitly clicks GIWA chain creation.
3. Validate the GIWA network and active MetaMask signer.
4. Call `createReceivable`.
5. Wait for a successful receipt.
6. Parse `ReceivableCreated` by event name and keep the uint256 ID as a string.
7. Call `POST /receivables/{id}/chain-created`.

Buyer verification

1. Automatically prioritize a Buyer-owned CREATED receivable, then fall back to the
   first visible receivable, so pending Buyer work is visible.
2. Show the review panel for every authenticated Buyer-owned CREATED receivable.
3. Before Seller chain creation, keep the verification CTA visible but disabled and
   explain that the Buyer can review the terms while waiting.
4. Provide a state refresh action so the Buyer can load Seller chain creation without
   leaving the page; reset the local attestation checkbox whenever data is refreshed.
5. Show Seller, Buyer, registered wallets, amounts, issue date, maturity date,
   document hash, and description.
6. Require an explicit checkbox confirming that the Buyer reviewed and accepts the
   displayed debt terms.
7. Require that the Seller chain-created metadata exists.
8. Validate the GIWA network and Buyer MetaMask signer.
9. Read `getReceivable` and compare its ID, parties, amounts, dates, document hash,
   and CREATED status with the backend response.
10. Call `verifyReceivable`.
11. Wait for a successful receipt.
12. Call `POST /receivables/{id}/verified`.

The review checkbox is not persisted as a separate Web2 approval. The Buyer
MetaMask transaction is the authoritative attestation, and the smart contract still
prevents tokenization until its status is VERIFIED.
Client-side comparison protects the signing UX but is not a substitute for the
backend RPC receipt and event verification that authorizes synchronization.

Seller tokenization

1. Show the tokenization CTA only to the authenticated Seller for a VERIFIED
   receivable with complete create and verify metadata.
2. Validate the configured GIWA network, stored contract address, and registered
   Seller MetaMask signer.
3. Read `getReceivable` and compare the parties, amounts, dates, document hash,
   onchain ID, and VERIFIED status with the backend response.
4. Call `tokenizeReceivable`.
5. Persist the submitted hash and replacement recovery metadata before waiting.
6. Parse `ReceivableTokenized` from the configured contract, preserve token ID as
   a string, and require the financing contract as custodian.
7. Confirm the transaction through the backend RPC-verified journal.
8. Call `POST /receivables/{id}/tokenized` with only `txHash`.
9. Reload the receivable as TOKENIZED and display its token ID and tokenize tx.

The NFT is minted to the ReceivableFinance contract as escrow, not directly to
the Seller wallet.

Funder funding

1. Dashboard links to the authenticated `/funding` marketplace.
2. Load `GET /receivables/funding-opportunities`; Seller and Buyer companies never
   receive their own receivables as candidates.
3. Display Seller, Buyer, face value, funding amount, maturity, document hash,
   token ID, and expected face-value difference.
4. Validate the configured RPC network and require
   `ReceivableFinance.paymentToken()` to equal `VITE_MOCK_KRW_ADDRESS`.
5. Require MockKRW decimals 0, DB/onchain terms equality, TOKENIZED status, the
   stored token ID, zero onchain Funder, and NFT ownership by ReceivableFinance
   escrow.
6. Display the registered Funder wallet's mKRW balance and allowance. Block both
   writes when balance is below `fundingAmount`.
7. When balance is insufficient, read the separately configured MockKRWFaucet.
   Validate deployed bytecode, its MockKRW link, zero-decimal token model, fixed
   claim amount, wallet eligibility, Faucet inventory, and Funder native gas.
   Show the recharge action only when one claim makes the wallet balance at least
   `fundingAmount`.
8. The registered Funder wallet explicitly calls `claim()` through MetaMask.
   Wait for a successful receipt and require exactly one matching `Claimed` plus
   one MockKRW `Transfer(Faucet, Funder, claimAmount)`, then refresh the existing
   Funding readiness. Never submit a second claim when receipt confirmation is
   uncertain.
9. When allowance is insufficient, expose only the first-step approval action.
   Approve exactly `fundingAmount`, wait for a successful receipt, verify the
   exact Approval event, and re-read allowance.
10. Never automatically open `fundReceivable` after approval. The user must review
   the second step and click it separately.
11. Immediately before funding, repeat the complete onchain, balance, and allowance
   preflight, then call `fundReceivable`.
12. Verify exactly one ReceivableFunded event, MockKRW Transfer from Funder to
    Seller, NFT Transfer from escrow to Funder, FUNDED post-state, and Funder NFT
    ownership.
13. Confirm FUND_RECEIVABLE through the backend journal, then call
    `POST /receivables/{id}/funded` with only `txHash`.

Approval is not stored as a receivable lifecycle journal entry. A reload reads the
current allowance as the approval authority. FUND_RECEIVABLE uses the shared
per-company hash/replacement recovery record and backend journal. A confirmed
funding whose DB write failed exposes only a backend synchronization retry; it
never sends another fundReceivable transaction.

The demo recharge is testnet-only token distribution, not a receivable lifecycle
transition. It transfers pre-funded MockKRW inventory without minting, has no
backend API or DB journal row, and does not give the frontend/backend the Owner
private key. Funding and Repayment use the same per-wallet pending claim record,
so an uncertain request cannot be resubmitted from the other page. The caller
still needs GIWA Sepolia ETH for claim, approve, and fund or repay.

Buyer repayment

1. Dashboard and receivable management link to the authenticated `/repayment`
   page.
2. Load the current company's receivables and show only FUNDED receivables for
   which the authenticated company is the Buyer.
3. Display the full face value, maturity date, registered Buyer wallet, current
   NFT owner, mKRW balance, and allowance. The current NFT owner is the actual
   repayment recipient and may differ from the original Funder.
4. Validate the configured network, ReceivableFinance and MockKRW addresses,
   `paymentToken()`, MockKRW decimals 0, complete Funding metadata, DB/onchain
   terms, FUNDED status, token ID, stored onchain Funder, and current NFT owner.
5. When the Buyer balance is below `faceValue`, validate the separately configured
   MockKRWFaucet, its MockKRW link, zero decimals, fixed claim, wallet eligibility,
   inventory, and native gas. Expose the claim only when the current balance plus
   one fixed claim covers the full face value.
6. Submit `claim()` only from the registered Buyer MetaMask wallet. Persist the
   hash and nonce immediately, verify exactly one `Claimed` and one
   `Transfer(Faucet, Buyer, claimAmount)`, then refresh repayment readiness.
   Never automatically approve or repay after the claim.
7. When allowance is insufficient, expose only the first-step approval action.
   Approve exactly `faceValue`, wait for a successful receipt, verify the exact
   Approval event, and re-read allowance.
8. Never automatically open `repayReceivable` after approval. The Buyer must
   review the second step and click it separately.
9. Immediately before repayment, repeat the complete onchain, balance, allowance,
   and current-owner preflight, then call `repayReceivable`.
10. Verify exactly one ReceivableRepaid event and one MockKRW Transfer from Buyer
   to the receipt-block current NFT owner for `faceValue`.
11. Read the receipt block and require onchain status REPAID while NFT ownership
   remains unchanged.
12. Confirm REPAY_RECEIVABLE through the backend journal, then call
    `POST /receivables/{id}/repaid` with only `txHash`.

Approval is not journaled. REPAY_RECEIVABLE uses the shared per-company
localStorage and PENDING/CONFIRMED/FAILED journal recovery. A CONFIRMED repayment
takes priority over later PENDING or FAILED attempts. If the onchain transaction
succeeds but DB synchronization fails, expose only a backend synchronization
retry and never send another MetaMask repayment.
If receivable management detects a Funding or Repayment recovery record, it
routes to the matching dedicated page instead of trying to synchronize an
unsupported transaction type itself.

Repayment selection uses the same request-generation guard as Funding. Detail,
journal, Web3 readiness, and Faucet responses for an older selection are ignored,
and receivable switching is disabled while a wallet action is running.

The contract does not enforce repayment at or after `maturityDate`. The frontend
displays maturity for review but does not introduce a client-only date gate.

Transaction UX

- Display MetaMask/block confirmation and backend synchronization as separate stages.
- Before showing or executing the Seller mint action for a DB `VERIFIED`
  receivable, reconcile TOKENIZE transactions from the server journal. This
  protects a different browser, a cleared localStorage, and a stale tab.
- Prefer any CONFIRMED TOKENIZE journal over PENDING or FAILED rows even when the
  confirmed row is not the newest submission. A later failed attempt must not
  cause a duplicate mint.
- A CONFIRMED TOKENIZE journal with DB still `VERIFIED` shows an explicit
  "onchain mint complete, server synchronization required" warning and a
  backend-only synchronization button. The button never opens MetaMask.
- If that backend-only synchronization encounters a changed or unconfirmed
  journal, retry re-runs the journal reconciliation first. It does not reuse the
  same DB-sync label for a MetaMask receipt-recovery action.
- A PENDING TOKENIZE journal disables minting and restores the existing
  transaction-confirmation recovery flow. If local replacement metadata no
  longer exists, never guess or submit a replacement transaction.
- Within PENDING recovery, keep a browser-submitted hash and its replacement
  metadata even when that hash is not yet visible in the server journal. Do not
  overwrite it with a different server PENDING hash; reconcile that additional
  row after the local transaction is resolved.
- If the journal cannot be checked, keep minting disabled and show a retry action.
  Only a state with no CONFIRMED/PENDING tokenization candidate may expose a new
  mint action.
- This gate protects transactions whose hashes have reached the browser or server
  journal. It is not an atomic cross-browser lock before submission; the current
  contract still rejects a simultaneous second tokenization, while preventing its
  failed gas cost would require a future server-side intent lease.
- Handle user rejection, missing MetaMask, wrong network, wallet mismatch,
  an already-pending wallet request, insufficient gas, contract revert, and missing
  contract configuration.
- Store txHash in per-company browser local storage immediately after MetaMask
  submission, before waiting for a receipt.
- If the page reloads while pending, resume the existing receipt check and event
  parsing instead of sending the contract call again.
- If the chain transaction succeeds but backend synchronization fails, retain the
  confirmed txHash and event data until the idempotent backend retry succeeds.
- The retry button calls only the backend; it never submits the contract transaction again.
- Explorer links are displayed only when `VITE_GIWA_EXPLORER_URL` is configured.
- Receivable details link the contract address and create, verify, tokenize,
  funding, and repayment transaction hashes to their matching explorer pages
  while retaining the full values on screen.
- Immediately after MetaMask returns a hash, keep it in per-company local storage
  and create a backend PENDING journal entry before waiting for a receipt.
- After a successful browser receipt, the frontend sends decimal-string block/gas
  hints. The backend independently verifies RPC transaction/receipt/block/calldata/
  events and returns authoritative CONFIRMED metadata before receivable state
  synchronization.
- RPC pending, confirmation-depth, reorg, and availability errors keep the local
  transaction for retry without sending another contract call.
- A backend-verified revert or deterministic transaction/event mismatch is
  terminal. Clear the invalid local recovery record and show the stable backend
  error instead of retrying synchronization forever.
- If a terminal backend response was lost, the idempotent PENDING POST returns the
  existing FAILED journal on recovery; convert it to
  `BLOCKCHAIN_TRANSACTION_FAILED` and clear the stale local record.
- `BLOCKCHAIN_VERIFICATION_RETRY_REQUIRED` and the RPC pending/reorg/configuration
  families keep the local record for a backend-only retry.
- `BLOCKCHAIN_SYNCHRONIZATION_EVENT_MISMATCH` is distinct from a verifier event
  failure. Move the local record back to receipt recovery so the next retry
  reparses the event instead of treating a bad payload as a failed onchain
  transaction.
- A hash is marked FAILED only for a mined failed receipt or a provider-confirmed
  replacement/cancellation.
- Receipt timeout, temporary network failure, and journal API failure leave the
  local transaction recoverable and do not resubmit the contract call.
- On a successful ethers replacement, store the replacement hash first, journal it
  as PENDING→CONFIRMED, and mark the original hash FAILED with
  `TRANSACTION_REPLACED`.
- Capture a replacement scan start block before each contract submission and
  persist the public transaction `from`, nonce, target, calldata, and value with
  the original hash.
- After reload, use ethers replacement detection while the original transaction
  remains RPC-readable.
- If the RPC no longer returns the original hash, scan bounded canonical block
  ranges with a persisted cursor. New records require the same sender and nonce;
  legacy hash-only records require one exact actor, contract, calldata, and
  zero-value match.
- Only an exact-intent repricing is resumed. A same-nonce changed call or
  cancellation is terminal, while no or ambiguous candidates stay retryable and
  never trigger a new contract submission.
- Reload recovery idempotently re-ensures the PENDING row, checks the existing
  receipt, updates the journal, and then resumes backend state synchronization.
- Browser records created before journal integration are backfilled on retry. If
  their exact create/verify metadata is already present in the backend, they are
  recognized as already synchronized and safely cleared.
- A cancelled replacement is kept in local storage when its FAILED journal update
  cannot be sent; retry completes that update before clearing the local record.
- Known limitation: a legacy hash-only record can search only the retained bounded
  block history. If the relevant block is no longer available from the RPC, safe
  automatic replacement discovery remains retryable and requires explorer review.

Business Number

- Input and display format: 123-45-67890
- API payload format: 1234567890

Wallet Connection

- Force MetaMask account selection before mapping
- Show the selected address before calling POST /wallet/connect
- Require explicit user confirmation
- On WALLET_ALREADY_MAPPED, explain the conflict and offer another account selection

API Error Handling

- `src/services/api.js` is the only backend API fetch layer.
- AuthStore, WalletStore, and ReceivableStore use `apiRequest`.
- `ApiError` exposes `status`, `code`, `message`, and `fieldErrors`.
- Display the backend user-safe message.
- Branch special UX by stable `code`, not by parsing message text.
- Network failures use `NETWORK_ERROR`.

MetaMask Connection Flow

1. Call `wallet_requestPermissions` to show MetaMask account selection.
2. Read the selected account and chain.
3. Display the selected address before saving.
4. Require explicit confirmation.
5. Call `POST /wallet/connect`.
6. On `WALLET_ALREADY_MAPPED`, show that the address belongs to another company and offer `다른 계정 선택`.

Wallet UX rules

- Never automatically save the previously authorized browser account.
- Keep the selected address pending until the user confirms.
- On duplicate mapping, keep the pending address visible so the user understands which address failed.
- Do not display any information about the company that already owns the address.
- MetaMask rejection code `4001` is displayed as a cancellation message.
- Contract writes never assume that MetaMask account index 0 is the registered
  company wallet.
- Request the receivable wallet explicitly with `getSigner(expectedAddress)`.
- When the expected wallet is not in the site's permitted accounts, open the
  MetaMask account permission picker once and verify the returned accounts again.
- If it is still unavailable, show both the receivable's expected address and the
  first permitted address so the user can select the correct account or identify a
  stale wallet mapping.
