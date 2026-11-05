# 네이밍과 코드 스타일

## 목차

- 네이밍 규칙
- 메서드 이름
- 코드 스타일

## 네이밍 규칙

| 대상 | 규칙 | 예시 |
|---|---|---|
| 패키지 | 소문자 | `member`, `commute` |
| 클래스, Enum, Record | PascalCase | `MemberResponse` |
| 메서드, 변수, 필드 | camelCase | `memberId`, `findMember` |
| 상수 | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` |
| boolean | is, has, can, should | `isActive`, `canEdit` |
| 컬렉션 | 복수형 | `members`, `workSchedules` |
| ID | `{resource}Id` | `memberId`, `organizationId` |

## 메서드 이름

| 목적 | 규칙 | 예시 |
|---|---|---|
| 생성 | create | `createMember` |
| 조회 | find, get, load | `findMembers`, `loadMember` |
| 수정 | update, change | `updateProfile`, `changeTime` |
| 삭제 | delete, remove | `deleteMember` |
| 검증 | validate, is, has | `validateWorkTime`, `isAvailable` |
| 권한 확인 | authorize, validatePermission | `validateEditPermission` |

- `get`: 이미 보유한 값에 접근할 때 사용한다.
- `find`: 없을 수 있는 조회에 사용한다.
- `load`: 없으면 예외를 던지는 조회에 사용한다.
- `process`, `handle`, `manage`, `execute`처럼 의미가 모호한 이름은 사용하지 않는다.
- `mgr`, `svc`, `dto`, `util`, `info`, `data` 같은 축약어는 이름의 핵심 단어로 사용하지 않는다.

## 코드 스타일

- Java 17을 기준으로 한다.
- 들여쓰기는 공백 4칸, 한 줄 최대 길이는 120자다.
- 중괄호는 K&R 스타일을 사용한다.
- wildcard import를 사용하지 않는다.
- import 순서는 java → jakarta → Spring → 외부 라이브러리 → 프로젝트 내부 패키지. 그룹 사이에는 한 줄을 비운다.
- Lombok은 `@Getter`, `@RequiredArgsConstructor`를 제한적으로 사용한다. Entity에 `@Data`, `@Setter`, `@ToString`을 사용하지 않는다.
- 컬렉션과 Optional에 null을 사용하지 않는다.
- Stream은 변환, 필터, 집계가 읽기 쉬울 때만 사용한다. 저장, 외부 API 호출 등 부수 효과가 있는 작업은 명시적 반복문을 사용한다.
- 주석은 코드가 설명하지 못하는 "왜"를 작성한다.
