# SimpleDb & Sql 테스트 케이스 정리

이 문서는 `SimpleDb`와 `Sql` 클래스 테스트 케이스를 기반으로 작성되었습니다.  
각 테스트 이름과 번호, 목적, 주의사항 및 트러블슈팅을 포함합니다.

---

## 1. 테스트 환경 설정
- DB 연결: `localhost`, `root` / `root123414`, DB: `mysql-1`
- SQL 로그 출력: `devMode = true`
- 테이블 초기화
  - `createArticleTable()` : article 테이블 생성
  - `truncateArticleTable()` : 테스트 전 데이터 삭제
  - `makeArticleTestData()` : 6개의 샘플 데이터 삽입

---

## 2. CRUD 테스트

### T001 - insert
- 목적: `Sql.insert()`로 새로운 글 추가, AUTO_INCREMENT PK 반환 확인
- 특징: `append()`를 이용한 SQL 조합

### T002 - update
- 목적: 다중 행 수정 (`WHERE id IN (?, ?, ...)`)
- 확인: `update()` 반환값 = 수정된 row 수

### T003 - delete
- 목적: 다중 행 삭제 (`WHERE id IN (?, ?, ...)`)
- 확인: `delete()` 반환값 = 삭제된 row 수

---

## 3. SELECT 테스트

### T004 - selectRows
- 목적: 여러 행 조회 (`List<Map<String,Object>>`)
- 확인: 컬럼 타입 (`LocalDateTime`, `boolean`, `String`, `Long`), 값 확인

### T005 - selectRow
- 목적: 단일 행 조회 (`Map<String,Object>`)
- 확인: 값과 타입 일치 확인

### T006 - selectDatetime
- 목적: 단일 컬럼 조회, `LocalDateTime` 반환 (`SELECT NOW()`)
- 확인: 현재 시간과 비교, 초 단위 오차 ≤ 1

### T007 - selectLong
- 목적: 단일 Long 값 조회
- SQL 예시: `SELECT id FROM article WHERE id = 1`

### T008 - selectString
- 목적: 단일 String 값 조회
- SQL 예시: `SELECT title FROM article WHERE id = 1`

### T009/T010/T011 - selectBoolean
- 목적: 단일 Boolean 값 조회
- 예시: `SELECT isBlind`, `SELECT 1=1`, `SELECT 1=0`

### T012 - select, LIKE 사용법
- 목적: 조건 조회 + LIKE
- 예시: `SELECT COUNT(*) FROM article WHERE id BETWEEN ? AND ? AND title LIKE CONCAT('%', ?, '%')`

### T013 - appendIn
- 목적: `IN` 절 지원
- SQL: `WHERE id IN (?)` → `?, ?, ?` 자동 변환

### T014 - selectLongs, ORDER BY FIELD
- 목적: 특정 순서대로 정렬 후 Long 리스트 반환
- 주의: `appendIn()` + `ORDER BY FIELD` 사용

---

## 4. 객체 매핑 테스트

### T015 - selectRows, Article
- 목적: Map → `Article` 객체 리스트 매핑
- 확인: 컬럼값 → 필드 자동 변환, 타입 호환 확인

### T016 - selectRow, Article
- 목적: 단일 Map → `Article` 객체 매핑
- 확인: 필드값 일치 확인

---

## 5. 멀티스레딩 테스트

### T017 - use in multi threading
- 목적: 여러 스레드에서 동시에 DB 조회
- 방법: `ExecutorService` + `CountDownLatch`
- 주의사항:
  1. Thread-local Connection 사용 필수
  2. 작업 종료 후 `simpleDb.close()` 호출
  3. 트랜잭션 시 스레드별 Connection 독립성 확인
- 트러블슈팅:
  - ThreadLocal Connection 미사용 → 동시 접근 충돌
  - Connection 미종료 → 커넥션 누수

---

## 6. 트랜잭션 테스트

### T018 - rollback
- 목적: 트랜잭션 시작 후 삽입 → `rollback()`
- 확인: 데이터 변경 이전 상태 유지
- 주의: `startTransaction()` 후 반드시 `rollback()` 호출

### T019 - commit
- 목적: 트랜잭션 시작 후 삽입 → `commit()`
- 확인: 데이터 변경 반영
- 주의: `startTransaction()` 후 반드시 `commit()` 호출

---

## 7. 트러블슈팅 (Troubleshooting)

### 7-1. SQL 실행 오류
- 원인: 문법 오류, 파라미터 개수 불일치
- 해결: `devMode = true` 로 SQL 로그 확인, `append()/appendIn()` 사용 시 `?` 개수 체크

### 7-2. 멀티스레딩 문제
- 원인: ThreadLocal Connection 미사용, Connection 미종료
- 해결: 스레드별 `genSql()` 호출 + 종료 시 `close()`

### 7-3. 객체 매핑 실패
- 원인: 필드명/컬럼명 불일치, 타입 불일치
- 해결: 필드명과 컬럼명 일치, primitive/Wrapper 변환 확인

### 7-4. 트랜잭션 미반영
- 원인: commit/rollback 호출 누락, ThreadLocal Connection 문제
- 해결: `startTransaction()` 후 반드시 `commit()` 또는 `rollback()` 호출

### 7-5. 테스트 실패 시 데이터 초기화
- 원인: 이전 테스트 데이터 잔존
- 해결: `truncateArticleTable()` 또는 테스트마다 초기 데이터 삽입
