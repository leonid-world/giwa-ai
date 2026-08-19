# Deployment

## Current Midnight v2 Local Deployment

Midnight remains local-only on network `undeployed`; it is not deployed to
Preprod/Mainnet, Railway, or Vercel. The current v2 contract is:

```text
12caaf76aef1de1c584b67462018810f6e4e7eb2535e136f560cb621e24a3f36
```

Provider ID 2 was registered using the deterministic local development secret
literal `PROVIDER_SECRET_KEY=2`. Registration transaction
`006abe69d8ba934519e19c4490ce77be724f75aae1bcb4e6b4fcd720258aa10601`
was included at local block `25714`; the public registry contained one Provider
after verification. The old v1 contract is preserved. Do not remove/recreate
Docker volumes or redeploy simply because an Indexer read is temporarily
missing.

Use [the local runbook](../../midnight/LOCAL_POC_RUNBOOK.md) as the executable
source of truth. It starts five Midnight-facing terminals: Docker Node +
Indexer + Proof Server, deterministic Mock Provider 2, Read API, Proof Bridge,
and Vue. Spring Boot and MySQL are required in addition. The Bridge and CLI
share the Midnight wallet/private-state database and must not run concurrently.

Required secret/config boundaries:

- Node/CLI/Bridge runtime stays on Node `22.21.1`.
- `PROVIDER_SECRET_KEY=2` is a deterministic local test value only; never use it
  on a network or asset with value.
- `MIDNIGHT_STORAGE_PASSWORD` opens the encrypted Midnight private state and
  derives the encrypted capability-outbox key. Keep it in an ignored local env
  file and preserve it across Bridge restarts; losing/changing it makes the
  outbox and private state unavailable.
- Spring requires a separate externally supplied 32-byte
  `MIDNIGHT_CAPABILITY_ENCRYPTION_KEY`. It is not stored in MySQL or Git.
- Existing MySQL installs apply
  `.codex/migrations/20260819_midnight_proof_requests.sql` once after backup and
  only when the table is absent.

Use `http://127.0.0.1:5173` consistently for the browser. The Bridge accepts
both loopback names, but `localhost` and `127.0.0.1` are different browser
origins/cookie scopes; Spring CORS must include the exact origin used by Vue.
Switching names mid-session can look like an authentication/CORS failure.

The current checks are Compact contract `39/39`, Mock Provider `87/87`, Read
API `60/60`, CLI `156` passed plus `1` optional environment test skipped (12
passing files plus 1 skipped file), Vue `141/141` across 21 files, and Spring
full Gradle `86/86` including focused Midnight `19/19`. CLI typecheck/build,
Vue lint/build, Spring `bootJar`, and deployment/registration preflight passed.
A fresh end-to-end v2 browser run through MetaMask, Spring `SUBMITTED`, Bridge
ACK, Funder resolve, and `COMPLETED` has not yet been recorded after the final
outbox changes, so deployment readiness must not claim that live evidence yet.

## Current Status

- Target network: GIWA Sepolia.
- Local Hardhat compilation passes with Solidity `0.8.24+commit.e11b9ed9`,
  optimizer enabled with 200 runs, viaIR disabled, and EVM version Paris.
- The original Remix MockKRW and ReceivableFinance deployment completed the live
  lifecycle but its compiler settings were not retained for final verification.
- The replacement Hardhat deployment and Blockscout verification are complete.
- Verified MockKRW:
  `0x5cD8a99Dcf5Fa00fb4fD9873b41A15F9C13C9d3F`.
- Verified ReceivableFinance:
  `0x0f264334f98BA0d22f7Fc6Bb901a5Fa36158a315`.
- User-confirmed Seller createReceivable and Buyer verifyReceivable live transactions
  are complete.
- User-confirmed Seller tokenizeReceivable live transaction, escrow NFT mint, and
  backend TOKENIZED/tokenId/tokenizeTxHash synchronization are complete.
- User-confirmed Funder mKRW approval, fundReceivable, backend FUNDED
  synchronization, and Seller mKRW receipt are complete on GIWA Sepolia.
- User-confirmed Buyer mKRW approval, repayReceivable, backend RPC proof, and DB
  REPAID synchronization are complete. Exact current-owner face-value receipt
  remains to be confirmed.
- The Repayment backend/frontend update has not yet been redeployed to Railway
  and Vercel.
- The Vercel frontend is deployed.
- The Railway backend is deployed, and `GET /health` returns `{"status":"UP"}`.
- Browser API calls from `https://giwa-ui.vercel.app` are working without CORS
  errors after normalizing the frontend API base URL.
- Public replacement addresses, deployment transactions, blocks, deployer, and
  compiler settings are recorded in
  `giwa-contrract/deployment/giwa-testnet.json`.
- Hardhat owner operations can transfer existing mKRW or mint additional
  test-only mKRW to Buyer/Funder wallets without using Remix.
- MockKRWFaucet is deployed separately at
  `0xa451FA95c3E2Efd771f6Ba556daBBf36f888ef2E` with a fixed
  `10,000,000 mKRW` claim. Its deployment metadata leaves the verified
  MockKRW/ReceivableFinance pair untouched.
- The tracked frontend environments and Funding/Repayment UIs use the Faucet
  address. The Faucet now holds `600,000,000 mKRW`, while the Vercel runtime
  variable and frontend redeployment are still pending.
- Owner-to-Funder `10,000 mKRW` transfer
  `0x276dc7572aa09e47b2cc55e1b75ae4e111cfbe1cb45f89ef73985a4716032ba0`
  is confirmed. A temporary public-RPC latest-state lag caused the original CLI
  post-state check to report a false error even though the receipt and Transfer
  were valid; owner tasks now retry state reads at the confirmed block and warn
  without treating an already confirmed transfer as failed.

## Historical v1: Midnight Local-Only Proof Bridge

Midnight is not part of the Railway or Vercel deployment. Never add the Proof
Bridge, Mock Attestation API, Proof Server, local Node/Indexer, Midnight wallet
seed, private-state password, raw financial values, or
`VITE_MIDNIGHT_PROOF_BRIDGE_ENABLED=true` to a production environment. No
Midnight contract is deployed to Preprod or Mainnet.

The development stack uses these loopback-only processes:

- Midnight Node `127.0.0.1:9944`
- Midnight Indexer `127.0.0.1:8088`
- Midnight Proof Server `127.0.0.1:6300`
- Mock Attestation API `127.0.0.1:4000`
- read-only capability API `127.0.0.1:4100`
- trusted Proof Bridge `127.0.0.1:4200`
- Vue development server on strict `127.0.0.1:5173`, with same-origin
  `/midnight-api` and a proof-flag-conditional `/midnight-proof` proxy

Use `midnight/LOCAL_POC_RUNBOOK.md` for the five copy-paste terminals and actor
test sequence. The existing Spring API/MySQL must also be reachable so Vue can
load authenticated receivable context, but they are not new Midnight
processes. Seller/Buyer start issuance from Receivables and sign with the
derived role wallet; Funder uses `/midnight` only to verify a delivered
capability. No user manually types an onchain ID in the current Vue flow.

Capability copy/export/import needs no sixth service. It is an explicit local
browser/OS handoff: Seller/Buyer copy to the clipboard or export a local file,
and Funder imports from the clipboard or a selected file. The file is never
uploaded to Spring or a delivery server. It is correlation-sensitive and may
remain in clipboard history, backups, or synced folders, so use only the
intended Funder, avoid shared/auto-synced locations, clear or overwrite the
clipboard, and delete obsolete files. Raw JSON is for advanced diagnostics.

Use Node 22.21.1 for the current Midnight CLI/Bridge runtime. Start
Node/Indexer/Proof Server
through `giwa-midnight/cli/standalone.yml`, then the already-registered Provider
2 Mock Attestation API, the read API, and the Proof Bridge. Use Node 24.19.0 for
the current `giwa-ui` toolchain and start Vue last; some installed transitive UI
dependencies require Node `>=24.15` or `>=22.22.2`. The
Provider process key must match the Provider 2 public key registered in the
current local Compact deployment; restarting an ephemeral Provider with a new
key requires intentional re-registration before proof creation.

From the `giwa-midnight` workspace, the Bridge command is:

```bash
nvm use 22.21.1
npm run proof-bridge --workspace zkloan-credit-scorer-cli
```

From `giwa-ui`, use:

```bash
nvm use 24.19.0
npm run dev
```

The process performs its wallet/private-state checks plus a 10-second-bounded
Indexer contract/Provider preflight before opening port 4200, then seals the
validated GIWA configuration in memory. Per-challenge preparation uses that
cache instead of querying the Indexer while raw inputs exist. A missing existing
contract-scoped private state, mismatched running Provider 2 key, unavailable
local service, or already-held process lock must fail closed rather than create
a new participant or overwrite state. The SDK Indexer query has no abort signal,
so one timed-out startup query may remain internally pending, but the server
does not open and no raw proof input has been accepted. This public preflight
does not yet impose an end-to-end deadline on every subsequent SDK
`joinContract` watcher; the Bridge still never opens its HTTP port until join
finishes, and a full join deadline remains a tracked hardening item.

The Bridge deliberately uses the public, disposable Local Dev genesis wallet
seed already used by the standalone CLI flow. It is not a production secret and
must never be reused on Preprod, Mainnet, or for assets with value. Real wallet
seeds and private keys remain prohibited in source, Vue, Spring, logs, and chat.

Do not run the interactive CLI and Proof Bridge concurrently. They share one
encrypted LevelDB participant state and wallet and use a common fail-fast
process lock. Do not delete or recreate the local Node container when the
current disposable contract/results must remain available; the compose setup
does not yet provide an approved durable Node/Indexer recovery design.

The Bridge is a trusted custodial single-user development process. Keep all
ports on literal loopback, leave CORS disabled, and access it only through the
Vue dev proxy. Current official Local Dev also supports Lace on `undeployed`,
but ADR-018 intentionally reuses the proven CLI participant instead of creating
and migrating a second Lace identity/private state. This does not authorize a
remote Bridge deployment.

Only the ignored/local development Vue environment may enable the route:

```env
VITE_MIDNIGHT_POC_ENABLED=true
VITE_MIDNIGHT_PROOF_BRIDGE_ENABLED=true
VITE_MIDNIGHT_PROOF_API_URL=/midnight-proof
```

`VITE_MIDNIGHT_PROOF_API_URL` must remain a same-origin absolute path. Do not
replace it with `http://127.0.0.1:4200`; the Vite proxy supplies the same-origin
browser boundary and the Bridge intentionally enables no CORS. Production and
tracked production configuration must keep both feature flags false.

With the proof flag on, both the Vite devtools plugin and Vue runtime Devtools
exposure are disabled, strict port `5173` prevents Origin drift, and the
port-4200 proxy is present. With the flag off, that proxy is absent. If the
Bridge returns `complete` but the read resolver has not seen the result yet, the
Midnight transaction has already finalized: retry only public resolution. Do
not restart the proof session or submit another proof for an Indexer delay.

## Contract Deployment

GIWA Sepolia Hardhat configuration:

- Chain ID: `91342`
- RPC: `https://sepolia-rpc.giwa.io`
- Explorer: `https://sepolia-explorer.giwa.io`
- Blockscout API: `https://sepolia-explorer.giwa.io/api`
- Compiler: `0.8.24+commit.e11b9ed9`
- Optimizer: enabled, 200 runs
- viaIR: disabled
- EVM version: Paris

The public GIWA RPC is rate-limited. Override `GIWA_RPC_URL` in the current
terminal when a dedicated provider is available.

1. Back up the existing MySQL database and preserve the original deployment
   addresses and transaction history.
2. Install the locked toolchain with `npm ci` in `giwa-contrract`.
3. Confirm the deployer has GIWA Sepolia ETH.
4. Export the private key only for the current terminal session:

   ```bash
   read -s "DEPLOYER_PRIVATE_KEY?GIWA deployer private key: "
   export DEPLOYER_PRIVATE_KEY
   ```

5. Deploy and verify:

   ```bash
   npm run deploy:giwa:verify
   unset DEPLOYER_PRIVATE_KEY
   ```

6. If Blockscout indexing delays verification, do not redeploy. Run:

   ```bash
   npm run verify:giwa
   ```

7. Print the address variables:

   ```bash
   npm run deployment:env
   ```

The deploy script checks chain ID, deployer balance, deployment receipts, and
`ReceivableFinance.paymentToken()`. It records addresses, deployment hashes,
blocks, deployer address, time, and compiler settings in
`giwa-contrract/deployment/giwa-testnet.json`. A completed metadata file prevents
an accidental repeat; an intentional later replacement requires transient
`ALLOW_REDEPLOY=true` and archives the prior metadata.

Before verification, the verification script independently checks the live chain
ID, bytecode at both recorded addresses, compiler metadata, and the
`ReceivableFinance.paymentToken()` link. A partially completed deployment is
resumed only after the recorded MockKRW code, owner, and zero-decimal model are
validated.

During the replacement deployment, the Finance creation receipt succeeded but
the first public-RPC `paymentToken()` read returned empty data. Explorer and RPC
checks recovered the already-deployed Finance at
`0x0f264334f98BA0d22f7Fc6Bb901a5Fa36158a315`; it was not redeployed. The
deployment and verification scripts now retry code/state visibility, and complete
receipt metadata is written before the post-deployment state check.

Never store the deployer private key, seed phrase, or MetaMask password.
Never rerun the destructive `.codex/schema.sql` against a populated database.

### Replacement deployment boundary

The replacement pair is a new independent system:

- The new ReceivableFinance has empty receivable mappings, counters, and NFT
  ownership.
- The new MockKRW has separate balances; the initial supply belongs to the new
  deployer.
- Old mKRW balances, NFT ownership, allowances, transaction hashes, and DB
  journal proofs remain tied to the original pair.
- Never update an old DB row's contract/token metadata to the new addresses.
- Prefer a new empty demo database while preserving the completed one.
- Transfer the new deployer's initial MockKRW balance to Buyer and Funder, or mint
  additional test supply only when necessary. Import the new token in MetaMask
  and approve the new Finance address.
- Clear only the old per-company `receivablePendingBlockchainSync` browser record
  before the fresh demo when necessary.

### MockKRW demo distribution

MockKRW deploys with `1,000,000,000 mKRW` in the owner wallet. Prefer moving that
existing balance because `transfer` does not change total supply. Use `mint` only
when additional test-token supply is intentionally required.

```bash
cd giwa-contrract
read -s "DEPLOYER_PRIVATE_KEY?GIWA MockKRW owner private key: "
export DEPLOYER_PRIVATE_KEY

npm run mkrw:transfer -- 0xFUNDER_OR_BUYER_WALLET 8000000
# Alternative additional issuance:
npm run mkrw:mint -- 0xFUNDER_OR_BUYER_WALLET 8000000

unset DEPLOYER_PRIVATE_KEY
```

Use only one operation for the intended distribution. Amounts are integer mKRW
without commas or decimal points. The tasks read the recorded MockKRW address,
require chain ID `91342`, verify deployed code, zero decimals, the current
onchain owner, native gas balance and any required transfer balance, wait for one
confirmed transaction, and verify the emitted Transfer plus post-transaction
balances and total supply.
They print the transaction hash before waiting so an uncertain client result can
be checked in the explorer instead of submitted again. Confirmed-block state
reads are retried to tolerate GIWA public-RPC visibility lag; an exact confirmed
Transfer remains successful even if the follow-up state query still needs more
time.

### MockKRWFaucet deployment

The demo Faucet is deployed separately against the existing MockKRW. It does not
replace or modify the recorded MockKRW/ReceivableFinance pair. Source verification
is optional for runtime use.

Current GIWA Sepolia deployment:

- MockKRWFaucet: `0xa451FA95c3E2Efd771f6Ba556daBBf36f888ef2E`
- MockKRW: `0x5cD8a99Dcf5Fa00fb4fD9873b41A15F9C13C9d3F`
- Claim amount: `10,000,000 mKRW`
- Transaction: `0xcd71ff3c79c17bfc4627a22c87a72139c30cd5a69cf686025744f5b30f8cc633`
- Block: `32667847`
- Current pre-funded inventory: `600,000,000 mKRW`, enough for sixty fixed claims.

```bash
cd giwa-contrract
read -s "DEPLOYER_PRIVATE_KEY?GIWA MockKRW owner private key: "
export DEPLOYER_PRIVATE_KEY

# Optional before the first deployment; defaults to 10000000.
export MKRW_FAUCET_CLAIM_AMOUNT=10000000
npm run deploy:faucet:giwa

unset MKRW_FAUCET_CLAIM_AMOUNT
unset DEPLOYER_PRIVATE_KEY
```

The command validates chain ID `91342`, the existing MockKRW bytecode and token
identity, zero decimals, the recorded/current owner, and native gas balance. A
successful receipt is stored in
`giwa-contrract/deployment/giwa-testnet-faucet.json`; the original
`giwa-testnet.json` remains unchanged. If immediate RPC state reads lag, rerun the
same command. It recovers the recorded Faucet and does not submit another
deployment.

After deployment, pre-fund the printed Faucet address separately with the
existing owner inventory:

```bash
npm run mkrw:transfer -- 0xFAUCET_ADDRESS 200000000
```

This example funds twenty `10,000,000 mKRW` claims without increasing total
supply. Choose the inventory intentionally before running the transfer. The
deployment alone does not create Faucet inventory; this pre-funding transfer
must complete before testing the public claim button. The deployed Faucet has
since been funded with `600,000,000 mKRW`; do not repeat the example transfer
unless additional inventory is intentionally required.

## Frontend Configuration

Copy `giwa-ui/.env.example` to an ignored `.env.local` and set:

```env
VITE_GIWA_CHAIN_ID=
VITE_GIWA_CHAIN_ID_HEX=
VITE_GIWA_RPC_URL=
VITE_GIWA_EXPLORER_URL=
VITE_RECEIVABLE_FINANCE_ADDRESS=
VITE_MOCK_KRW_ADDRESS=
VITE_MOCK_KRW_FAUCET_ADDRESS=
```

Decimal and hexadecimal chain IDs must represent the same network.
The tracked development, production, and example files currently set
`VITE_MOCK_KRW_FAUCET_ADDRESS` to the deployed Faucet. Funding uses it when the
Funder balance is below `fundingAmount`; Repayment uses the same address when the
Buyer balance is below `faceValue`. Add the same variable in Vercel and rebuild.
No Railway/backend Faucet variable is required because claim is a frontend
MetaMask transaction outside the receivable lifecycle journal.

Vercel serves the Vue Router history-mode SPA through the root `vercel.json`
rewrite to `index.html`. Keep `public/robots.txt`, `public/sitemap.xml`, favicon,
Apple touch icon, and social preview assets in the Vite public directory. The
canonical and social metadata currently target `https://giwa-ui.vercel.app`;
update them together if the production domain changes.

## Railway Backend

Railway uses `giwa-api/Dockerfile`, which provides both the Java 17 compiler and
runtime. Do not keep a custom Railpack build command such as `./gradlew bootJar`;
let Railway detect the Dockerfile and use its ENTRYPOINT.

Source configuration

- If `giwa-api` is its own Git repository, the service root is the repository root.
- If the whole `gasok` repository is connected, set Railway Root Directory to
  `/giwa-api`.
- The build log should contain `Using detected Dockerfile`.

Add a Railway MySQL service, then create these reference variables on the API
service. Replace `MySQL` if the database service has a different name.

```env
MYSQLHOST=${{MySQL.MYSQLHOST}}
MYSQLPORT=${{MySQL.MYSQLPORT}}
MYSQLDATABASE=${{MySQL.MYSQLDATABASE}}
MYSQLUSER=${{MySQL.MYSQLUSER}}
MYSQLPASSWORD=${{MySQL.MYSQLPASSWORD}}
```

Also configure:

```env
JWT_SECRET=<random secret of at least 32 bytes>
CORS_ALLOWED_ORIGINS=https://<production-vercel-domain>
GIWA_RPC_URL=<backend GIWA Sepolia JSON-RPC URL>
GIWA_CHAIN_ID=<decimal GIWA Sepolia chain ID>
GIWA_RECEIVABLE_FINANCE_ADDRESS=<deployed ReceivableFinance address>
GIWA_MOCK_KRW_ADDRESS=<deployed MockKRW address>
GIWA_RPC_TIMEOUT_MS=10000
GIWA_MIN_CONFIRMATIONS=1
```

- Do not commit `JWT_SECRET`.
- Treat an RPC URL containing an API key as a secret and do not commit or log it.
- `GIWA_CHAIN_ID`, `GIWA_RECEIVABLE_FINANCE_ADDRESS`, and
  `GIWA_MOCK_KRW_ADDRESS` must match the deployed frontend network,
  ReceivableFinance contract, and its `paymentToken()`.
- Do not add a trailing slash to the CORS origin.
- Do not add a trailing slash to `VITE_API_URL`. The frontend also normalizes it
  defensively before appending API paths.
- Multiple exact origins are comma-separated.
- Railway provides `PORT`; the application reads it automatically.
- The existing `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, and `DB_PASSWORD`
  variables remain supported and override the Railway MySQL names.
- Remove stale or blank `DB_*` variables. Even an empty override prevents the
  corresponding Railway `MYSQL*` fallback from being used.

Database initialization

- For a brand-new empty Railway database, run `.codex/schema.sql` exactly once.
- That file contains destructive DROP statements. Never run it against a populated
  database.
- For an existing populated database, use the non-destructive migration files only.
- `spring.sql.init.mode=never`, so application startup does not create tables.
- Before deploying the transaction journal backend, run the non-destructive
  `.codex/migrations/20260730_blockchain_transactions.sql` against an existing
  database. The canonical fresh schema already contains this table.
- Before deploying RPC verification over an existing journal table that has none
  of the four RPC proof columns, run the preflight and one-time ALTER in
  `.codex/migrations/20260730_blockchain_transaction_rpc_verification.sql`.
- If that existing table has no `verification_version`, run
  `.codex/migrations/20260730_blockchain_transaction_verification_version.sql`
  after its preflight. This is also the upgrade path when the four proof columns
  were already deployed earlier.
- A fresh database initialized from the current `.codex/schema.sql` already has the
  proof columns and must not run either journal migration afterward.
- Funding and Repayment reuse existing receivable and journal columns. Neither
  requires a new database migration.
- A table created by the current
  `.codex/migrations/20260730_blockchain_transactions.sql` also already has the
  proof columns; do not run the RPC ALTER after it.

RPC-verified transaction journal rollout order

1. Back up MySQL and confirm whether `blockchain_transactions` exists.
2. If absent, run `.codex/migrations/20260730_blockchain_transactions.sql` and skip
   both ALTER migrations. If it exists with none of the four proof columns, run
   `.codex/migrations/20260730_blockchain_transaction_rpc_verification.sql` once.
   If the existing table lacks `verification_version`, then run
   `.codex/migrations/20260730_blockchain_transaction_verification_version.sql`
   once.
3. Configure all five `GIWA_*` Railway variables above.
4. Deploy the Railway backend and verify `/health`.
5. Verify an authenticated `POST /blockchain-transactions` can create PENDING.
6. Retry or execute a Seller create or Buyer verify transaction and require
   CONFIRMED to contain `blockHash`, `eventReceivableId`, and `rpcVerifiedAt`
   before receivable synchronization.
7. Deploy the Vercel frontend error-handling update.

Deploy the backend before the frontend because the updated frontend requires the
new journal endpoints.

For the tokenization rollout, deploy the backend `/tokenized` synchronization
endpoint before deploying the Seller mint CTA.

For Funding and Repayment rollout, deploy the backend `/funded` and `/repaid`
synchronization endpoints plus FUND_RECEIVABLE and REPAY_RECEIVABLE journal
verification before deploying their frontend pages.

After deployment:

1. Generate a Railway public domain.
2. Optionally set the Railway health-check path to `/health`.
3. Open `https://<railway-domain>/health` and require `{"status":"UP"}`.
   This endpoint is a liveness check only; it does not prove that MySQL is
   connected or that the schema has been initialized.
4. Set Vercel `VITE_API_URL=https://<railway-domain>`.
5. Redeploy Vercel because Vite environment variables are build-time values.
6. Test signup, login, wallet load, receivable list, Seller create, and Buyer verify
   from the Vercel origin.

## GIWA Live Verification

The Buyer Verify TODO is fully verified only after:

1. Seller sends `createReceivable` on GIWA.
2. `ReceivableCreated` yields an onchain ID.
3. Backend stores create tx metadata while status stays CREATED.
4. Buyer sends `verifyReceivable` from the registered Buyer wallet.
5. Backend RPC verifies each receipt, signer, contract call, and lifecycle event.
6. Backend status becomes VERIFIED with one verification history row.
7. Both transaction hashes open in the configured explorer.

## GIWA Tokenization Live Verification

The Tokenize implementation is live-verified only after:

1. Open a DB VERIFIED receivable as its Seller.
2. Confirm the Seller tokenization CTA is visible.
3. Submit `tokenizeReceivable` from the registered Seller MetaMask wallet.
4. Require the backend journal to become RPC-verified CONFIRMED with
   `eventTokenId`.
5. Require `POST /receivables/{id}/tokenized` to return status TOKENIZED with
   matching `tokenId` and `tokenizeTxHash`.
6. Confirm `ownerOf(tokenId)` is the ReceivableFinance contract escrow address.
7. Confirm the tokenize transaction opens in the configured explorer.

User verification result:

- The Seller mint CTA successfully submitted a real GIWA transaction.
- The NFT mint completed and the frontend received the synchronized TOKENIZED
  receivable result.
- The journal-aware recovery UI must be redeployed with the frontend before
  production users rely on cross-browser/manual synchronization recovery.

## GIWA Funding Live Verification

The Funding implementation is live-verified after:

1. Open an unrelated company's DB TOKENIZED receivable as Funder.
2. Approve the exact `fundingAmount` of MockKRW from the registered Funder wallet.
3. Submit `fundReceivable` as a separate MetaMask transaction.
4. Require the backend journal to verify ReceivableFunded, the Funder-to-Seller
   MockKRW Transfer, and the escrow-to-Funder NFT Transfer.
5. Require `POST /receivables/{id}/funded` to return status FUNDED with the
   Funder identity and `fundingTxHash`.
6. Confirm the Seller received `fundingAmount` mKRW and the Funder owns the NFT.

User verification result:

- The real Funder approval and fundReceivable transaction succeeded.
- The backend synchronized the receivable to FUNDED.
- The Seller mKRW receipt was confirmed.

## GIWA Repayment Live Verification

The Repayment implementation is live-verified only after:

1. Open a DB FUNDED receivable as its Buyer.
2. Confirm the displayed current NFT owner is the intended repayment recipient.
3. Approve the exact `faceValue` of MockKRW from the registered Buyer wallet.
4. Submit `repayReceivable` as a separate MetaMask transaction.
5. Require the backend journal to verify ReceivableRepaid and the
   Buyer-to-current-owner MockKRW Transfer for `faceValue`.
6. Require `POST /receivables/{id}/repaid` to return status REPAID with the
   matching `repayTxHash`.
7. Confirm the current NFT owner received the full face value and still owns the
   NFT.
8. Confirm the repayment transaction opens in the configured explorer.

User verification result:

- Buyer approval, repayReceivable, and DB REPAID synchronization succeeded.
- Exact current NFT-owner face-value receipt is still pending confirmation.
