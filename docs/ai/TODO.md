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

- [x] Define `giwa-midnight/` as the dedicated Git-submodule workspace
- [x] Document local-only Midnight trust boundary, network, components, and data classification
- [x] Add `giwa-midnight` Git submodule from `leonid-world/giwa-midnight`
- [x] Initialize the Node 24 Midnight workspace and install official SDK dependencies
- [ ] Official ZK Loan contract compiles
- [ ] Local Midnight services are healthy
- [ ] Official CLI deploys a contract
- [ ] Mock provider is registered
- [ ] Official loan proof succeeds
- [ ] Financial fields replace credit fields
- [ ] GASOK financial proof succeeds via CLI
- [ ] Vue integration begins
