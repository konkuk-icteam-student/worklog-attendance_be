# Flyway 마이그레이션 작성 규칙

## 목차

- 파일 위치
- 파일 이름
- 규칙
- 파일 안 순서

## 파일 위치

도메인별 폴더 `src/main/resources/db/migration/{도메인}/`에 둔다.

| 폴더 | 담는 테이블 |
|---|---|
| `member/` | `organization`, `member`, `member_profile`, `email_verification_code`, `password_reset_token` |
| `commute/` | `workplace`, `work_*`, `todo`, `todo_completion`, `notification`, `notification_check_state` |
| `faq/` | FAQ (나중에) |

- 나중에 앱을 컨테이너 단위로 나눌 때 그 도메인의 마이그레이션 폴더만 옮기면 되게 하려고 폴더를 나눈다.
- 테이블을 만들거나 고치는 파일은 그 테이블이 속한 도메인 폴더에 둔다. 한 파일에서 두 도메인의 테이블을 함께 고치지 않는다.
- Flyway 설정은 하나로 두고 모든 폴더를 읽게 한다.

```yaml
spring:
  flyway:
    locations: classpath:db/migration/member, classpath:db/migration/commute
```

## 파일 이름

형식은 `V{번호}__{설명}.sql`.

- 번호는 폴더와 상관없이 전체에서 하나로 이어진다. Flyway는 모든 폴더의 파일을 번호 순서대로 한 줄로 실행한다. 예: `member/V1__create_member_tables.sql`, `commute/V2__create_commute_tables.sql`, `commute/V3__add_work_schedule_history.sql`
- 다른 도메인 테이블을 가리키는 FK가 있으면 그 테이블을 만드는 파일이 더 앞 번호여야 한다. 예: commute의 `member_id` FK → member 테이블이 먼저
- 설명은 영어 snake_case 동사로 시작한다. 예: `V3__add_work_schedule_history.sql`, `V4__alter_todo_drop_todo_date.sql`
- 번호 관리는 한 명이 맡는다. 테이블 변경이 필요하면 담당자에게 요청하거나, PR에서 담당자가 번호를 확인한다. 두 사람이 같은 번호를 쓰면 나중에 머지하는 쪽이 번호를 올린다.

## 규칙

- 한번 `staging`에 머지된 마이그레이션 파일은 절대 고치지 않는다. 수정이 필요하면 새 파일을 추가한다. Flyway가 체크섬으로 변경을 감지해 앱이 뜨지 않는다.
- 파일 하나에 목적 하나. 테이블 추가와 다른 테이블 수정을 한 파일에 섞지 않는다.
- 스키마 변경(DDL)과 데이터 변경(DML)은 가능하면 파일을 나눈다. 같이 있어야 하는 경우(Enum 값 변경 등)는 예외다.
- 운영 데이터가 있는 테이블을 바꿀 때는 단계를 나눈다. 한 번에 RENAME/DROP하면 배포 중인 구버전 앱이 깨질 수 있다.
  - 컬럼 이름 변경 예: 새 컬럼 추가 → 데이터 복사 → 코드 전환 배포 → 옛 컬럼 삭제
- NOT NULL 컬럼을 기존 테이블에 추가할 때는 DEFAULT를 주거나, 추가 → 값 채우기 → SET NOT NULL 순서로 한다.
- 마이그레이션 안의 SQL은 PostgreSQL 문법으로 쓴다. 로컬·테스트도 PostgreSQL(Docker 또는 Testcontainers)에서 실행한다. H2는 PostgreSQL 전용 문법(EXCLUDE, timestamptz 동작 등)과 맞지 않는다.
- 운영 DB에 직접 DDL을 실행하지 않는다. 긴급 수정도 마이그레이션 파일로 만든 뒤 배포한다.

## 파일 안 순서

```sql
CREATE TABLE work_schedule_history ( ... );          -- 1. 테이블

ALTER TABLE work_schedule_history ADD CONSTRAINT ...; -- 2. 제약 (FK, UNIQUE, CHECK)

CREATE INDEX idx_work_schedule_history_member_id_work_date
  ON work_schedule_history (member_id, work_date);    -- 3. 인덱스 (필요할 때만)

COMMENT ON TABLE work_schedule_history IS '...';      -- 4. COMMENT
```
