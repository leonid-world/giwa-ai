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
Midnight is inserted only as an additional financial eligibility verification step.

### Planned Integrated Business Flow

This is a future Phase 3 product flow, not current Phase 2 enforcement. The
CLI-only proof result does not yet authorize or block any GIWA funding action.
Making Midnight eligibility a GIWA funding gate requires separate approval and
an architectural decision after the CLI and Vue proof flows work.

Current Phase 3A is narrower than this future flow: an isolated, development-only
Vue page reads the two CLI-created public results through a local read-only
adapter. It does not collect financial values, create proofs, identify a GASOK
company, or affect Funding.

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
Seller 재무정보 입력
↓
GASOK Mock Attestation API가 재무정보에 서명
↓
Seller가 Attested Financial Data 수신
↓
Midnight Witness가 비공개 재무정보를 Compact Circuit에 전달
↓
Compact Circuit이 다음 항목 검증

- 등록된 Attestation Provider의 서명인지 확인
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
  Indexer를 통해 Eligibility 결과 조회
  ↓
  GASOK Frontend에서 Midnight Verification 상태 표시
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
9. Integrate the working Midnight flow into the existing Vue frontend.
10. Integrate Spring Boot only if backend coordination is required.

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

The initial Phase 2 public output may expose only:

- fundingEligibility
- providerId
- policyVersion
- pseudonymous verification commitment

Risk tiers, maximum funding ratios, issued/expiry timestamps, and an actual
GASOK-company binding are deferred until their policies are approved.

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
- that the Compact Circuit was executed correctly
- that the Zero-Knowledge Proof is valid

Midnight does not independently verify whether the real-world financial data itself is truthful.

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

### Phase 3 — Vue Integration

- [x] Existing Vue project structure is preserved.
- [x] React dependencies are not introduced.
- [x] Midnight code is isolated in dedicated services or composables.
- [x] A dev-only `/midnight` page reads the CLI-proven public result through a
  localhost-only adapter without wallet, proof, or private-state access.
- [x] Mock-attested, company-unbound, and non-funding-gate limitations are shown.
- [ ] Choose and approve a local `undeployed` browser submission architecture;
  official Lace local signing is unavailable and Preprod is out of scope.
- [ ] Only after that decision, verify attestation, proof generation, transaction
  submission, and Indexer refresh from Vue end to end.
