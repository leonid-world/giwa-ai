# Midnight Glossary

- **Compact**: Midnight smart-contract language used to define private and
  public ledger state plus circuits.
- **Compact circuit / ZK circuit**: A contract function whose execution is
  proven with a zero-knowledge proof. It can assert facts about private witness
  data without publishing that data.
- **Witness**: Local code that supplies a circuit's private input at proving
  time. GASOK witnesses include mock-attested financial data and signatures.
- **Private state**: Encrypted local state retained by the CLI or browser. It is
  never submitted to the Midnight network.
- **Public state**: Contract state queryable from the Midnight Indexer. For this
  PoC it contains only verification outputs and commitments.
- **Attestation**: A signed statement binding a provider to specific private
  input values.
- **Attestation Provider**: The signer whose public key is registered in the
  contract. GASOK's initial provider is mock-only, not a financial institution.
- **Schnorr signature**: The signature scheme used by the official ZK Loan
  pattern and verified inside the Compact circuit.
- **Proof Server**: Local service that generates ZK proofs from circuit material
  and private witness data. It listens on port 6300.
- **Midnight Node**: The blockchain node that validates and accepts Midnight
  transactions. The local endpoint is port 9944.
- **Indexer**: GraphQL service for querying and subscribing to public Midnight
  ledger state. The local endpoint is port 8088.
- **Read-only Midnight API**: Local port-4100 adapter that queries the Indexer,
  decodes the Compact public ledger, and returns the approved Vue DTO. It does
  not create proofs, sign transactions, or access private state.
- **Midnight Local Dev**: Official Docker-based local environment containing a
  Node, Indexer, Proof Server, and funding wizard.
- **`undeployed`**: Network ID for the local Midnight development environment.
  It is the only allowed network for this PoC.
- **Preprod / Mainnet**: Remote Midnight networks. They are explicitly outside
  this PoC's deployment scope.
- **Company commitment**: A one-way public value that binds a verification
  result to a company context without revealing raw financial values.
- **Funding eligibility**: The public boolean/result indicating whether the
  private financial witness met the Compact policy.
- **Lace Wallet**: Midnight-compatible browser wallet. It is not required for
  CLI phases, and the official ZK Loan UI documents that Lace cannot balance or
  sign for the local `undeployed` chain.
