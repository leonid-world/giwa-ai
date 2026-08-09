# Codex Workflow

Before coding

- Read every file under docs/ai.
- Analyze repository.
- Compare code with TODO.
- Report implementation plan.

After coding

- Update TODO.md.
- Update CONTEXT.md.
- Update DECISIONS.md if architecture changed.
- Run build.
- Run tests if available.
- Report modified files.

Rules

- Never store private keys.
- Never refactor architecture without approval.
- Follow existing coding conventions.
- Keep MVP simple.
- Do not add unnecessary abstractions.
- Backend never signs blockchain transactions.
- Frontend always uses MetaMask Signer.
