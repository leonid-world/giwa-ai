# Database

## Tables

companies

기업 정보

users

로그인 계정

company_wallets

기업 ↔ Wallet Mapping

receivables

매출채권

receivable_documents

증빙 문서

blockchain_transactions

온체인 Transaction

receivable_status_history

상태 변경 이력

---

## Design Rules

- Wallet Address만 저장
- Private Key 저장 금지
- Business Number = CHAR(10), 숫자 10자리 문자열
- Amount = BigDecimal
- Current schema onchain_receivable_id and token_id = BIGINT UNSIGNED
- Java currently maps sequential onchain IDs to Long and serializes them as JSON strings
- Arbitrary uint256 IDs would require a future schema/Java migration before use
- TxHash 저장
- `(contract_address, onchain_receivable_id)` is unique
- create_tx_hash and verify_tx_hash are unique
- Seller tokenization stores `token_id` and `tokenize_tx_hash` only after the
  backend verifies the TOKENIZE receipt and event through RPC.
- The client never supplies the authoritative token ID.
- Funding uses the existing `funder_company_id`, `funder_wallet_address`,
  `mock_token_address`, and `funding_tx_hash` columns; no new Funding migration is
  required.
- Those four fields and status FUNDED are written atomically only after the
  backend verifies FUND_RECEIVABLE, ReceivableFunded, the MockKRW payment
  Transfer, and the escrow NFT Transfer through RPC.
- Repayment reuses the existing `repay_tx_hash` column and status value REPAID;
  no new Repayment migration is required.
- Status REPAID, `repay_tx_hash`, and the Buyer status-history row are written
  atomically only after the backend verifies REPAY_RECEIVABLE,
  ReceivableRepaid, and the MockKRW Buyer-to-recipient face-value Transfer.
- The repayment recipient is the current NFT owner recorded by the onchain event.
  No dedicated recipient column is required. `funder_wallet_address` remains the
  historical original Funder and is never used as the current repayment recipient.
- `blockchain_transactions.tx_hash` is globally unique across lifecycle stages.
- Journal status is `PENDING`, `CONFIRMED`, or `FAILED`.
- Receipt block/gas values are stored only after backend RPC confirmation.
- PENDING `chain_id` starts as the stored company-wallet snapshot. CONFIRMED
  `chain_id` is replaced by the RPC network value.
- `block_hash`, `event_receivable_id`, `event_token_id`, and `rpc_verified_at`
  preserve the backend verification proof summary.
- `verification_version` starts at zero and increments atomically for each
  authoritative proof or terminal verification write.

## Blockchain Transaction Journal

The canonical table is already present in `.codex/schema.sql`.

For an existing database, run
`.codex/migrations/20260730_blockchain_transactions.sql`. It only creates the
table when absent and does not drop or rewrite existing data.

If the table already existed before RPC verification and has none of the four RPC
proof columns, first run the preflight and then the one-time ALTER in
`.codex/migrations/20260730_blockchain_transaction_rpc_verification.sql`. The
application queries the new proof columns, so this migration must finish before
deploying the updated backend.

If `verification_version` is absent, run the preflight and one-time ALTER in
`.codex/migrations/20260730_blockchain_transaction_verification_version.sql`.
This second migration also covers a database where the four RPC proof columns were
already applied during an earlier rollout.

Do not run either ALTER after the current
`.codex/migrations/20260730_blockchain_transactions.sql`; the create-table
migration already includes all five columns. Do not rerun an ALTER whose preflight
column is already present.

The H2 integration-test schema mirrors the journal columns, foreign keys, and
global transaction-hash uniqueness, including the RPC proof fields.

## Existing MySQL Migration

`spring.sql.init.mode=never`, so editing `.codex/schema.sql` does not update an
already populated local database.

Do not rerun `.codex/schema.sql`; it contains destructive DROP statements.

For the current database:

1. Back up the database.
2. Run the duplicate preflight SELECT statements in
   `.codex/migrations/20260730_receivable_chain_metadata_uniques.sql`.
3. Continue only when all three queries return zero rows.
4. Run the ALTER TABLE statement in that file once.
