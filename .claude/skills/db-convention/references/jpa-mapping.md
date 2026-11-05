# JPA 엔티티 매핑 규칙

| 항목 | 규칙 |
|---|---|
| 스키마 생성 | `spring.jpa.hibernate.ddl-auto: validate`. 엔티티가 DB와 다르면 앱이 뜨지 않게 해서 불일치를 바로 발견한다 |
| 이름 매핑 | Java camelCase ↔ DB snake_case는 Spring 기본 전략이 자동 변환하므로 같은 이름이면 `@Column(name)` 생략 가능. 다르면 명시 |
| PK | `@GeneratedValue(strategy = GenerationType.IDENTITY)` |
| Enum | `@Enumerated(EnumType.STRING)` |
| 연관관계 | `@ManyToOne(fetch = FetchType.LAZY)` 기본. FK 컬럼을 Long으로만 들고 있지 말고 연관관계로 매핑하거나, Long으로 둘 경우에도 DB FK는 건다 |
| `@ElementCollection` | 쓰지 않는다. PK 없는 테이블이 생긴다. 별도 엔티티로 만든다 |
| 길이·NULL | `@Column(length, nullable)`은 Flyway SQL과 같게 적는다. validate는 길이까지 검사하지 않으므로 사람이 맞춘다 |
| 시각 | `*_at`(timestamptz)은 `OffsetDateTime` 또는 `Instant`, date는 `LocalDate`, time은 `LocalTime` |
