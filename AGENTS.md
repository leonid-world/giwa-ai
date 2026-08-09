# Project Agent Rules

## Before Coding

- Read all files under docs/ai.
- Understand the current architecture.
- Compare the codebase with TODO.md and CONTEXT.md.
- Never change architecture without approval.

## During Coding

- Follow existing coding conventions.
- Keep the MVP simple.
- Avoid unnecessary refactoring.

## After Coding

- Update TODO.md.
- Update CONTEXT.md.
- Update DECISIONS.md only if an architectural decision changed.
- Run the appropriate build or tests.
- Summarize modified files.

## GASOK Midnight PoC

The `gasok-midnight` branch is a local-only privacy proof-of-concept.

### Goal

Add a Midnight-based private financial eligibility verification flow
without changing the production GIWA funding architecture.

Midnight is used only for:

- private financial input handling
- attestation signature verification
- zero-knowledge eligibility proof generation
- public verification result storage

GIWA remains responsible for:

- receivable tokenization
- funding
- repayment
- wallet and asset transactions

### Hard constraints

- Do not deploy Midnight contracts to Preprod or Mainnet.
- Do not replace Vue with React.
- Do not rewrite the existing GIWA Solidity contracts unless required.
- Do not store raw private financial data in MySQL or Midnight public state.
- The local Attestation API is a mock provider, not a real bank or accounting institution.
- Never describe mock-attested data as bank-verified data.
- Preserve all existing GASOK flows outside the Midnight PoC.

### Required implementation order

1. Add the `giwa-midnight/` workspace.
2. Run the official ZK Loan example locally without modifications.
3. Start the local Midnight node, indexer, and proof server.
4. Complete the official CLI proof flow.
5. Replace ZK Loan credit fields with GASOK financial fields.
6. Verify the GASOK flow through the CLI.
7. Integrate the working flow into the existing Vue frontend.
8. Integrate the backend only after the CLI and Vue proof flows work.

Do not skip directly to frontend integration.
