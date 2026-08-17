# Midnight Glossary

- **Compact**: Midnight smart-contract language used to define private and
  public ledger state plus circuits.
- **Compact circuit / ZK circuit**: A contract function whose execution is
  proven with a zero-knowledge proof. It can assert facts about private witness
  data without publishing that data.
- **Witness**: Local code that supplies a circuit's private input at proving
  time. GASOK witnesses include mock-attested financial data and signatures.
- **Private state**: Encrypted local state retained by the CLI-compatible local
  participant. ADR-018's Bridge reuses that same state; Vue does not read it.
  Private state is never submitted to the Midnight network.
- **Public state**: Contract state queryable from the Midnight Indexer. For the
  Phase 2.5 result map it contains an opaque lookup key plus eligibility,
  provider ID, and policy version; the contract admin, registered Provider
  public keys, configured GIWA chain, and contract are also public values.
- **Attestation**: A signed statement binding a provider to specific private
  input values and an exact GIWA/Midnight proof context.
- **Attestation Provider**: The signer whose public key is registered in the
  contract. GASOK's providers are mock-only, not financial institutions.
  Provider 1 is the legacy role-context policy; Provider 2 first enforces the
  off-chain EIP-712 role-wallet authorization policy.
- **Schnorr signature**: The signature scheme used by the official ZK Loan
  pattern and verified inside the Compact circuit.
- **Proof Server**: Trusted local service that generates ZK proofs from circuit
  material and the plaintext private witness. It listens on port 6300. It does
  not receive the Midnight wallet key or sign the transaction.
- **Midnight Node**: The blockchain node that validates and accepts Midnight
  transactions. The local endpoint is port 9944.
- **Indexer**: GraphQL service for querying and subscribing to public Midnight
  ledger state. The local endpoint is port 8088.
- **Read-only Midnight API**: Local port-4100 adapter pinned to the approved
  Midnight contract. Its exact `POST /v1/eligibility-results/resolve` endpoint
  verifies a version-1 capability, recomputes its lookup key, queries that one
  Indexer result, and returns the approved Vue DTO. It does not anonymously list
  results, create proofs, sign transactions, or access private state.
- **Local Proof Bridge**: Trusted, custodial development process on loopback
  port 4200. It reuses the proven CLI encrypted private state and Midnight dev
  wallet, coordinates Provider 2, asks the Proof Server to generate a proof,
  and submits the transaction. It is not Spring Boot, a remote service, or a
  production wallet architecture.
- **Proof session**: CSPRNG-identified, memory-only, one-shot Bridge record that
  moves from authorization through attestation, proving/submission, indexing,
  and a terminal state. Its ID is sent only in POST bodies. A completed session
  returns a proof capability, which Vue resolves independently through the read
  API rather than trusting the Bridge for the public result. Completion proves
  transaction finalization, not that the asynchronous Indexer is already caught
  up. The `indexing` state is a transition into completion, not a Bridge-side
  query; delayed public visibility is recovered by retrying only the resolver
  read. An unsigned prepared session expires on an internal timer even without
  another browser request, discarding its prepared tuple and releasing the one
  active slot.
- **Midnight Local Dev**: Official Docker-based local environment containing a
  Node, Indexer, Proof Server, and funding wizard.
- **`undeployed`**: Network ID for the local Midnight development environment.
  It is the only allowed network for this PoC.
- **Preprod / Mainnet**: Remote Midnight networks. They are explicitly outside
  this PoC's deployment scope.
- **Company commitment**: A one-way pseudonym derived from the local company
  secret and PIN. It is not a business number, legal-company identity, or proof
  that the requester controls a GIWA wallet.
- **GIWA receivable subject**: One canonical party of one onchain receivable,
  identified by GIWA chain, ReceivableFinance address, uint256 receivable ID,
  `SELLER` or `BUYER` role, and the corresponding wallet read from GIWA RPC.
  This context label does not prove that caller-supplied financial inputs belong
  to that party.
- **GIWA binding hash**: Domain-separated hash of the GIWA receivable subject.
  The hash is signed and checked by Compact so a signature cannot be moved to a
  different receivable, role, wallet, chain, or ReceivableFinance deployment.
- **Midnight deployment hash**: Domain-separated hash of the local Midnight
  contract address. It prevents an attestation for one deployment from being
  replayed against another deployment.
- **Receivable eligibility lookup key**: Opaque public map key derived from the
  company commitment, GIWA binding hash, Midnight deployment hash, and policy
  version. The ledger value contains only eligibility, provider ID, and policy
  version.
- **Proof capability**: Versioned CLI output that gives an intended verifier the
  lookup key and the public GIWA/Midnight context needed to interpret it. The
  dev-only Vue verifier accepts a manually pasted capability and resolves only
  that exact result. It has no PIN, secret, raw financial value, or signature,
  but it is correlation-sensitive, is not meant for general publication, and
  still requires a future secure delivery/access design.
- **One-shot result**: A rule that rejects insertion when the exact lookup key
  already exists. It prevents exact replay but does not provide freshness,
  expiry, revocation, or a latest-result policy.
- **Authorization challenge**: A cryptographically random Provider 2 record
  held in Mock Provider memory for at most two minutes and consumed on the first
  attestation attempt. The expiry bounds its lifetime; atomic one-shot
  consumption prevents replay. Neither makes the resulting Midnight eligibility
  result fresh or unexpired.
- **Attestation request commitment**: Salted hash that binds the exact private
  mock financial tuple and company commitment to the public authorization
  context. The manual `/midnight/authorize` tool sees only this hash, never the
  raw financial values or hidden salt. `/midnight/prove` has already accepted
  the caller-supplied raw tuple transiently before the challenge, but it still
  never receives the Bridge-generated hidden salt.
- **Wallet-control authorization**: The Provider 2 EIP-712 signature proving
  control of the canonical Seller/Buyer EOA at attestation issuance time. Vue
  signs it through `/midnight/authorize` or `/midnight/prove`, and the Mock
  Provider recovers it off-chain before Schnorr issuance. Compact/Midnight does
  not independently verify this secp256k1 signature. Provider 1 results do not
  have this property.
- **Funding eligibility**: The public boolean/result indicating whether the
  private financial witness met the Compact policy.
- **Lace Wallet**: Midnight-compatible browser wallet. Current official
  Midnight Local Dev and wallet-connector documentation supports Lace on local
  `undeployed` using the local Node, Indexer, and Proof Server. ADR-018 still
  uses the Bridge to preserve the already-proven CLI identity/private state;
  direct Vue + Lace is a possible later self-custody replacement, not the
  architecture implemented now.
