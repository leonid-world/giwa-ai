# WORKFLOW

## GASOK Main Business Flow

회원가입
↓
로그인
↓
MetaMask 연결
↓
기업-지갑 매핑
↓
채권 등록
↓
Buyer 채권 내용 검토
↓
Seller GIWA 채권 생성
↓
Buyer 명시적 채무 확인 + GIWA 지갑 검증
↓
토큰화
↓
제3자 Funder가 TOKENIZED 채권 선택
↓
잔액 부족 시 Funder가 사전 예치된 데모 mKRW 1회 충전
↓
Funder MockKRW 사용 승인
↓
Funder 자금 공급 + Seller mKRW 지급 + NFT 이전
↓
Backend RPC 검증 + DB FUNDED 동기화
↓
Buyer가 faceValue와 현재 NFT 소유자 확인
↓
잔액 부족 시 Buyer가 사전 예치된 데모 mKRW 1회 충전
↓
Buyer MockKRW 사용 승인
↓
Buyer 상환 + 현재 NFT 소유자에게 faceValue 지급
↓
Backend ReceivableRepaid + MockKRW Transfer RPC 검증
↓
DB REPAID 동기화

## GASOK Midnight PoC Flow

Midnight functionality is implemented only on the `gasok-midnight` branch.

The Midnight integration is a local-only privacy proof-of-concept.
It must not change the production GIWA funding architecture.

The existing GASOK flow remains intact.
Midnight is inserted only as an additional eligibility check over
caller-supplied mock financial inputs, bound to a per-receivable Seller/Buyer
role context.

Phase 2.5 proof creation is complete through the CLI: the Mock Provider resolves
receivable `#1` and its canonical role wallets from GIWA RPC, then separate
role-bound proofs produce separate opaque ledger keys. The inputs remain
caller-supplied mock values and are not proven to belong to those wallets. The
read side is also complete for the local PoC: an intended Funder can paste one
intentionally shared Proof capability into the development-only Vue page, which
requests an exact result from the pinned local adapter. This remains reference
information, not a Funding gate.

ADR-017 code adds a separate Provider 2 issuance gate without moving proof
submission into Vue. The CLI keeps the private values and hidden salt, hands a
two-minute EIP-712 request to `/midnight/authorize`, receives one-line MetaMask
signature JSON back, and then asks the Provider for its Schnorr attestation.
Provider 1 results remain legacy. The actual Provider 2 registration,
Seller MetaMask-signing handoff, and full CLI-submitted local Midnight runtime
E2E have now completed. The manual `/midnight/authorize` route remains available
for learning and diagnostics.

ADR-018 adds the complete development-only Vue path without changing GIWA or
Spring. `/midnight/prove` sends the private mock tuple once to a trusted
loopback Proof Bridge, clears it after challenge creation, asks MetaMask to
authorize the canonical role only after an explicit user action, and polls the
Bridge while it attests, proves, and submits through the existing CLI-compatible
Midnight wallet/private state. The completed capability is then resolved through
the independent read adapter and Indexer. This is a custodial local PoC, not a
Funding gate or production wallet design.

### Planned Integrated Business Flow

This is a future Phase 3 product flow, not current Phase 2.5 enforcement. The
bound proof result does not authorize or block any GIWA funding action. Making
Midnight eligibility a GIWA funding gate requires separate approval and an
architectural decision after browser proof submission and the remaining access
controls are proven.

The preserved Phase 3A read side accepts a manually pasted,
correlation-sensitive Proof capability and calls
`POST /v1/eligibility-results/resolve`. The adapter is the sole authority for
the pinned contract
`7e3ea9d741ce0f5862db6f46d0ad720be2586cd7d0405ec77e4a0478aa50f4fb`;
there is no Vue contract-address default and no anonymous GET result list. The
page displays only the exact receivable number, Seller/Buyer role, canonical
party wallet, eligibility, provider ID, and policy version. The new
`/midnight/prove` route uses the same resolver after its Bridge session. Neither
route affects Funding. Secure multi-user capability delivery and verifier access
remain future work; the direct loopback response and manual paste are local
learning-PoC handoffs only.

회원가입
↓
로그인
↓
MetaMask 연결
↓
기업-지갑 매핑
↓
채권 등록
↓
Seller와 Buyer가 각자 자신의 역할에 대한 증명 흐름 실행
↓
Bridge가 HTTP 입력 전 10초 제한 startup Indexer preflight로 계약/Provider를 확인하고
검증된 GIWA 설정을 memory에 seal함
↓
dev-only `/midnight/prove`가 비공개 mock 재무값과 PIN을 component memory에서 입력받음
↓
Vue가 loopback Proof Bridge에 challenge 생성을 1회 요청
↓
Bridge가 CLI-compatible encrypted private state와 Provider 2 challenge를 준비
↓
challenge 응답 즉시 Vue가 raw 재무값과 PIN을 reactive form에서 제거
↓
사용자가 별도 버튼을 눌러 canonical 역할 지갑을 MetaMask에서 명시적으로 서명
↓
Vue가 session ID와 authorization response를 Bridge에 전달
↓
GASOK Mock Attestation API가 GIWA RPC에서 채권과 역할 지갑 확인
↓
Mock Provider가 challenge를 1회 소비하고 commitment와 EOA signer를 검증
↓
검증 후 Provider 2가 재무정보와 채권 역할 컨텍스트에 Schnorr 서명
↓
Bridge가 각 역할의 Attested Financial Data를 encrypted private state의 witness로 준비
↓
Midnight Witness가 비공개 재무정보를 Compact Circuit에 전달
↓
Compact Circuit이 다음 항목 검증

- 등록된 Attestation Provider의 서명인지 확인
- 서명이 같은 GIWA 채권/역할/지갑 및 Midnight 배포용인지 확인
- 연매출 기준 충족 여부 확인
- 부채비율 기준 충족 여부 확인
- 연체 건수 기준 충족 여부 확인
  ↓
  Midnight Proof Server가 ZK Proof 생성
  ↓
  Midnight Local Node가 Proof 검증
  ↓
  검증 성공 시 Funding Eligibility 결과를 Midnight Ledger에 기록
  ↓
  Bridge가 finalized proof capability를 보존하고 완료 session에 반환
  (per-session Indexer 조회는 하지 않음)
  ↓
  Vue가 read API에 capability를 제출하고 opaque lookup key로 Indexer 결과 재조회
  (지연 시 이 resolver 조회만 재시도)
  ↓
  독립 조회가 성공한 뒤 GASOK Frontend에서 Midnight Verification 상태 표시
  ↓
  Buyer 채권 내용 검토
  ↓
  Seller GIWA 채권 생성
  ↓
  Buyer 명시적 채무 확인 + GIWA 지갑 검증
  ↓
  토큰화
  ↓
  제3자 Funder가 TOKENIZED 채권 선택
  ↓
  Funder가 Midnight Verification 결과 확인
  ↓
  별도 승인 전에는 참고 정보로만 표시하고 Funding gate로 사용하지 않음
  ↓
  잔액 부족 시 Funder가 사전 예치된 데모 mKRW 1회 충전
  ↓
  Funder MockKRW 사용 승인
  ↓
  Funder 자금 공급 + Seller mKRW 지급 + NFT 이전
  ↓
  Backend RPC 검증 + DB FUNDED 동기화
  ↓
  Buyer가 faceValue와 현재 NFT 소유자 확인
  ↓
  잔액 부족 시 Buyer가 사전 예치된 데모 mKRW 1회 충전
  ↓
  Buyer MockKRW 사용 승인
  ↓
  Buyer 상환 + 현재 NFT 소유자에게 faceValue 지급
  ↓
  Backend ReceivableRepaid + MockKRW Transfer RPC 검증
  ↓
  DB REPAID 동기화

## Midnight PoC Development Workflow

Midnight development must follow the order below.

Do not skip directly to frontend integration.

1. Create and switch to the `gasok-midnight` branch.
2. Add `giwa-midnight/` as the dedicated Midnight workspace under the GASOK root.
   It is a Git submodule backed by `https://github.com/leonid-world/giwa-midnight.git`.
   The legacy untracked `midnight/` directory is not the PoC workspace and must
   not receive implementation files.
3. Reproduce the official Midnight ZK Loan tutorial inside `giwa-midnight/`
   without GASOK-specific behavioral modifications.
4. Compile the original Compact contract successfully.
5. Start the local Midnight development environment.
   - Midnight Node
   - Midnight Indexer
   - Midnight Proof Server
6. Run the official CLI flow end-to-end.
   - deploy Compact contract
   - register Mock Attestation Provider
   - request attestation
   - generate ZK proof
   - submit Midnight transaction
   - query public verification result
7. Replace ZK Loan credit data with GASOK financial data.
8. Verify the GASOK financial eligibility flow through CLI.
9. Bind separate Seller and Buyer proofs to the canonical GIWA receivable
   context and verify both through CLI.
10. Integrate the exact, capability-based read side into the existing Vue
    frontend through the adapter's single pinned contract authority.
11. Add the Provider 2 two-step CLI/Vue EIP-712 authorization handoff while raw
    financial values remain in CLI/Provider memory for that manual flow.
12. Register Provider 2 and verify the actual MetaMask authorization plus local
    attestation, proof submission, and Indexer result end to end.
13. Add the approved trusted local Proof Bridge and `/midnight/prove`; verify
    challenge, explicit MetaMask authorization, Provider attestation, proof,
    transaction submission, status polling, and independent Indexer resolution.
14. Treat secure multi-user capability delivery/access and direct Vue + Lace
    self-custody as separate later architectures. Current official local Lace
    support means this is a product/identity migration choice, not a platform
    blocker.
15. Integrate Spring Boot only if backend coordination is required by the proven
    browser flow.

## Midnight Verification Rules

The initial PoC may use rules such as:

- annualRevenue >= 500,000,000 KRW
- debtRatio <= 200%
- overdueCount <= 1

These values are private inputs.

Raw financial values must not be written to:

- Midnight public ledger state
- GASOK MySQL
- application logs
- committed local files

The Phase 2.5 public eligibility-result entries may expose only:

- eligible
- providerId
- policyVersion
- opaque receivable-eligibility lookup key
- sealed GIWA chain and ReceivableFinance deployment configuration

The contract admin and registered Provider public-key registry are also public
control-plane state. They are not financial result fields.

The CLI proof capability additionally contains the pseudonymous company
commitment and GIWA/Midnight correlation context required to interpret the
opaque key. It must be shared only with an intended verifier. Risk tiers,
maximum funding ratios, legal-company identity, issued/freshness/latest/expiry
semantics, refresh rounds, and Funding enforcement remain deferred until their
policies are approved.

## Trust Boundary

The GASOK Attestation API is a mock provider for local demonstration purposes.

It does not prove that financial data originated from:

- a real bank
- a tax authority
- an accounting firm
- an ERP provider

Midnight verifies:

- that the financial data was signed by a registered provider
- that the signed data was not modified
- that the signature is bound to the encoded GIWA receivable party and Midnight deployment
- that the Compact Circuit was executed correctly
- that the Zero-Knowledge Proof is valid

The Mock Provider verifies through GIWA RPC which Seller or Buyer wallet the
configured ReceivableFinance contract records, then labels the caller-supplied
mock inputs with that role context. Provider 2 also issues a bounded two-minute
challenge, consumes it once, recomputes the salted private-request commitment,
and recovers the canonical EOA from the EIP-712 response before Schnorr
issuance. Provider 1 does not have that gate and is legacy.

In ADR-018, Vue sees the raw mock tuple and PIN only until the Bridge returns a
challenge, then clears them. The trusted loopback Bridge owns the existing
Midnight development wallet and encrypted participant private state. The local
Proof Server receives the plaintext witness needed to construct the proof but
never the wallet key. MetaMask signs only the GIWA role EIP-712 request. The
Midnight public ledger receives only the opaque key and minimal result.

The Compact circuit verifies the registered Provider's Schnorr signature; it
does not independently verify the secp256k1 EIP-712 signature. Provider 2
therefore proves canonical role-wallet control only at issuance time. It does
not prove that the inputs belong to that wallet, real-world financial truth,
legal-company identity, bank/accounting provenance, current eligibility, or
Funding approval.

## Local Development Rules

Use only the local Midnight `undeployed` network.

Expected local components:

- GASOK Vue frontend
- GASOK Spring Boot backend
- GIWA testnet integration
- Midnight Local Node
- Midnight Indexer
- Midnight Proof Server
- GASOK Mock Attestation API
- Midnight CLI
- Midnight Local Proof Bridge on `127.0.0.1:4200`

Do not deploy Midnight contracts to Preprod or Mainnet.

Do not replace Vue with React.

The official Midnight React examples may be used only as implementation references.

Do not modify existing GIWA Solidity contracts unless Midnight integration explicitly requires it.

## Phase Completion Criteria

### Phase 1 — Official Midnight Example

- Compact contract compiles successfully.
- Midnight Node is running.
- Midnight Indexer is running.
- Midnight Proof Server is running.
- Official CLI can deploy the contract.
- Mock Attestation Provider can be registered.
- ZK Proof generation succeeds.
- Midnight transaction submission succeeds.
- Public contract state can be queried.

### Phase 2 — GASOK Domain Mapping

- GASOK financial fields are used instead of ZK Loan credit fields.
- Financial data is passed as private input.
- Provider signature validation succeeds.
- Invalid provider signatures are rejected.
- Eligibility rules are enforced by the Compact Circuit.
- Raw financial values are not stored publicly.
- Verification results can be queried through CLI.

### Phase 2.5 — GIWA Receivable-Subject Binding

- [x] The local contract seals GIWA chain `91342` and ReceivableFinance
  `0x0f264334f98BA0d22f7Fc6Bb901a5Fa36158a315`.
- [x] The Mock Provider resolves the canonical Seller/Buyer wallet by uint256
  receivable ID through GIWA RPC.
- [x] The signed eight-field message binds private values,
  company-commitment hash, GIWA context, Midnight deployment, provider, and
  policy.
- [x] Seller and Buyer use different opaque lookup keys for the same receivable.
- [x] Exact same-key replay is rejected.
- [x] CLI E2E produced Seller `true` and Buyer `false` for receivable `#1`.
- [ ] Independent actors, refresh/expiry policy, and secure capability delivery
  remain future work.

### ADR-017 — Provider 2 Role-Wallet Authorization

- [x] The Provider issues a bounded, random, two-minute EIP-712 challenge and
  consumes it on the first attestation attempt.
- [x] The CLI keeps raw financial values and hidden salt private while handing
  only the typed request to Vue and accepting a one-line response.
- [x] The dev-only `/midnight/authorize` route validates exact context, selects
  the canonical MetaMask EOA, signs, and verifies the recovered signer.
- [x] Provider 2 re-resolves the GIWA role, recomputes the private commitment,
  and recovers the EOA before issuing the unchanged Schnorr attestation.
- [x] Provider 1 is displayed as legacy without wallet authorization; Compact
  and Midnight are not described as independently verifying EIP-712.
- [x] Register Provider 2 on the current replacement deployment and confirm one
  registered Provider in public contract state.
- [x] Execute a real Seller MetaMask authorization followed by the full local
  attestation/proof/transaction/Indexer E2E.

### Phase 3 — Vue Integration

- [x] Existing Vue project structure is preserved.
- [x] React dependencies are not introduced.
- [x] Midnight code is isolated in dedicated services or composables.
- [x] A dev-only `/midnight` page resolves one manually pasted Proof capability
  through the localhost-only adapter without wallet, proof, or private-state
  access.
- [x] The adapter pins the approved Phase 2.5 contract and exposes only exact
  `POST /v1/eligibility-results/resolve`; anonymous result enumeration is absent.
- [x] The page distinguishes Provider 2 wallet-authorized issuance from Provider
  1 legacy results without claiming current capability-presenter wallet control,
  company identity, bank verification, data truth, funding approval, or a
  Funding gate.
- [x] Live API smoke resolved receivable `#1` Seller `true` and Buyer `false`,
  and rejected a tampered capability with HTTP 400.
- [x] Actual development-browser capability submissions resolved Seller `true`
  and Buyer `false`, including the valid-proof explanation for `false`. This
  verifies the read path only, not browser proof submission.
- [x] A separate development-only `/midnight/authorize` tool performs the
  Provider 2 MetaMask signing handoff without raw financial values,
  Attestation/Spring/Midnight HTTP requests, browser persistence, or Funding
  integration.
- [ ] Add direct exhaustive unit tests for the complete pre-existing capability
  validation/response module and role-authorization schema/real-signer module.
  The focused proof-flow suite covers its mocked integration boundary,
  timeout/abort, races, same-origin configuration, and both route flags.
- [x] Choose and approve ADR-018's trusted local Proof Bridge to reuse the
  proven CLI identity/state. The choice is not based on a Lace limitation;
  current official Local Dev supports Lace on `undeployed`.
- [x] Complete the loopback port-4200 Bridge with exact body-only, one-shot,
  single-active proof sessions, a common CLI/Bridge private-state lock, bounded
  errors/timeouts, idempotent cleanup attempts after both success and failure,
  and mandatory stale-witness sanitization before a later prepare.
- [x] Expire unsigned prepared sessions on an internal deadline timer, discard
  their tuple, and free the single active slot without requiring another poll.
- [x] Purge terminal capability/error/status records automatically after 60
  seconds without waiting for another request.
- [x] Bound startup Indexer preflight to 10 seconds, keep the server closed on
  failure, cache the sealed GIWA configuration, and issue no per-challenge
  Indexer query while raw inputs exist. The non-abortable SDK may leave one
  timed-out startup query internally pending, but it has seen no raw tuple.
- [x] Preserve the capability immediately after transaction finalization;
  expose `complete` and recover delayed visibility through resolver-only retry
  without proof resubmission.
- [x] Complete the development-only `/midnight/prove` route with transient
  private inputs, immediate post-challenge clearing, explicit MetaMask action,
  status polling, no automatic submission retry, and independent read-API
  resolution of the returned capability.
- [x] Run the focused Vue proof-flow checks on Node 24.19.0: 8 files / 36 tests,
  changed-file ESLint/Oxlint/Prettier, the production Vite build, and a high-
  severity npm audit with zero findings. These do not complete
  the broader legacy capability/authorization test item above.
- [x] Confirm the production artifact has zero proof marker matches and run a
  live Seller `#1` Vue → Bridge challenge smoke; after challenge creation all
  four private values were absent from DOM/captured console output.
- [ ] Repeat the live browser path with MetaMask. The in-app browser lacked a
  provider, so signing, proof submission, transaction, and resolution were not
  exercised by that smoke.
- [x] On Node 22.21.1, verify 20 files / 242 tests passed with 1 optional
  environment file/test skipped, plus CLI typecheck/build, diff-check, and a
  high-severity npm audit with zero findings after the locked Restify transitive
  overrides.
- [ ] Run a real Seller and Buyer browser-triggered local Bridge E2E before
  marking the Phase 3B runtime complete.
- [ ] Design secure capability delivery and verifier access before any remote or
  multi-user use; direct loopback response solves only the local learning PoC.
- [ ] Consider direct Vue + Lace only as a separately approved self-custody
  replacement with participant/private-state migration.

ADR-017 verification includes the complete Attestation API suite `72/72`, CLI
`60` with `1` optional environment E2E skipped, Vue lint/build checks, and the
separate live Seller Provider 2 MetaMask-to-Midnight transaction at block
`2854`. This proves the local CLI-submitted flow, not browser proof submission,
financial-data truth, or a Funding gate.
