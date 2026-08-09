# Architecture

## Network Boundary

GIWA and Midnight are separate networks with separate responsibilities. The
Midnight PoC is local-only on `undeployed`; GIWA remains the existing testnet
integration. No Midnight deployment reaches Preprod or Mainnet.

```text
GASOK Vue (existing giwa-ui)
  ├─ Existing GIWA lifecycle UI ──────────────────────────────┐
  └─ Phase-3-only eligibility UI ───────────────────────────┐ │
                                                        │ │
GASOK Spring Boot (existing giwa-api)                   │ │
  ├─ auth / business logic / MySQL                       │ │
  └─ GIWA RPC verification and transaction journal       │ │
                                                        │ │
GIWA Sepolia                                            │ │
  └─ ReceivableFinance / MockKRW / NFT settlement ◀─────┘ │
                                                          │
Midnight PoC (giwa-midnight, local Docker network) ◀─────┘
  ├─ CLI and encrypted private state
  ├─ Mock Attestation API
  ├─ Proof Server
  ├─ Midnight Node
  └─ Indexer → public eligibility result only
```

## Component Ownership

| Component | Owns | Must not own |
| --- | --- | --- |
| Vue | UI and, after CLI proof success, display/submission of an eligibility verification | private keys, raw financial data persistence, GIWA architecture changes |
| Spring Boot | authentication, MySQL, REST API, GIWA receipt/event verification and journal | Attestation-provider signing in the initial PoC, Midnight transaction signing |
| GIWA contracts | receivable ownership, tokenization, funding, repayment, current-NFT-owner settlement | financial eligibility proof or private financial data |
| Midnight contract | provider-signature verification, private eligibility proof, public outcome/commitment | raw financial values, GIWA assets, GIWA lifecycle state |
| Attestation API | mock financial-data signing with an ignored local provider key | bank/accounting-provider claim, MySQL persistence |
| CLI | Phase-1 and Phase-2 deployment, provider registration, proof submission, local encrypted private state | Vue replacement or production wallet flow |
| Proof Server | local proof generation | public remote exposure or persisted raw financial data |
| Indexer | query of public Midnight result | private witness/state query |

## Data Classification

| Data | Location |
| --- | --- |
| annual revenue, debt ratio, overdue count, signatures, witness secret, mnemonic | CLI/browser encrypted private state only |
| provider ID, company commitment, verification status, eligibility, risk tier, maximum funding ratio, issue/expiry time | Midnight public state and Indexer |
| account/company identity, receivable lifecycle, GIWA transaction proof summaries | existing Spring Boot/MySQL and GIWA chain according to existing rules |

## Integration Sequence

1. `giwa-midnight` CLI proves the official ZK Loan example locally.
2. The same CLI proves the GASOK financial-eligibility contract.
3. Vue consumes only the proven public result through an isolated Midnight service
   or composable.
4. Spring Boot integration is considered only when required by the proven Vue
   flow. Existing GIWA lifecycle APIs and Solidity contracts remain unchanged.
