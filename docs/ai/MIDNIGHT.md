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
signature, the user secret, and witness randomness. They remain encrypted at
rest in CLI or later browser private state. The local Mock Attestation API must
see the raw values to sign them, and the local Proof Server must process the
witness to generate a proof; both are inside the local PoC trust boundary and
must not persist or log the values. Raw values must not be stored in MySQL,
Midnight public state, committed files, or frontend telemetry.

The initial public result is limited to a pseudonymous verification commitment,
provider ID, policy version, and funding eligibility. Result existence means a
valid proof transaction was accepted, so a second verification-status field is
unnecessary. Risk tier, maximum funding ratio, and issued/expiry timestamps are
deferred until their policies and time source are approved. The public result
contains no raw financial value or signature.

During CLI-only Phase 2, the commitment is derived from a local secret and PIN.
It is not yet cryptographically bound to a GASOK business number. That binding
belongs to a later integration design and must not use an unsalted hash of the
low-entropy 10-digit business number.

## Local Components and Ports

| Component | Purpose | Host endpoint |
| --- | --- | --- |
| Midnight Node | validates Midnight transactions | `http://127.0.0.1:9944` |
| Midnight Indexer | public-state query and subscription | `http://127.0.0.1:8088/api/v4/graphql` |
| Midnight Indexer WS | state subscription | `ws://127.0.0.1:8088/api/v4/graphql/ws` |
| Midnight Proof Server | local ZK-proof generation | `http://127.0.0.1:6300` |
| Mock Attestation API | mock financial-data signing | `http://127.0.0.1:4000` |
| Read-only Midnight API | decodes Indexer public state for the dev-only Vue viewer | `http://127.0.0.1:4100` |

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

Phase 3A is the wallet-free read side of step 8. The development-only Vue route
`/midnight` calls `giwa-midnight/api` through the Vite `/midnight-api` proxy.
That adapter uses the official Indexer public-data provider and generated
`GasokEligibility.ledger()` decoder, returns only commitment, eligibility,
Mock Provider ID, and policy version, and has no transaction or private-state
surface. It binds to `127.0.0.1`, uses `Cache-Control: no-store`, and is not a
Spring Boot API. It accepts only the configured GASOK contract address and
limits each Indexer query to 10 seconds; a local redeployment requires updating
both `MIDNIGHT_CONTRACT_ADDRESS` for the adapter and
`VITE_MIDNIGHT_DEFAULT_CONTRACT` for Vue.

The official ZK Loan browser UI is Preprod-only: Lace cannot provide balance or
sign transactions for the local `undeployed` chain. Because this PoC forbids
Preprod, browser proof submission is not simulated. Phase 3B requires a
separately approved local-wallet or local bridge trust boundary.

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
- No claim that Lace can submit a proof on the local `undeployed` chain.
- No raw financial-data persistence in MySQL or public ledger state.
- No real bank, credit-bureau, or accounting-provider claim.
- No change to existing GASOK flows until the proof has succeeded through CLI.
