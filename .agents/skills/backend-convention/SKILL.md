---
name: backend-convention
description: 정보운영팀 백엔드(Spring Boot, Java 17) 코딩 컨벤션. 패키지 구조, Controller·Service·Repository 책임, Entity·DTO, 예외, 네이밍, 코드 스타일, 테스트 규칙을 담는다. 백엔드 Java 코드를 작성·수정할 때 사용한다. 테이블 설계와 Flyway 마이그레이션은 db-convention을 사용한다.
---

# 백엔드 컨벤션

사람이 쓰든 AI가 쓰든 같은 기능은 같은 구조와 방식으로 구현되게 한다.

## 기본 원칙

- 기능은 도메인 단위로 관리한다.
- Controller, Service, Entity, Repository의 책임을 섞지 않는다.
- Entity를 API 요청·응답에 직접 사용하지 않는다.
- DB 구조의 기준은 Flyway SQL과 DB 컨벤션(db-convention)이다.
- 기존 유사 기능을 먼저 확인하고 같은 패턴을 따른다.
- 구조 선택이나 정책 결정이 필요하거나 규칙에 없는 상황이면, 임의로 정하지 말고 먼저 묻는다.

## 상세 규칙

작업에 해당하는 파일만 읽는다.

- **패키지 구조, 의존성 방향, Controller·Service·Repository, 예외 처리**: [references/architecture.md](references/architecture.md)
- **Entity, DTO, JPA 매핑**: [references/entity-dto.md](references/entity-dto.md)
- **네이밍, 코드 스타일**: [references/naming-style.md](references/naming-style.md)
- **테스트**: [references/testing.md](references/testing.md)

## 작업 순서

아래 체크리스트를 복사해 진행 상황을 표시한다.

```
- [ ] 1. 상세 규칙 중 작업에 해당하는 파일 읽기
- [ ] 2. 기존 유사 기능과 테스트 확인
- [ ] 3. 구현
- [ ] 4. 금지 목록으로 검토
- [ ] 5. 보고
```

1. 상세 규칙 중 작업에 해당하는 파일을 읽는다.
2. 같은 도메인의 기존 코드와 테스트를 확인하고 같은 패턴을 따른다.
3. 구현한다. DB 변경이 필요하면 db-convention을 따른다.
4. 아래 금지 목록으로 검토한다. 걸리는 것이 있으면 고친 뒤 다시 검토한다.
5. 변경한 파일, 설계 이유, 실행한 테스트와 미실행 사유를 보고한다.

## 금지 목록

- `XxxServiceImpl`
- 필드 주입 (생성자 주입만 사용)
- Entity의 `@Data`, `@Setter`, `@ToString`, public setter
- `FetchType.EAGER`, `EnumType.ORDINAL`
- Entity를 Request·Response로 사용
- 업무 도메인 코드를 `global`, `common`, `util`에 두기
- 다른 도메인의 Repository 직접 호출
- Controller에서 Repository 호출, 비즈니스 검증, 트랜잭션, try-catch로 오류 응답 생성
- `RuntimeException` 직접 throw (도메인 예외와 ErrorCode를 쓴다)
- Repository에서 null 반환 (단건은 `Optional`, 목록은 빈 컬렉션)
- 이미 `staging`에 머지된 migration 파일 수정
- wildcard import
- 모호한 이름: `Dto`, `VO`, `Info`, `Data`, `process`, `handle`, `manage`, `execute`
- 테스트 없는 새 유스케이스
