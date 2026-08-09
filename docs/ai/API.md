# REST API

## Auth

POST /auth/signup

POST /auth/login

GET /auth/me

---

## Health

GET /health

Public deployment health endpoint.

```json
{
  "status": "UP"
}
```

---

## Wallet

POST /wallet/connect

GET /wallet/me

---

## Receivable

Implemented

POST /receivables

GET /receivables

GET /receivables/funding-opportunities

GET /receivables/{id}

POST /receivables/{id}/chain-created

POST /receivables/{id}/verified

POST /receivables/{id}/tokenized

POST /receivables/{id}/funded

POST /receivables/{id}/repaid

---

## Receivable Chain Created

Seller-only synchronization after a confirmed `createReceivable` transaction.
This endpoint stores blockchain metadata and keeps the receivable status as `CREATED`.

```json
{
  "onchainReceivableId": "1",
  "txHash": "0x...",
  "contractAddress": "0x..."
}
```

## Receivable Verified

Buyer-only synchronization after a confirmed `verifyReceivable` transaction.
This endpoint changes `CREATED` to `VERIFIED` and adds status history.

```json
{
  "txHash": "0x..."
}
```

## Receivable Tokenized

Seller-only synchronization after a confirmed `tokenizeReceivable` transaction.
This endpoint changes `VERIFIED` to `TOKENIZED`, stores the RPC-verified NFT token
ID and transaction hash, and adds status history.

```json
{
  "txHash": "0x..."
}
```

The client does not submit `tokenId`. The backend uses only `eventTokenId` from
the matching RPC-verified `TOKENIZE_RECEIVABLE` journal proof.

## Funding Opportunities

Returns only unassigned `TOKENIZED` receivables for which the authenticated
company is neither Seller nor Buyer. A candidate may read that receivable detail,
but transaction-list access is limited to the candidate's own FUND_RECEIVABLE
journal rows until it becomes the assigned Funder.

## Receivable Funded

Third-party Funder synchronization after a confirmed `fundReceivable`
transaction. This endpoint changes `TOKENIZED` to `FUNDED`, stores the
authenticated Funder company and wallet, configured MockKRW address, funding
transaction hash, and status history.

```json
{
  "txHash": "0x..."
}
```

The client does not submit the Funder identity, payment-token address, or token
ID. The backend derives them from authentication, server configuration, and the
RPC-verified FUND_RECEIVABLE proof.

## Receivable Repaid

Buyer-only synchronization after a confirmed `repayReceivable` transaction. This
endpoint changes `FUNDED` to `REPAID`, stores the repayment transaction hash, and
adds status history.

```json
{
  "txHash": "0x..."
}
```

The client does not submit the repayment recipient, face value, or token ID. The
backend derives the Buyer from authentication and accepts only the
RPC-verified REPAY_RECEIVABLE proof. The verified `ReceivableRepaid` recipient
must equal the recipient of the MockKRW Transfer from Buyer for the full face
value. It is the current NFT owner and may differ from the original Funder.

Synchronization rules

- Same metadata retry is idempotent and returns the current receivable.
- Different metadata retry returns `409 / BLOCKCHAIN_METADATA_CONFLICT`.
- Verification before chain creation returns `409 / RECEIVABLE_NOT_ONCHAIN`.
- Tokenization before Buyer verification returns
  `409 / RECEIVABLE_NOT_VERIFIED_ONCHAIN`.
- Funding before tokenization returns
  `409 / RECEIVABLE_NOT_TOKENIZED_ONCHAIN`.
- Repayment before Funding returns
  `409 / RECEIVABLE_NOT_FUNDED_ONCHAIN`.
- A wrong state returns `409 / INVALID_RECEIVABLE_STATUS`.
- Seller-only and Buyer-only violations return `403 / ONLY_SELLER` or `403 / ONLY_BUYER`.
- Seller or Buyer Funding returns `403 / RELATED_PARTY_CANNOT_FUND`.
- Backend does not sign or submit blockchain transactions.

---

## Transaction

Implemented.

POST /blockchain-transactions

PATCH /blockchain-transactions/{txHash}/confirmed

PATCH /blockchain-transactions/{txHash}/failed

GET /receivables/{id}/transactions

Create a PENDING journal entry immediately after MetaMask returns a transaction
hash.

```json
{
  "receivableId": 1,
  "transactionType": "CREATE_RECEIVABLE",
  "contractAddress": "0x...",
  "txHash": "0x..."
}
```

Supported transaction types

- `CREATE_RECEIVABLE`
- `VERIFY_RECEIVABLE`
- `TOKENIZE_RECEIVABLE`
- `FUND_RECEIVABLE`
- `REPAY_RECEIVABLE`

The backend derives company, wallet, chain ID, and contract function from the
authenticated user, receivable, and transaction type.

While the row is PENDING, the returned chain ID is a provisional stored
wallet-mapping snapshot. Successful confirmation replaces it with the chain ID
read from the configured backend RPC network.

Request backend confirmation after the browser observes a successful receipt:

```json
{
  "blockNumber": "12345",
  "gasUsed": "21000",
  "effectiveGasPrice": "1000000000"
}
```

The three request values remain decimal strings and are validated for transport
compatibility, but they are advisory. The backend independently fetches and
validates the transaction, receipt, canonical block, confirmation depth, calldata,
and contract events. CONFIRMED stores only RPC-derived chain/block/gas/event proof
data.

Confirmed responses also include `blockHash`, `eventReceivableId`,
`eventTokenId` for TOKENIZE, FUND, and REPAY, `rpcVerifiedAt`, and
`verificationVersion`. Integer values are serialized as JSON strings.

Mark a definitively failed or replaced transaction:

```json
{
  "errorCode": "TRANSACTION_REPLACED",
  "errorMessage": "replacement transaction used"
}
```

Journal rules

- Transaction hashes and addresses are normalized to lowercase.
- The same submission and same status transition retry are idempotent.
- Reusing a transaction hash with different metadata returns
  `409 / BLOCKCHAIN_TRANSACTION_CONFLICT`.
- Only the submitting company can confirm or fail its transaction.
- Seller, Buyer, and assigned Funder can list all receivable transactions.
  An unassigned Funding candidate sees only its own FUND_RECEIVABLE rows.
- Transaction lists are ordered by `submittedAt` descending and then journal ID
  descending. Tokenization recovery must inspect every TOKENIZE row: an earlier
  CONFIRMED result takes priority over a later PENDING or FAILED attempt.
- The journal response does not contain browser-only replacement metadata such as
  nonce, calldata, scan cursor, or replacement links. Preserve an existing local
  recovery payload when its hash matches a PENDING journal row.
- Receipt integers are sent and returned as decimal strings.
- `chain-created`, `verified`, `tokenized`, `funded`, and `repaid`
  synchronization require a matching CONFIRMED journal entry with
  `rpcVerifiedAt` and the expected emitted receivable ID.
- REPAY_RECEIVABLE also requires the verified token ID to match the DB token ID.
- A legacy CONFIRMED row without `rpcVerifiedAt` is verified and backfilled through
  RPC before it can authorize a new synchronization.
- The backend revalidates even a previously verified CONFIRMED proof immediately
  before the first lifecycle write and refreshes it if the canonical placement
  changed.
- RPC/network/receipt/confirmation/reorg uncertainty leaves the row retryable.
- Only after a coherent canonical proof reaches the required confirmation depth
  does a reverted receipt or deterministic transaction/event mismatch mark the
  unsynchronized row FAILED.
- Success and failure writes compare `verificationVersion`; a stale concurrent
  result returns a retryable conflict instead of overwriting newer proof.
- A failure write checks the receivable lifecycle hash and cannot downgrade a
  transaction that is already synchronized.

Journal error codes

- `400 / INVALID_TRANSACTION_TYPE`
- `400 / INVALID_TRANSACTION_HASH`
- `400 / INVALID_RECEIPT_METADATA`
- `403 / ONLY_SELLER` or `403 / ONLY_BUYER`
- `403 / RELATED_PARTY_CANNOT_FUND`
- `404 / BLOCKCHAIN_TRANSACTION_NOT_FOUND`
- `409 / BLOCKCHAIN_TRANSACTION_CONFLICT`
- `409 / INVALID_BLOCKCHAIN_TRANSACTION_STATUS`
- `409 / BLOCKCHAIN_TRANSACTION_NOT_CONFIRMED`
- `409 / INVALID_RECEIVABLE_STATUS`
- `409 / CONTRACT_ADDRESS_MISMATCH`
- `409 / RECEIVABLE_WALLET_NOT_MAPPED`
- `409 / FUNDER_WALLET_NOT_CONNECTED`
- `409 / BLOCKCHAIN_TRANSACTION_PENDING`
- `409 / BLOCKCHAIN_CONFIRMATIONS_PENDING`
- `409 / BLOCKCHAIN_CANONICAL_BLOCK_PENDING`
- `409 / BLOCKCHAIN_REORG_DETECTED`
- `409 / BLOCKCHAIN_TRANSACTION_REVERTED`
- `409 / BLOCKCHAIN_TRANSACTION_VERIFICATION_FAILED`
- `409 / BLOCKCHAIN_EVENT_MISMATCH`
- `409 / BLOCKCHAIN_SYNCHRONIZATION_EVENT_MISMATCH`
- `409 / BLOCKCHAIN_VERIFICATION_RETRY_REQUIRED`
- `502 / BLOCKCHAIN_RPC_INVALID_RESPONSE`
- `503 / BLOCKCHAIN_RPC_NOT_CONFIGURED`
- `503 / BLOCKCHAIN_RPC_CONFIGURATION_MISMATCH`
- `503 / BLOCKCHAIN_RPC_UNAVAILABLE`

---

## Error Response

All API errors use the same JSON shape.

- status
- code
- message
- path
- timestamp
- fieldErrors

Wallet conflict

- HTTP 409
- code = WALLET_ALREADY_MAPPED

Authentication required

- HTTP 401
- code = AUTHENTICATION_REQUIRED

Access denied

- HTTP 403
- code = ACCESS_DENIED
