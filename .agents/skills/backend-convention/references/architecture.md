# 구조와 계층 규칙

## 목차

- 패키지 구조
- 의존성 방향
- Controller 규칙
- Service 규칙
- Repository 규칙
- 예외 처리 규칙

## 패키지 구조

기능별로 도메인을 나누고, 각 도메인 안에서 동일한 구조를 사용한다.

```
com.better.Commutemate
├── global
│   ├── config
│   ├── error
│   ├── security
│   └── response
│
├── member
│   ├── controller
│   │   └── dto
│   │       ├── request
│   │       └── response
│   ├── service
│   ├── entity
│   ├── repository
│   └── exception
│
├── commute
│   └── (member와 같은 구조)
│
└── notification
    └── (member와 같은 구조)
```

| 패키지 | 책임 |
|---|---|
| `{도메인}/controller` | API 요청·응답 처리, 로그인 사용자 정보 추출, Service 호출 |
| `{도메인}/controller/dto/request` | API 요청값, 입력 형식 검증 |
| `{도메인}/controller/dto/response` | API 응답 데이터 |
| `{도메인}/service` | 비즈니스 로직, 트랜잭션, 권한 및 도메인 규칙 검증 |
| `{도메인}/entity` | DB 테이블과 매핑되는 JPA Entity, Entity 내부 상태 변경 |
| `{도메인}/repository` | DB 조회·저장, Querydsl 쿼리 |
| `{도메인}/exception` | 해당 도메인에서 사용하는 예외와 에러 코드 |
| `global` | 모든 도메인이 함께 쓰는 설정, 보안, 공통 예외 처리, 공통 응답 |

## 의존성 방향

```
Controller → Service → Repository → Database
                  ↓
                Entity
```

- Controller는 Service만 호출한다.
- Service는 Repository와 Entity를 사용한다.
- Repository는 DB 조회·저장만 담당한다.
- Entity는 API Request/Response로 직접 사용하지 않는다.
- 다른 도메인의 Repository를 직접 호출하지 않는다. 다른 도메인의 기능이 필요하면 해당 도메인의 Service를 호출한다.
- `global`, `common`, `util`에 업무 도메인 로직을 넣지 않는다.

## Controller 규칙

Controller는 HTTP 요청과 응답만 담당한다.

담당하는 것: URL 매핑, Request DTO 수신, `@Valid` 기반 입력 검증, 로그인 사용자 정보 추출, Request DTO를 Service에 전달, Response DTO 반환.

금지하는 것: Repository 직접 호출, Entity 직접 수정, 비즈니스 규칙 검증, 트랜잭션 처리, 복잡한 조건문과 반복문, 예외를 직접 잡아 오류 응답 생성.

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/work-schedules")
public class WorkScheduleController {

    private final WorkScheduleService workScheduleService;

    @PostMapping
    public ResponseEntity<WorkScheduleResponse> create(
            @AuthenticationPrincipal LoginMember loginMember,
            @Valid @RequestBody CreateWorkScheduleRequest request
    ) {
        WorkScheduleResponse response = workScheduleService.create(
                loginMember.memberId(),
                request
        );

        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

## Service 규칙

Service는 비즈니스 로직과 트랜잭션을 담당한다.

- 클래스명은 `{도메인}Service` 또는 `{기능}{도메인}Service`로 작성한다. 예: `MemberService`, `WorkScheduleService`, `NotificationService`
- `XxxServiceImpl`은 사용하지 않는다.
- 쓰기 작업에는 `@Transactional`, 조회 작업에는 `@Transactional(readOnly = true)`를 사용한다.
- public 메서드 하나는 하나의 유스케이스만 담당한다.
- Service에서 API 응답 형식을 직접 만들지 않는다.
- Service는 Entity 또는 Response DTO를 반환한다. 프로젝트 내에서 한 방식을 선택해 통일한다.

```java
@Service
@RequiredArgsConstructor
@Transactional
public class WorkScheduleService {

    private final WorkScheduleRepository workScheduleRepository;
    private final MemberRepository memberRepository;

    public WorkScheduleResponse create(
            Long memberId,
            CreateWorkScheduleRequest request
    ) {
        Member member = memberRepository.findById(memberId)
                .orElseThrow(MemberNotFoundException::new);

        WorkSchedule workSchedule = WorkSchedule.create(
                member,
                request.workDate(),
                request.startTime(),
                request.endTime()
        );

        WorkSchedule savedWorkSchedule = workScheduleRepository.save(workSchedule);

        return WorkScheduleResponse.from(savedWorkSchedule);
    }
}
```

## Repository 규칙

Repository는 DB 조회와 저장만 담당한다.

- 단건 조회는 `Optional<T>`를 반환한다.
- 목록 조회는 빈 컬렉션을 반환한다. null을 반환하지 않는다.
- 화면 단의 필터링이나 정렬을 Service에서 메모리로 처리하지 않는다.
- 페이징, 집계, 복잡한 조회는 Querydsl 또는 전용 쿼리로 구현한다.
- Repository는 비즈니스 로직을 포함하지 않는다.

```java
Optional<Member> findByEmail(String email);

List<WorkSchedule> findAllByMemberIdAndWorkDateBetween(
        Long memberId,
        LocalDate startDate,
        LocalDate endDate
);
```

## 예외 처리 규칙

- 예상 가능한 업무 실패는 프로젝트 공통 예외 또는 도메인 예외로 처리한다.
- `RuntimeException`을 직접 던지지 않는다.
- Controller에서 try-catch로 오류 응답을 만들지 않는다.
- HTTP 상태 코드와 오류 응답 생성은 전역 예외 처리기에서 담당한다.
- 예외 메시지에는 비밀번호, 토큰, 인증 헤더, 개인정보를 포함하지 않는다.
- DB 무결성 오류는 적절한 공통 예외 또는 도메인 예외로 변환한다.

```java
public class MemberNotFoundException extends DomainException {

    public MemberNotFoundException() {
        super(MemberErrorCode.MEMBER_NOT_FOUND);
    }
}
```
