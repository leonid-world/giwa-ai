# Architecture

## Network Boundary

GIWA and Midnight are separate networks with separate responsibilities. The
Midnight PoC is local-only on `undeployed`; GIWA remains the existing testnet
integration. No Midnight deployment reaches Preprod or Mainnet.

```text
GASOK Vue (existing giwa-ui)
  ├─ Existing GIWA lifecycle UI ──────────────────────────────┐
  └─ Dev-only /midnight public-result viewer ──────────────┐ │
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
  ├─ Indexer → public eligibility result only
  └─ Read-only API (127.0.0.1:4100) → decoded public DTO only
```

## Component Ownership

| Component | Owns | Must not own |
| --- | --- | --- |
| Vue | Dev-only display of the already-proven public eligibility result; a future approved submission UI | private keys, raw financial data persistence, GIWA architecture changes |
| Spring Boot | authentication, MySQL, REST API, GIWA receipt/event verification and journal | Attestation-provider signing in the initial PoC, Midnight transaction signing |
| GIWA contracts | receivable ownership, tokenization, funding, repayment, current-NFT-owner settlement | financial eligibility proof or private financial data |
| Midnight contract | provider-signature verification, private eligibility proof, public outcome/commitment | raw financial values, GIWA assets, GIWA lifecycle state |
| Attestation API | mock financial-data signing with an ignored local provider key | bank/accounting-provider claim, MySQL persistence |
| CLI | Phase-1 and Phase-2 deployment, provider registration, proof submission, local encrypted private state | Vue replacement or production wallet flow |
| Proof Server | local proof generation | public remote exposure or persisted raw financial data |
| Indexer | query of public Midnight result | private witness/state query |
| Read-only Midnight API | local Indexer query and generated Compact-ledger decoding for Vue | wallet, proof, attestation, mutation, Spring/MySQL responsibilities |

## Data Classification

| Data | Location |
| --- | --- |
| annual revenue, debt ratio, overdue count, signatures, witness secret, mnemonic | CLI encrypted private state at rest; processed transiently by the local Mock Attestation API and Proof Server without persistence or logging |
| provider ID, policy version, pseudonymous company/verification commitment, eligibility | Midnight public state and Indexer |
| account/company identity, receivable lifecycle, GIWA transaction proof summaries | existing Spring Boot/MySQL and GIWA chain according to existing rules |

## Integration Sequence

1. `giwa-midnight` CLI proves the official ZK Loan example locally.
2. The same CLI proves the GASOK financial-eligibility contract.
3. Phase 3A lets Vue consume only the proven public result through the local
   read-only adapter and an isolated service/composable.
4. Local browser proof submission remains a separate Phase 3B design because
   the official Lace path cannot sign the local `undeployed` chain.
5. Spring Boot integration is considered only when required by the proven Vue
   flow. Existing GIWA lifecycle APIs and Solidity contracts remain unchanged.

Phase 2 does not enforce a GIWA funding gate. Risk tiers, maximum funding ratios,
issued/expiry timestamps, and binding the pseudonymous CLI commitment to a GASOK
company require separately approved policies before implementation.
