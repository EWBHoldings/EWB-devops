# Java Coding Standards

These standards apply to all Java projects within EWB. They are aligned with Google Java Style Guide and Spring Boot best practices.

---

## Language and Tooling

- Use **Java 21 LTS**
- Use **Maven** or **Gradle** as the build tool (Maven is preferred for new projects)
- Use **Checkstyle** for style enforcement
- Use **JUnit 5** and **Mockito** for testing
- Use **Spring Boot 3.x** for application services

---

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Package | lowercase, dot-separated | `com.EWB.payments.service` |
| Class | PascalCase | `OrderService` |
| Interface | PascalCase (no `I` prefix) | `OrderRepository` |
| Method | camelCase | `getOrderById` |
| Variable | camelCase | `orderCount` |
| Constant (`static final`) | SCREAMING_SNAKE_CASE | `MAX_RETRY_ATTEMPTS` |
| Enum type and values | PascalCase / SCREAMING_SNAKE_CASE | `OrderStatus.PENDING` |

---

## Package Structure (Spring Boot)

```
com.EWB.myservice/
├── controller/        # REST controllers
├── service/           # Business logic interfaces and implementations
├── repository/        # Spring Data JPA repositories
├── domain/            # JPA entities and domain objects
├── dto/               # Request/response data transfer objects
├── exception/         # Custom exception classes
├── config/            # Spring configuration classes
└── MyServiceApplication.java
```

---

## Classes

- One public class per file; file name matches class name
- Keep classes focused — a class over 300 lines is a signal to extract responsibility
- Use `record` for immutable DTOs (Java 16+)
- Mark utility classes `final` with a private constructor
- Prefer composition over inheritance

---

## Methods

- Methods should do one thing — extract multi-step logic into private helper methods
- Limit method length to approximately 20-30 lines
- Use early returns to reduce nesting
- Avoid long parameter lists — use parameter objects or builder pattern

---

## Spring Boot Specifics

- Annotate service classes with `@Service`, repositories with `@Repository`, controllers with `@RestController`
- Use constructor injection — never field injection (`@Autowired` on fields is discouraged):

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

- Use `@Transactional` at the service layer, not the repository or controller layer
- Validate request DTOs with Bean Validation annotations (`@NotNull`, `@NotBlank`, `@Size`)

---

## Exception Handling

- Define custom exception classes for domain errors
- Use `@ControllerAdvice` / `@RestControllerAdvice` for centralised HTTP error handling
- Do not expose stack traces in API responses

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(OrderNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(ex.getMessage()));
    }
}
```

---

## Null Handling

- Use `Optional<T>` for return values that may be absent — do not return `null` from service methods
- Validate method arguments using `Objects.requireNonNull()` or Bean Validation at API boundaries
- Annotate parameters and return types with `@NonNull` / `@Nullable` from a JSR-305 compatible library

---

## Logging

- Use **SLF4J** with **Logback** — do not use `System.out.println`
- Use parameterized log messages — never string concatenation:

```java
// Wrong
log.info("Processing order " + orderId);

// Correct
log.info("Processing order {}", orderId);
```

- Log at appropriate levels: `DEBUG` for diagnostic detail, `INFO` for notable events, `WARN` for recoverable issues, `ERROR` for failures requiring attention

---

## Testing

- Use **JUnit 5** (`@Test`, `@BeforeEach`, `@ParameterizedTest`)
- Use **Mockito** for mocking dependencies
- Use **AssertJ** for fluent assertions
- Use `@SpringBootTest` for integration tests; `@ExtendWith(MockitoExtension.class)` for unit tests
- Test method naming: `methodName_scenario_expectedResult`

```java
@Test
void getOrderById_orderExists_returnsOrder() {
    // Arrange
    var order = new Order(UUID.randomUUID(), OrderStatus.PENDING);
    when(orderRepository.findById(order.getId())).thenReturn(Optional.of(order));

    // Act
    var result = orderService.getOrderById(order.getId());

    // Assert
    assertThat(result).isPresent();
    assertThat(result.get().getStatus()).isEqualTo(OrderStatus.PENDING);
}
```
