---
title: "Projects"
permalink: /projects/
author_profile: true
sidebar:
  nav: "sidebar-category"
---

제가 작업한 주요 프로젝트들을 소개합니다.

## AromaBank - 향료 관리 시스템

**기간**: 2025.04 ~ 2025.10
**역할**: 백엔드 리드 개발자 (기여도 100%)  
**기술 스택**: Java 8+, Spring Framework, Spring Security, MyBatis, MySQL, JSP, jQuery, Apache POI  
**핵심 목표**: 데이터 정제, 규제 대응 자동화, MES 기반 실시간 공정 관리

향료 제조업체를 위한 종합 관리 시스템으로, 제품 정보부터 안전성 문서까지 통합 관리하는 웹 애플리케이션입니다.

### 주요 기능
- 다차원 동적 검색 – MyBatis Dynamic SQL로 10개 이상의 복합 필터를 지원, 인덱스/프로시저 적용으로 검색 시간 50% 단축
- 실시간 규제 검증 – IFRA 표준에 따라 배합 변경 시 알레르기 물질 한도 초과를 자동 계산하고 경고
- 복합 원료 역산 – 다단계 중첩 원료(Mixture) 내 유해 성분을 재귀적으로 합산해 안전성 검증 정확도 향상
- 엑셀 자동화 엔진 – Apache POI로 데이터 양에 따라 동적 행/스타일을 조절하고 COA·IFRA 인증서 등 법적 문서를 픽셀 단위로 출력
- 데이터 표준화 – 레거시 엑셀을 3NF로 정규화, 공통 코드 시스템을 도입해 중복 제거 및 무결성 확보
- 협업 워크플로우 – R&D → 생산 승인까지 전자결재 파이프라인과 이력 추적(승인/레시피 로그) 100% 기록
- MES & 재고 관리 – FIFO 및 Lot 추적 Stored Procedure로 재고 정확도 99% 확보, 실시간 대시보드·KPI 리포트 제공
- 사용자 권한/보안 – Spring Security 기반 인증·인가, 사업장/거래처 단위 접근 제어

### 기술적 성과
- 검색/조회 성능 50% 개선(인덱스, 프로시저, Dynamic SQL)으로 대용량 데이터 운영 안정화
- 법적 문서(COA/IFRA) 자동화와 디지털 결재로 부서 간 처리 시간 50% 단축
- 데이터 정규화 및 공통 코드 체계로 유지보수성·데이터 무결성 확보
- 실시간 재고·KPI 대시보드와 추적 로그로 운영 투명성 강화

---

## Logi ERP - 물류 통합 관리 시스템

**기간**: 2023.11 ~ 2024.10 (1년)  
**역할**: 풀스택 개발자  
**기술 스택**: Java, Spring Framework, JSP, MySQL, MyBatis, JavaScript, Stored Procedure, Apache POI

(주) 성진 물류회사의 물류 전반을 관리하기 위한 ERP 시스템으로, 주문·재고·거래·통계 등 주요 업무를 웹 기반으로 통합 관리하는 시스템입니다.

### 주요 기능
- 거래 관리 – 가상계좌 API 및 SDSI 연동으로 실시간 입금 내역 자동 반영 및 DB 연동
- 코드 자동화 – MySQL Trigger를 이용한 거래처·사업장·매출/매입 코드 자동 채번
- 주문/관리 페이지 분리 – 거래처 전용 주문 페이지와 내부 관리 페이지 분리 및 데이터 연동
- 재고 관리 최적화 – 마감 기능 구현으로 전잔고 검색 속도 개선
- 통계 및 리포트 – 매출/매입 데이터 통계화 및 세금계산서, 거래처 원장 일괄 출력 기능
- Excel 기반 기능 – 테이블 데이터 Export 및 통계 데이터 Excel 자동화 처리
- 회전율/수불부 관리 – 재고 흐름 및 회전율 관리 기능 제공
- 검색 성능 최적화 – Procedure 기반 검색 최적화로 대용량 데이터 처리 속도 향상

### 기술적 성과
- Trigger & SP 활용으로 핵심 데이터 자동화 및 기존 프로그램 대비 검색 성능 50% 이상 개선
- 거래처 전용 주문 시스템 분리로 업무 효율성 향상
- 재고관리, 마감 기능으로 데이터 조회 속도 안정화 및 시스템 신뢰성 확보
- 문서 자동화 및 대량 인쇄 기능으로 업무 처리 시간 단축 및 관리 비용 절감

---

## i-rept - 파충류 커뮤니티 & 쇼핑몰 앱

**기간**: 2023.03 ~ 2023.08 (6개월)  
**역할**: 풀스택 개발자  
**기술 스택**: Flutter, Dart, Spring Boot, JPA, Firebase, AWS EC2

기존 PHP 기반 서비스를 Flutter와 Spring Boot, JPA로 전면 재구현한 파충류 커뮤니티·커머스 올인원(Super App) 모바일 애플리케이션입니다.

### 시스템/아키텍처
- Polyglot Persistence – 트랜잭션 데이터는 MySQL, 실시간 채팅은 Firebase Firestore, 대용량 미디어는 AWS S3로 분리해 성능·비용 최적화
- 계층형 백엔드 아키텍처(Controller/Service/Repository)와 Provider 패턴 기반 MVVM으로 유지보수성 강화

### Flutter 개발
- REST API 연동 및 Provider 패턴으로 반응형 UI 구현, 월별 다이어리 데이터 비동기 캐싱으로 스와이프 시 끊김 없는 UX
- 커스텀 TableCalendar로 먹이/탈피/산란 등 7가지 이벤트를 색상별 마커로 시각화, Syncfusion Charts로 체중 변화 그래프 제공
- Firebase Authentication 소셜 로그인(Kakao, 네이버, Apple, Google, Facebook) 및 이메일/비밀번호 로그인
- Firebase Firestore `StreamBuilder` 기반 실시간 1:1 채팅, 구독 범위 최적화로 데이터 사용량 절감
- Firebase Cloud Messaging 푸시 알림, Google Play·App Store 심사 및 배포
- 펫 다이어리, 개인 일기, 거래 페이지 등 커머스·커뮤니티 핵심 기능 구현

### Spring Boot & JPA
- JpaRepository 기반 CRUD 및 QueryDSL 유틸로 동적 검색/필터 처리, 타입 세이프 쿼리로 안정성 확보
- 이미지 최적화 파이프라인(Thumbnailator)으로 S3 업로드 전 해상도/용량 1/10 경량화, 공용 URL 반환
- Firebase Auth/메시징 연동 Callback API 및 Spring Security 기반 인증·인가 적용

### AWS/인프라
- AWS EC2에 백엔드 배포, S3에 이미지 저장, Redis·MySQL로 캐싱/주 데이터 운영

### 성과
- 이미지 최적화 및 비동기 처리로 앱 초기 로딩 속도 3초 미만 달성
- 실시간 채팅·알림을 별도 소켓 서버 없이 Firestore/FCM으로 구현해 운영 비용 절감
- 아키텍처 계층화와 Provider 패턴으로 출시 후 기능 추가 시 회귀 버그 감소

### 앱 링크
- [Google Play](https://play.google.com/store/apps/details?id=com.reptcomunity&hl=ko)
- [Apple App Store](https://apps.apple.com/kr/app/irept/id1492882301)

---

## Next.js 랜딩페이지 재구축

**기간**: 2023.08 ~ 2023.09 (2개월)  
**역할**: 프론트엔드 개발자  
**기술 스택**: Next.js, TypeScript, JavaScript(ES6+), React Hooks, AWS Lightsail

기존 PHP 기반 템플릿으로 제작된 랜딩페이지를 Next.js + TypeScript 기반으로 재구축하여, 성능 최적화 및 사용자 경험을 강화한 반응형 웹 페이지입니다.

### 주요 기능
- 페이지 재구축 – PHP 템플릿 기반 랜딩페이지를 Next.js + TypeScript로 전면 재개발
- 인터랙션 구현 – 바닐라 JS 및 커스텀 React Hooks를 활용한 동적 UI/UX 개발
- 반응형 디자인 – 모바일·웹 뷰 대응을 위한 반응형 레이아웃 구현
- 멀티미디어 최적화 – 다수의 동영상 및 이미지 최적화로 페이지 로딩 속도 개선
- 배포 환경 구축 – AWS Lightsail 환경에서 Next.js 프로젝트 배포 및 운영

### 기술적 성과
- 최신 프레임워크 도입으로 유지보수성 및 확장성 확보
- 반응형 UI 적용으로 다양한 디바이스에서 사용자 경험 향상
- AWS Lightsail 배포 자동화로 운영 효율성 증대

## AI 기반 운동 & 루틴 추천 및 기록 프로그램 (**개인 프로젝트**)

**기간**: 2025.12 ~ 진행중
**기술 스택**: React, Spring Boot, JPA, Redis, Kafka, Python, MySQL, Docker, Github Action

AI 기반의 운동 루틴 기록 & 추천 시스템입니다. 마이크로서비스 아키텍처(MSA)를 기반으로 설계되었으며, 실시간 데이터 처리를 위해 Kafka를, 동시성 제어를 위해 Redis를 사용합니다.  
서버는 개발을 위한 우분투 리눅스 서버와 배포용 AWS 서버를 동시에 운영합니다.  
배포 자동화를 위해 Github Action을 이용해 CI/CD를 자동화합니다.


