# MOST IMPORTANT For Midnight work

## Midnight

초기 PoC에서 Spring Boot는 Attestation Provider가 아님
Mock Attestation API는 giwa-midnight/attestation-api에서 별도 실행

# Backend

## Stack

Spring Boot

Java17

MyBatis

JWT

MySQL

---

## Modules

auth

company

wallet

receivable

transaction

document

---

## Responsibility

Authentication

Database

REST API

Business Logic

Transaction History

---

## Common Error Handling

Implementation

- `common/error/ApiError.java`
- `common/error/ApiException.java`
- `common/error/GlobalExceptionHandler.java`
- `config/SecurityErrorHandler.java`

All API errors use this JSON shape.

- `status`: HTTP status number
- `code`: stable machine-readable error code
- `message`: user-safe message
- `path`: request path
- `timestamp`: error time
- `fieldErrors`: validation errors by field

Exception rules

- Use `ApiException` when frontend UX must branch on a stable error code.
- `ResponseStatusException` is converted to the common shape as a fallback.
- Bean Validation errors return `400 / VALIDATION_FAILED`.
- Invalid JSON returns `400 / INVALID_REQUEST_BODY`.
- Database constraint conflicts return `409 / DATA_CONFLICT`.
- Unexpected errors return `500 / INTERNAL_SERVER_ERROR` without exposing internal details.

Security rules

- Missing or invalid authentication returns `401 / AUTHENTICATION_REQUIRED`.
- Authenticated users without permission return `403 / ACCESS_DENIED`.
- `/error` and ERROR dispatcher requests are permitted so an original 4xx/5xx response is not replaced by Security 403.

Wallet conflict

- A wallet address remains globally unique across companies.
- Same wallet and same company is idempotent.
- Reconnecting the same company wallet with a different network refreshes its
  stored chain ID.
- Same wallet and different company returns `409 / WALLET_ALREADY_MAPPED`.
- Do not expose the owning company name, company ID, or business number in the error response.
- Database unique-constraint races are also translated to `WALLET_ALREADY_MAPPED`.

Verification

- `WalletErrorResponseIntegrationTests` verifies duplicate wallet 409.
- The same test verifies missing authentication returns 401 instead of 403.

---

## Receivable Blockchain Synchronization

Endpoints

- `POST /receivables/{id}/chain-created`
- `POST /receivables/{id}/verified`
- `POST /receivables/{id}/tokenized`
- `GET /receivables/funding-opportunities`
- `POST /receivables/{id}/funded`
- `POST /receivables/{id}/repaid`

Rules

- Seller company authorization is checked before chain-created metadata is saved.
- Buyer company authorization is checked before verification is saved.
- MyBatis UPDATE statements repeat the company, status, and required metadata conditions.
- `chain-created` stores onchain ID, contract address, create tx hash, and updated user.
- `verified` stores verify tx hash, changes status to VERIFIED, and inserts one status-history row.
- `tokenized` stores the RPC-verified token ID and tokenize tx hash, changes
  VERIFIED to TOKENIZED, and inserts one status-history row.
- Funding opportunities include only unassigned TOKENIZED receivables for a
  company that is neither Seller nor Buyer.
- `funded` accepts only `txHash`, derives the current Funder company and primary
  wallet, stores the configured MockKRW address and funding tx hash, changes
  TOKENIZED to FUNDED, and inserts one status-history row.
- `repaid` accepts only `txHash`, requires the authenticated Buyer and its
  registered receivable wallet, stores the repayment tx hash, changes FUNDED to
  REPAID, and inserts one status-history row.
- Reads used for transition decisions bypass the MyBatis local cache so concurrent requests see the latest state.
- Same blockchain metadata is idempotent, including after a later state transition.
- Different blockchain metadata uses `409 / BLOCKCHAIN_METADATA_CONFLICT`.
- Database unique constraints reject same-stage reuse of the same contract/onchain
  ID, create transaction hash, or verify transaction hash.
- Service-level duplicate checks provide a clear conflict before the write;
  database constraints remain the final defense against concurrent requests.
- The `blockchain_transactions` journal provides race-safe global transaction hash
  uniqueness.
- New chain-created, verified, tokenized, funded, and repaid state changes require a
  matching CONFIRMED journal entry for the receivable, company, wallet, type,
  contract, transaction hash, and emitted receivable ID.
- The tokenized request accepts only `txHash`; `token_id` comes from the journal's
  RPC-verified `event_token_id`, never from client input.
- The funded request accepts only `txHash`; Funder identity, MockKRW address, and
  token ID come from authenticated/configured/RPC-verified server state.
- The repaid request accepts only `txHash`; Buyer identity and the token ID come
  from authenticated/RPC-verified server state. The repayment recipient comes
  from the onchain event and matching MockKRW Transfer, not the stored Funder.
- Existing already-synchronized metadata remains idempotent even when it predates
  the journal.

RPC verification

- Require `GIWA_RPC_URL`, `GIWA_CHAIN_ID`,
  `GIWA_RECEIVABLE_FINANCE_ADDRESS`, and Funding/Repayment verification's
  `GIWA_MOCK_KRW_ADDRESS`.
- Fetch the chain, transaction, receipt, latest block, and receipt block directly
  from RPC.
- Require a successful receipt in the canonical block with at least
  `GIWA_MIN_CONFIRMATIONS`.
- Match transaction/receipt hashes, block hashes, signer, target, zero native
  value, chain, ABI selector, and every expected calldata argument.
- Decode only logs emitted by the configured contract and require exactly one
  expected lifecycle event.
- CREATE compares Seller, Buyer, face value, funding amount, dates, and document
  hash with DB data. VERIFY and TOKENIZE bind the stored onchain receivable ID and
  expected actor; TOKENIZE also checks the ERC-721 mint Transfer.
- FUND binds the Funder signer, fundReceivable calldata, ReceivableFunded event,
  stored token ID, Seller and funding amount, the MockKRW Funder-to-Seller
  Transfer, and the ReceivableFinance escrow-to-Funder ERC-721 Transfer.
- REPAY binds the Buyer signer, repayReceivable calldata, stored token ID,
  ReceivableRepaid Buyer/recipient/face value, and the MockKRW Buyer-to-recipient
  Transfer. The recipient is the current NFT owner and is intentionally not
  compared with the original Funder wallet.
- Persist RPC-derived chain, block hash/number, gas values, emitted IDs, and
  verification time. Client receipt values never overwrite these fields.
- Verify and backfill legacy CONFIRMED rows that have no `rpc_verified_at` before
  allowing a new receivable lifecycle synchronization.
- Re-run the full proof verification immediately before the first lifecycle state
  write even when a row already has `rpc_verified_at`; refresh a valid proof if a
  transaction was canonically re-included in a different block.
- Increment `verification_version` with every authoritative success/failure write
  and require the caller's previously read version in the UPDATE. Concurrent stale
  results fail CAS and must retry.
- The successful lifecycle-boundary proof refresh holds the journal row lock until
  the receivable write completes. Terminal updates also require that the same hash
  has not already been written to the matching receivable lifecycle column.
- Treat incoherent transaction/receipt snapshots, noncanonical blocks, and
  insufficient confirmations as retryable. Record a deterministic terminal
  failure only after a coherent canonical proof reaches the configured depth.

Not implemented yet

- Blockchain indexer

Verification

- `ReceivableOnchainServiceIntegrationTests` covers chain creation, Buyer
  verification, Seller tokenization, third-party Funding, Buyer Repayment,
  authorization, required prior lifecycle state, idempotent retry, conflicting
  metadata, duplicate history prevention, RPC-authoritative token IDs,
  RPC-verified journal requirements, atomic status guards, and legacy CONFIRMED
  backfill.

---

## Blockchain Transaction Journal

Endpoints

- `POST /blockchain-transactions`
- `PATCH /blockchain-transactions/{txHash}/confirmed`
- `PATCH /blockchain-transactions/{txHash}/failed`
- `GET /receivables/{id}/transactions`

Rules

- Supported types are CREATE_RECEIVABLE, VERIFY_RECEIVABLE,
  TOKENIZE_RECEIVABLE, FUND_RECEIVABLE, and REPAY_RECEIVABLE.
- The client supplies only receivable ID, type, contract address, and transaction
  hash when submitting.
- Company, stored receivable wallet, chain ID, function name, and initial PENDING
  status are derived by the backend.
- The initial chain ID is copied from the stored company-wallet mapping and is
  replaced by the configured RPC network chain during confirmation.
- The stored receivable wallet must still map to the authenticated company.
- CREATE requires the Seller and CREATED status.
- VERIFY requires the Buyer, CREATED status, and complete Seller chain metadata.
- TOKENIZE requires the Seller, VERIFIED status, and completed Buyer verification
  metadata.
- FUND requires an unrelated third-party company, TOKENIZED status, complete
  tokenization metadata, and that company's connected primary wallet.
- REPAY requires the Buyer company, FUNDED status, complete Funding metadata,
  and the registered Buyer wallet.
- Transaction hash uniqueness is enforced globally by the database.
- Identical create, confirm, and fail retries are idempotent.
- Conflicting metadata or terminal-state changes return stable 409 errors.
- Only the submitting company can update a journal row.
- Seller, Buyer, and assigned Funder can list the full receivable journal. An
  unassigned funding candidate can list only its own FUND_RECEIVABLE rows.
- Client receipt block/gas values enter the API as decimal strings for JavaScript
  precision safety, but are advisory; stored confirmation metadata comes from RPC.
- A timeout or temporary API/network error does not mark a transaction FAILED.
- Missing receipts, insufficient confirmations, and possible reorgs also remain
  retryable. Reverted receipts and deterministic signer/target/calldata/event
  mismatches become FAILED.
- Concurrent proof changes return `BLOCKCHAIN_VERIFICATION_RETRY_REQUIRED`.
- A valid proof whose emitted ID differs only from the synchronization request
  returns `BLOCKCHAIN_SYNCHRONIZATION_EVENT_MISMATCH` without failing the journal.
- A successful replacement is stored under its actual hash, while the original is
  marked FAILED with `TRANSACTION_REPLACED`.

Verification

- `BlockchainTransactionServiceIntegrationTests` covers server-derived metadata,
  roles/states/contracts, idempotency, conflicts, receipt parsing, company
  authorization, list visibility/order, RPC proof persistence, terminal/retryable
  verification failures, lifecycle-boundary revalidation, legacy terminal cleanup,
  CAS version ordering, post-synchronization failure prevention, and JSON string
  serialization.
- `BlockchainTransactionVerifierTests` cover CREATE/VERIFY/TOKENIZE/FUND/REPAY
  ABI and event validation, Funding payment/NFT Transfers, Repayment payment to
  the event recipient, chain/canonical/confirmation checks, signer/target
  mismatches, reverted receipts, duplicate events, and safe configuration/RPC
  failures.
- `GiwaJsonRpcClientTests` use a local HTTP JSON-RPC server to verify proof parsing,
  pending null results, and sanitized upstream errors.

---

## Railway Deployment Runtime

- `Dockerfile` uses a Java 17 JDK builder and Java 17 JRE runtime.
- Gradle packages the Spring Boot executable as `build/libs/app.jar`.
- The container runs as a non-root `app` user.
- `server.port` reads Railway `PORT`, then local `SERVER_PORT`, then `8080`.
- The server binds to `0.0.0.0`.
- Datasource configuration accepts the existing `DB_*` variables and Railway MySQL
  `MYSQLHOST`, `MYSQLPORT`, `MYSQLDATABASE`, `MYSQLUSER`, and `MYSQLPASSWORD`.
- `CORS_ALLOWED_ORIGINS` is a comma-separated list of exact frontend origins.
- `GET /health` is public and returns `{"status":"UP"}` for Railway health checks.
- Production must set a strong `JWT_SECRET`; the local fallback must not be used.
