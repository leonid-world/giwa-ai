# GIWA Receivable Financing MVP

## Overview

GIWA 기반 RWA(Real World Asset) 매출채권 토큰화 MVP.

매출채권을 ERC-721 NFT로 토큰화하고,
제3의 Funder가 자금을 공급한 뒤
Buyer가 채권 금액을 상환하는 Supply Chain Finance 데모를 구현한다.
현재 MVP 컨트랙트는 만기 시점 자체를 강제하지 않는다.

본 프로젝트는 실제 금융서비스가 아닌
GIWA Hackathon용 MVP이다.

---

## Goals

- 일반 로그인
- MetaMask 연동
- 기업과 Wallet Mapping
- 매출채권 등록
- Buyer 검증
- ERC721 토큰화
- Funding
- Repayment

---

## Current Stage

End-to-end MVP lifecycle completed.

Replacement contracts deployed and source-verified with the reproducible
Hardhat configuration. Application address rollout and a fresh lifecycle remain.

---

## Tech Stack

Frontend

- Vue3
- Vite
- Pinia
- ethers.js

Backend

- Spring Boot
- Java17
- MyBatis
- MySQL

Blockchain

- Solidity
- OpenZeppelin
- GIWA Sepolia

---

## Out of Scope

- Real Payment
- KYC
- Credit Scoring
- Liquidity Pool
- Multiple Funders
- Oracle
- Production Security
