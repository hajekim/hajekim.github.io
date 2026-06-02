---
title: "금융 정산 마이그레이션에서 소수점 정밀도를 보장하는 법: 6종 데이터베이스 실증 가이드"
date: 2026-06-02 00:00:00 +0900
description: "Oracle에서 BigQuery로 대출 이자 정산 쿼리를 이행할 때 발생하는 1원 소수점 오차의 원인을 분석하고, 후순위 나눗셈(Division Deferral) 기법으로 BigQuery·Spanner·PostgreSQL·MySQL·Presto·Trino 6종 데이터베이스 전체에서 100.0000% 정합성을 달성한 실증 가이드"
categories: data
tags: [BigQuery, Oracle, Spanner, PostgreSQL, MySQL, Presto, Trino, "소수점 정밀도", 마이그레이션, "금융 정산"]
---

## 1. 도입

## 2. 마이그레이션 개요

## 3. 대출 이자 수식 및 스키마 이행

## 4. 문제 발견: AS-IS 쿼리의 1원 오차

## 5. 원인 분석: 데이터베이스별 수치 연산 아키텍처

### 5.1 플랫폼별 정밀도 처리 방식 비교

### 5.2 David Goldberg 논문과의 간극

### 5.3 UNION ALL과 묵시적 타입 격하

## 6. 해결: 후순위 나눗셈(Division Deferral)

## 7. 6종 데이터베이스 교차 검증 결과

## 8. 프로덕션 방어 수칙

## 9. 결론
