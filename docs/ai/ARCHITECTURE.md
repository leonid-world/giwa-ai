# Architecture

Frontend
│
├── Login
├── Dashboard
├── Receivable
├── Wallet
├── Funding
└── Repayment

↓

Backend

↓

MySQL

↓

GIWA Blockchain

↓

MetaMask

## Responsibility

Frontend

- UI
- Wallet
- MetaMask
- Signer
- Contract Call
- Separate MockKRW Approval and Funding/Repayment Calls

Backend

- Authentication
- Business Logic
- Database
- REST API
- RPC Receipt/Event Verification
- Blockchain Transaction Journal

Blockchain

- Ownership
- Tokenization
- Settlement
- Current NFT Owner Repayment
