# Entity, DTO, JPA 규칙

## 목차

- Entity 규칙
- DTO 규칙
- JPA 및 DB 규칙

## Entity 규칙

Entity는 DB 상태와 도메인 규칙을 관리한다.

- 모든 필드는 private으로 작성한다.
- 생성자는 protected로 작성한다.
- public setter를 만들지 않는다. 상태 변경은 의미 있는 메서드로 제공한다.
- `@Data`, `@Setter`, `@ToString`은 Entity에 사용하지 않는다.
- 연관관계는 `FetchType.LAZY`를 사용한다. `FetchType.EAGER`는 사용하지 않는다.
- Entity 생성 시 필요한 값 검증은 생성 메서드 또는 생성자에서 수행한다.

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class WorkSchedule {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long workScheduleId;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id", nullable = false)
    private Member member;

    private LocalDate workDate;
    private LocalTime startTime;
    private LocalTime endTime;

    public static WorkSchedule create(
            Member member,
            LocalDate workDate,
            LocalTime startTime,
            LocalTime endTime
    ) {
        validateTime(startTime, endTime);

        WorkSchedule workSchedule = new WorkSchedule();
        workSchedule.member = member;
        workSchedule.workDate = workDate;
        workSchedule.startTime = startTime;
        workSchedule.endTime = endTime;

        return workSchedule;
    }

    public void changeTime(LocalTime startTime, LocalTime endTime) {
        validateTime(startTime, endTime);

        this.startTime = startTime;
        this.endTime = endTime;
    }

    private static void validateTime(LocalTime startTime, LocalTime endTime) {
        if (!startTime.isBefore(endTime)) {
            throw new InvalidWorkTimeException();
        }
    }
}
```

## DTO 규칙

- Request와 Response DTO를 분리한다.
- Entity를 API 요청 또는 응답으로 직접 사용하지 않는다.
- HTTP DTO는 record 사용을 기본으로 한다.
- Request DTO에서는 입력값의 형식과 필수 여부를 검증한다.
- DB 조회가 필요한 검증, 권한 검증, 비즈니스 규칙 검증은 Service 또는 Entity에서 처리한다.
- Response에는 클라이언트에 필요한 데이터만 포함한다.
- 비밀번호, 토큰 원문, 인증 헤더, 내부 감사 정보는 응답에 포함하지 않는다.

```java
public record CreateWorkScheduleRequest(
        @NotNull LocalDate workDate,
        @NotNull LocalTime startTime,
        @NotNull LocalTime endTime
) {
}
```

DTO 이름:

| 구분 | 규칙 | 예시 |
|---|---|---|
| 생성 요청 | `Create{Resource}Request` | `CreateMemberRequest` |
| 수정 요청 | `Update{Resource}Request` | `UpdateMemberRequest` |
| 단건 응답 | `{Resource}Response` | `MemberResponse` |
| 상세 응답 | `{Resource}DetailResponse` | `MemberDetailResponse` |
| 목록 항목 | `{Resource}SummaryResponse` | `WorkScheduleSummaryResponse` |

`Dto`, `VO`, `Info`, `Data`처럼 의미가 모호한 이름은 사용하지 않는다.

## JPA 및 DB 규칙

- DB 구조는 DB 컨벤션(db-convention)과 Flyway SQL이 기준이다.
- 스키마 변경은 Flyway migration으로만 한다. `ddl-auto=validate`를 사용한다.
- 이미 `staging`에 머지된 migration 파일은 수정하지 않는다.
- Enum은 `@Enumerated(EnumType.STRING)`만 사용한다.
- Enum 변경 시 Java Enum과 DB CHECK 제약을 같은 PR에서 변경한다.
- 시점 데이터는 `Instant` 또는 `OffsetDateTime`, 날짜는 `LocalDate`, 하루 안의 시각은 `LocalTime`을 사용한다.
- DB 컬럼명은 snake_case, Java 필드명은 camelCase를 사용한다.
