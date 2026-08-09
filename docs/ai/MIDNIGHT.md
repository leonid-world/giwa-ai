# GASOK Midnight PoC

## Scope

The Midnight PoC exists only on the `gasok-midnight` branch. It privately
verifies Seller financial eligibility for the existing GASOK receivable-funding
demo. It is local-only and uses the Midnight `undeployed` network.

`giwa-midnight/` is the dedicated workspace and Git submodule. Its target
structure is `contract/`, `api/`, `cli/`, `attestation-api/`, and
`docker-compose.yml`.

GIWA remains the sole system for receivable tokenization, funding, repayment,
wallet mapping, and asset transactions. Midnight does not replace or deploy a
GIWA Solidity contract.

## Trust Model

The GASOK Attestation API is a local mock provider. It signs demonstration
financial inputs but does not prove that they originated from a bank, tax
authority, accounting firm, or ERP provider. Product copy must call the result
"mock-attested" and must never describe it as bank-verified.

Midnight proves that a registered provider signed the witness data, the signed
data was not modified, and the Compact eligibility circuit executed correctly.
It does not prove that the underlying real-world financial data is truthful.

## Private and Public Data

Private inputs are annual revenue, debt ratio, overdue count, attestation
signature, the user secret, and witness randomness. They remain in the CLI or
later the user's browser private state. They must not be stored in MySQL,
Midnight public state, logs, committed files, or frontend telemetry.

Public results are limited to a company commitment, provider ID, verification
status, funding eligibility, risk tier, maximum funding ratio, and issued/expiry
timestamps. The public result contains no raw financial value or signature.

## Local Components and Ports

| Component | Purpose | Host endpoint |
| --- | --- | --- |
| Midnight Node | validates Midnight transactions | `http://127.0.0.1:9944` |
| Midnight Indexer | public-state query and subscription | `http://127.0.0.1:8088/api/v4/graphql` |
| Midnight Indexer WS | state subscription | `ws://127.0.0.1:8088/api/v4/graphql/ws` |
| Midnight Proof Server | local ZK-proof generation | `http://127.0.0.1:6300` |
| Mock Attestation API | mock financial-data signing | `http://127.0.0.1:4000` |

The proof server is localhost-only. Do not run a second independent container on
port 6300 while Midnight Local Dev is running.

## Required Delivery Order

1. Reproduce the official ZK Loan contract, CLI, and Attestation API unchanged.
2. Compile Compact and build the contract package.
3. Run the local Node, Indexer, and Proof Server.
4. Complete the official CLI flow: fund wallet, deploy, register provider,
   request attestation, generate proof, submit, and query public state.
5. Replace only the loan-domain fields with GASOK financial eligibility fields.
6. Connect the mock provider and test valid and invalid signatures through CLI.
7. Verify the full GASOK proof flow through CLI.
8. Only then add isolated Vue services/composables. Add Spring Boot only if an
   already-proven Vue flow needs backend coordination.

## Initial Eligibility Policy

The initial private-input policy is:

- annual revenue >= 500,000,000 KRW
- debt ratio <= 200%
- overdue count <= 1

Threshold values and policy version are contract configuration, not raw company
financial data. Any policy change after the first working CLI proof requires a
documented contract-version decision.

## Exclusions

- No Midnight Preprod or Mainnet deployment.
- No React dependency or React conversion.
- No Lace Wallet requirement during CLI Phase 1.
- No raw financial-data persistence in MySQL or public ledger state.
- No real bank, credit-bureau, or accounting-provider claim.
- No change to existing GASOK flows until the proof has succeeded through CLI.
