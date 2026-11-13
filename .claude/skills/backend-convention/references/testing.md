# 테스트 규칙

- 새 유스케이스에는 최소 한 개 이상의 테스트를 작성한다.
- 버그 수정 시 재현 테스트를 먼저 추가한다.
- Domain 규칙은 단위 테스트로 검증한다.
- Repository 쿼리는 PostgreSQL Testcontainers 기반으로 검증한다.
- API 계약은 `@WebMvcTest` 또는 통합 테스트로 검증한다.
- PostgreSQL/Flyway 프로젝트에서 H2만으로 DB 테스트를 통과 처리하지 않는다.
- 테스트 메서드명은 영어로 작성한다.
- 테스트 의도는 `@DisplayName`에 "조건에서 행위하면 결과한다" 형태의 한글 문장으로 작성한다.
- given / when / then은 빈 줄로 구분한다.

```java
@Test
@DisplayName("종료 시각이 시작 시각보다 이르면 근무 일정을 생성할 수 없다")
void create_whenEndTimeIsBeforeStartTime_throwsInvalidWorkTimeException() {
    // given
    CreateWorkScheduleRequest request = new CreateWorkScheduleRequest(
            LocalDate.now(),
            LocalTime.of(18, 0),
            LocalTime.of(9, 0)
    );

    // when & then
    assertThatThrownBy(() -> workScheduleService.create(1L, request))
            .isInstanceOf(InvalidWorkTimeException.class);
}
```
