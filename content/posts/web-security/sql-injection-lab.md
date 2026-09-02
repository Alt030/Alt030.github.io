---
title: "SQL Injection 기초 분석: 입력값 검증부터 대응까지"
date: 2026-09-02
description: 로컬 실습 환경에서 SQL Injection의 원인과 안전한 대응 방법을 확인한 기록
tags:
  - web-security
  - sql-injection
  - lab
---

# SQL Injection 기초 분석

> [!warning] 실습 범위
> 이 글은 직접 구축했거나 명시적으로 허가받은 로컬 실습 환경에서만 재현하는 것을 전제로 합니다.

## 개요

SQL Injection은 사용자 입력이 SQL 문의 구조에 직접 포함될 때 발생할 수 있다. 이번 실습에서는 로그인 API의 입력 처리 과정을 관찰하고, 오류 원인과 대응 방법을 정리했다.

## 실습 환경

| 항목 | 내용 |
| --- | --- |
| 대상 | 로컬 학습용 웹 애플리케이션 |
| 주소 | `http://127.0.0.1:3000` |
| 프록시 | Burp Suite Community Edition |
| 데이터베이스 | SQLite |

## 요청 확인

Burp Suite로 로그인 요청을 가로채 입력값이 JSON 형식으로 전달되는 것을 확인했다.

![[assets/burp-request-example.svg|Burp Suite에서 로그인 요청을 확인하는 예시 화면]]

### HTTP Request

```http
POST /api/login HTTP/1.1
Host: 127.0.0.1:3000
Content-Type: application/json
Content-Length: 44

{"username":"test","password":"test1234"}
```

### HTTP Response

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{"message":"아이디 또는 비밀번호가 올바르지 않습니다."}
```

## 원인 분석

서버에서 문자열 연결로 SQL 문을 구성하면 입력값이 데이터가 아니라 SQL 구문의 일부로 해석될 가능성이 생긴다.

```sql
SELECT id, username
FROM users
WHERE username = '사용자 입력'
  AND password = '사용자 입력';
```

단일 따옴표를 입력했을 때 서버 오류가 발생한다면 SQL 구문 오류가 외부 입력에 의해 유발되는지 추가로 확인해야 한다. 다만 오류 발생만으로 취약점을 확정하지 않고, 서버 코드와 쿼리 처리 방식을 함께 검토해야 한다.

```http
POST /api/login HTTP/1.1
Host: 127.0.0.1:3000
Content-Type: application/json

{"username":"'","password":"test"}
```

## 터미널에서 재현

동일한 요청을 `curl`로 전송해 프록시 외부에서도 응답이 동일한지 확인했다.

```bash
curl --request POST 'http://127.0.0.1:3000/api/login' \
  --header 'Content-Type: application/json' \
  --data '{"username":"test","password":"test1234"}'
```

## 대응 방안

가장 중요한 조치는 SQL 문과 사용자 입력을 분리하는 것이다. 입력값은 Prepared Statement의 매개변수로 전달하고, 상세한 데이터베이스 오류는 사용자에게 노출하지 않는다.

```sql
SELECT id, username
FROM users
WHERE username = ?
  AND password_hash = ?;
```

| 구분 | 취약한 방식 | 권장 방식 |
| --- | --- | --- |
| 쿼리 생성 | 문자열 연결 | Prepared Statement |
| 오류 처리 | DB 오류 원문 반환 | 일반화된 오류 메시지와 서버 로그 분리 |
| 비밀번호 | 평문 비교 | 안전한 해시 함수로 검증 |
| 권한 | 애플리케이션 계정에 과도한 권한 | 필요한 권한만 부여 |

## 정리

이번 실습에서 확인한 핵심은 특수문자 자체가 아니라, 외부 입력이 SQL 문의 구조와 분리되어 처리되는지 여부다. 재현 결과와 함께 원인, 영향, 코드 수준의 대응 방안을 기록해야 진단 결과의 설득력이 높아진다.
