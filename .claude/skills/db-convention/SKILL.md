---
name: db-convention
description: 정보운영팀 백엔드 PostgreSQL DB 컨벤션. 테이블·컬럼·키·제약 이름과 타입, 감사 컬럼, Enum, Flyway 마이그레이션 작성, JPA 엔티티 매핑 규칙을 담는다. 테이블 설계, ERD 수정, 마이그레이션 SQL 작성·수정, DB 변경이 있는 PR 검토 때 사용한다.
---

# DB 컨벤션

대상은 PostgreSQL DB와 이를 관리하는 Flyway 마이그레이션 SQL, JPA 엔티티다. 누가 테이블을 추가하거나 고쳐도 같은 모양이 되게 한다.

## 기본 원칙

- DB 구조의 기준은 Flyway SQL이다. JPA가 테이블을 만들거나 고치지 않는다(`ddl-auto: validate`). ERD Cloud와 문서는 Flyway SQL을 따라 갱신한다.
- DB가 지킬 수 있는 규칙은 DB에 건다. FK, UNIQUE, CHECK, NOT NULL을 코드에만 맡기지 않는다.
- 이름만 보고 의미와 타입을 알 수 있게 한다. 접미사로 타입을 드러낸다.
- 예외를 만들지 않는다. 규칙에 안 맞는 경우가 생기면 사용자에게 알리고 규칙 수정을 제안한다.

## 상세 규칙

작업에 해당하는 파일만 읽는다.

- **테이블 설계, ERD 수정** (이름, 테이블, 키, 컬럼, 감사 컬럼, 타입, Enum, 제약 이름, COMMENT): [references/table-design.md](references/table-design.md)
- **Flyway 마이그레이션 파일 작성**: [references/flyway-migration.md](references/flyway-migration.md)
- **JPA 엔티티 매핑**: [references/jpa-mapping.md](references/jpa-mapping.md)

## 작업 순서

```
- [ ] 1. 상세 규칙 중 작업에 해당하는 파일 읽기
- [ ] 2. 기존 마이그레이션과 마지막 버전 번호 확인
- [ ] 3. 작성
- [ ] 4. 체크리스트로 검토
- [ ] 5. 보고
```

1. 상세 규칙 중 작업에 해당하는 파일을 읽는다.
2. `src/main/resources/db/migration/` 아래 기존 마이그레이션을 읽고 현재 스키마와 마지막 버전 번호를 확인한다. 번호는 담당자 한 명이 관리하므로 새 번호를 쓰기 전에 사용자에게 확인한다.
3. SQL과 엔티티를 작성한다.
4. 아래 체크리스트로 검토한다. 걸리는 것이 있으면 고친 뒤 다시 검토한다.
5. 추가·변경한 테이블과 컬럼, 마이그레이션 파일명, 실행 여부를 보고한다.

## 체크리스트

- 마이그레이션 파일 이름이 `V{번호}__{설명}.sql` 형식이고, 이미 머지된 파일은 고치지 않았다.
- 테이블·컬럼 이름이 snake_case, 단수형, 예약어 아님, 접미사 규칙을 따른다.
- ID를 저장하는 컬럼에 FK가 있고, FK 컬럼에 인덱스가 있다.
- 모든 제약·인덱스에 이름을 지정했다.
- NULL 허용 컬럼에 NULL의 의미를 COMMENT로 적었다.
- Enum 컬럼에 그 컬럼 전용 CHECK가 있고, Java enum과 값이 같다.
- 테이블·컬럼 COMMENT를 달았다.
- 로컬 PostgreSQL에서 빈 DB부터 마이그레이션을 끝까지 실행했고, `ddl-auto: validate`로 앱이 뜬다.
- 테이블 명세와 ERD Cloud를 같이 갱신했다.
