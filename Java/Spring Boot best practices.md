
# Spring Boot Best Practices

## 📋 Contents

1. [Project Structure](#1-project-structure)
2. [Dependency Management](#2-dependency-management)
3. [Configuration Management](#3-configuration-management)
4. [Bean Management and Dependency Injection](#4-bean-management-and-dependency-injection)
5. [REST API Design](#5-rest-api-design)
6. [Exception Handling](#6-exception-handling)
7. [Validation](#7-validation)
8. [Database and JPA](#8-database-and-jpa)
9. [Security](#9-security)
10. [Logging](#10-logging)
11. [Testing](#11-testing)
12. [Performance Optimization](#12-performance-optimization)
13. [Monitoring and Observability](#13-monitoring-and-observability)
14. [Docker and Deployment](#14-docker-and-deployment)
15. [Common Pitfalls](#15-common-pitfalls)
16. [Interview Q&A](#17-interview-qa)

***

## 1. Project Structure

### Layer-based structure (Traditional)

```
src/main/java/com/example/app/
├── controller/       # REST endpoints
├── service/          # Business logic
├── repository/       # Data access
├── model/            # Entities
├── dto/              # Data Transfer Objects
├── config/           # Configuration classes
├── exception/        # Custom exceptions
├── security/         # Security config
└── util/             # Utilities
```

### Feature-based structure (Recommended for larger apps)

```
src/main/java/com/example/app/
├── user/
│   ├── UserController.java
│   ├── UserService.java
│   ├── UserRepository.java
│   ├── User.java
│   └── UserDTO.java
├── order/
│   ├── OrderController.java
│   ├── OrderService.java
│   └── ...
└── config/
    └── SecurityConfig.java
```

**Advantages:**

- High cohesion — related code stays together
- Easier to navigate and maintain
- Better for microservices evolution

***

### Keep configuration separate

```
src/main/resources/
├── application.yml            # Default config
├── application-dev.yml        # Dev environment
├── application-prod.yml       # Production
├── db/
│   └── migration/             # Flyway/Liquibase scripts
└── static/                    # Static resources
```

***

## 2. Dependency Management

### Use Spring Boot Starter dependencies

✅ **Correct:**

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

❌ **Avoid:**

```xml
<!-- Don't manually manage Spring dependencies -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>6.0.9</version>
</dependency>
```

***

### Use BOM for version management

```xml

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.2.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

***

### Exclude unnecessary dependencies

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

***

### Use `test` scope correctly

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

***

## 3. Configuration Management

### Use YAML over Properties

✅ **Preferred:**

```yaml
spring:
  application:
    name: my-app
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
```

❌ **Less readable:**

```properties
spring.application.name=my-app
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=${DB_USER}
```

***

### Use profiles for environments

```yaml
# application.yml
spring:
  profiles:
    active: ${SPRING_PROFILE:dev}

---
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
logging:
  level:
    root: DEBUG

---
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://prod-db:5432/mydb
logging:
  level:
    root: WARN
```

**Activate profile:**

```bash
java -jar app.jar --spring.profiles.active=prod
```

***

### Use `@ConfigurationProperties` for type-safe config

✅ **Correct:**

```java

@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    @NotNull
    private String name;

    @Min(1024)
    private int maxUploadSize;

    private Security security = new Security();

    public static class Security {
        private boolean enabled;
        private String apiKey;
    }
}
```

```yaml
app:
  name: MyApp
  max-upload-size: 10485760
  security:
    enabled: true
    api-key: ${API_KEY}
```

Enable in main class:

```java

@SpringBootApplication
@EnableConfigurationProperties(AppProperties.class)
public class Application {
}
```

***

### Externalize secrets

❌ **Never hardcode:**

```yaml
spring:
  datasource:
    password: admin123  # Bad!
```

✅ **Use environment variables:**

```yaml
spring:
  datasource:
    password: ${DB_PASSWORD}
```

Or use **Spring Cloud Config**, **Vault**, **AWS Secrets Manager**.

***

## 4. Bean Management and Dependency Injection

### Prefer constructor injection

✅ **Best practice:**

```java

@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    public UserService(UserRepository userRepository,
                       EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```

**Benefits:**

- Immutability (final fields)
- Easier testing
- Clear dependencies
- No `@Autowired` needed (since Spring 4.3)

***

❌ **Avoid field injection:**

```java

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository; // Hard to test
}
```

***

### Use `@RequiredArgsConstructor` (Lombok)

```java

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
}
```

***

### Avoid circular dependencies

❌ **Problem:**

```java

@Service
public class ServiceA {
    private final ServiceB serviceB;
}

@Service
public class ServiceB {
    private final ServiceA serviceA; // Circular!
}
```

✅ **Solutions:**

1. Refactor to extract common logic
2. Use `@Lazy` (temporary fix)
3. Use events or interfaces

***

### Use `@Component` stereotypes correctly

| Annotation        | Purpose               |
|-------------------|-----------------------|
| `@Component`      | Generic Spring bean   |
| `@Service`        | Business logic layer  |
| `@Repository`     | Data access layer     |
| `@Controller`     | MVC controller        |
| `@RestController` | REST API endpoints    |
| `@Configuration`  | Configuration classes |

***

## 5. REST API Design

### Use proper HTTP methods

| Method   | Use Case                | Idempotent |
|----------|-------------------------|------------|
| `GET`    | Retrieve resources      | ✅ Yes      |
| `POST`   | Create new resource     | ❌ No       |
| `PUT`    | Replace entire resource | ✅ Yes      |
| `PATCH`  | Partial update          | ❌ No       |
| `DELETE` | Remove resource         | ✅ Yes      |

---

### Use `ResponseEntity` for full control

✅ **Correct:**

```java

@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserDTO created = userService.create(request);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }
}
```

***

### Use proper HTTP status codes

| Code | Meaning               | Use Case                          |
|------|-----------------------|-----------------------------------|
| 200  | OK                    | Successful GET/PUT/PATCH          |
| 201  | Created               | Successful POST                   |
| 204  | No Content            | Successful DELETE                 |
| 400  | Bad Request           | Validation errors                 |
| 401  | Unauthorized          | Missing authentication            |
| 403  | Forbidden             | Insufficient permissions          |
| 404  | Not Found             | Resource doesn't exist            |
| 409  | Conflict              | Duplicate or constraint violation |
| 500  | Internal Server Error | Unexpected server error           |

***

### Version your APIs

```java

@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {
}

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
}
```

Or use headers:

```java
@GetMapping(value = "/users", headers = "X-API-VERSION=1")
```

***

### Use DTOs, not entities

❌ **Bad:**

```java

@PostMapping
public User createUser(@RequestBody User user) {
    return userRepository.save(user); // Exposes entity
}
```

✅ **Good:**

```java

@PostMapping
public UserDTO createUser(@Valid @RequestBody CreateUserRequest request) {
    User user = userMapper.toEntity(request);
    User saved = userRepository.save(user);
    return userMapper.toDTO(saved);
}
```

***

## 6. Exception Handling

### Use `@ControllerAdvice` for global exception handling

```java

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(
        MethodArgumentNotValidException ex) {
        Map<String, String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage
            ));

        ValidationErrorResponse response = new ValidationErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            errors,
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unexpected error", ex);
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

***

### Define custom exceptions

```java

@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

@ResponseStatus(HttpStatus.CONFLICT)
public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}
```

***

### Standardize error responses

```java

@Getter
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;
}

@Getter
@AllArgsConstructor
public class ValidationErrorResponse {
    private int status;
    private String message;
    private Map<String, String> errors;
    private LocalDateTime timestamp;
}
```

***

## 7. Validation

### Use Bean Validation annotations

```java
public class CreateUserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;

    @NotNull
    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must not exceed 120")
    private Integer age;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone number")
    private String phone;
}
```

***

### Validate in controllers

```java

@PostMapping
public ResponseEntity<UserDTO> createUser(
    @Valid @RequestBody CreateUserRequest request) {
    // Validation happens automatically
    UserDTO created = userService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

***

### Custom validators

```java

@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}

public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {

    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null) return true;
        return !userRepository.existsByEmail(email);
    }
}
```

Usage:

```java

@UniqueEmail
@Email
private String email;
```

***

## 8. Database and JPA

### Use connection pooling (HikariCP)

Spring Boot uses HikariCP by default:

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
```

***

### Use database migrations

**Flyway:**

```xml

<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
```

```sql
-- V1__create_users_table.sql
CREATE TABLE users
(
    id         BIGSERIAL PRIMARY KEY,
    name       VARCHAR(100)        NOT NULL,
    email      VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

***

### Avoid N+1 queries

See [Hibernate Best Practices](#) section for details.

✅ **Use:**

- `@EntityGraph`
- `JOIN FETCH`
- `@BatchSize`
- DTO projections

***

### Use projections for read queries

```java
public interface UserSummary {
    String getName();

    String getEmail();
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findAllProjectedBy();
}
```

***

### Use pagination

```java

@GetMapping
public Page<UserDTO> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "id,asc") String[] sort) {

    Pageable pageable = PageRequest.of(page, size, Sort.by(
        sort[1].equals("desc") ? Sort.Direction.DESC : Sort.Direction.ASC,
        sort[0]
    ));

    return userService.findAll(pageable);
}
```

***

## 9. Security

### Use Spring Security

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

***

### Configure security properly

```java

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

***

### Use method-level security

```java

@Service
public class UserService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    @PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
    public UserDTO getUser(Long userId) {
        return userRepository.findById(userId)
            .map(userMapper::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    }
}
```

***

### Never log sensitive data

❌ **Bad:**

```java
log.info("User login: {}",user); // May contain password!
```

✅ **Good:**

```java
log.info("User login: userId={}, email={}",user.getId(), user.getEmail());
```

***

### Use HTTPS in production

```yaml
server:
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: ${KEYSTORE_PASSWORD}
    key-store-type: PKCS12
```

***

## 10. Logging

### Use SLF4J with Logback

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class UserService {

    public UserDTO create(CreateUserRequest request) {
        log.debug("Creating user: {}", request.getEmail());

        try {
            User user = userMapper.toEntity(request);
            User saved = userRepository.save(user);
            log.info("User created: id={}, email={}", saved.getId(), saved.getEmail());
            return userMapper.toDTO(saved);
        } catch (Exception e) {
            log.error("Failed to create user: {}", request.getEmail(), e);
            throw e;
        }
    }
}
```

***

### Configure log levels per environment

```yaml
# application-dev.yml
logging:
  level:
    root: INFO
    com.example.app: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE

# application-prod.yml
logging:
  level:
    root: WARN
    com.example.app: INFO
  file:
    name: /var/log/app.log
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

***

### Use structured logging

```java
log.info("Order processed: orderId={}, userId={}, amount={}, status={}",
         order.getId(), 
         order.getUserId(), 
         order.getAmount(), 
         order.getStatus());
```

***

## 11. Testing

### Use test slices

```java

@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUser() throws Exception {
        UserDTO user = new UserDTO(1L, "John", "john@example.com");
        when(userService.findById(1L)).thenReturn(Optional.of(user));

        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"));
    }
}
```

***

### Use `@DataJpaTest` for repository tests

```java

@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldFindByEmail() {
        User user = new User("John", "john@example.com");
        userRepository.save(user);

        Optional<User> found = userRepository.findByEmail("john@example.com");

        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("John");
    }
}
```

***

### Use Testcontainers for integration tests

```java

@SpringBootTest
@Testcontainers
class UserServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserService userService;

    @Test
    void shouldCreateUser() {
        CreateUserRequest request = new CreateUserRequest("John", "john@test.com", 25);
        UserDTO created = userService.create(request);
        assertThat(created.getId()).isNotNull();
    }
}
```

***

### Aim for high test coverage

- **Unit tests**: 80%+ coverage
- **Integration tests**: Critical paths
- **E2E tests**: Happy path scenarios

***

## 12. Performance Optimization

### Enable caching

```java

@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "products");
    }
}
```

```java

@Service
public class UserService {

    @Cacheable("users")
    public UserDTO findById(Long id) {
        return userRepository.findById(id)
            .map(userMapper::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    }

    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

***

### Use async processing

```java

@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("Async-");
        executor.initialize();
        return executor;
    }
}
```

```java

@Service
public class EmailService {

    @Async
    public CompletableFuture<Void> sendEmail(String to, String subject, String body) {
        // Send email asynchronously
        log.info("Sending email to: {}", to);
        return CompletableFuture.completedFuture(null);
    }
}
```

***

### Enable compression

```yaml
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/xml,text/plain
    min-response-size: 1024
```

***

### Optimize JVM settings

```bash
java -Xms512m -Xmx2048m \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:+HeapDumpOnOutOfMemoryError \
     -jar app.jar
```

***

## 13. Monitoring and Observability

### Use Spring Boot Actuator

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  metrics:
    tags:
      application: ${spring.application.name}
```

***

### Add custom health indicators

```java

@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    @Autowired
    private DataSource dataSource;

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            return Health.up()
                .withDetail("database", "Reachable")
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```


***

## 14. Docker and Deployment

### Create optimized Dockerfile

```dockerfile
# Multi-stage build
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline
COPY src src
RUN ./mvnw package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

***

### Use Docker Compose for local development

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      DB_HOST: postgres
      DB_USER: user
      DB_PASSWORD: password
    depends_on:
      - postgres

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

***

### Configure graceful shutdown

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

***

## 15. Common Pitfalls

### Blocking in async methods

❌ **Wrong:**

```java

@Async
public void process() {
    Thread.sleep(1000); // Blocks thread pool
}
```

✅ **Correct:**

```java

@Async
public CompletableFuture<Void> process() {
    return CompletableFuture.runAsync(() -> {
        // Non-blocking operation
    });
}
```

***

### Not closing resources

❌ **Wrong:**

```java
InputStream is = new FileInputStream("file.txt");
// May leak if exception occurs
```

✅ **Correct:**

```java
try(InputStream is = new FileInputStream("file.txt")){
    // Automatically closed
    }
```

***



### Ignoring transaction boundaries

❌ **Wrong:**

```java
public void createOrder(Order order) {
    orderRepository.save(order); // No transaction
    inventoryService.reduceStock(order); // Separate transaction
    // If second call fails, first change is committed!
}
```

✅ **Correct:**

```java

@Transactional
public void createOrder(Order order) {
    orderRepository.save(order);
    inventoryService.reduceStock(order);
    // Both succeed or both rollback
}
```

***

### Using `@Autowired` on fields

See [Bean Management](#4-bean-management-and-dependency-injection) — prefer constructor injection.

***

## 16. Interview Q&A

| Question                                                                    | Short Answer                                                                                              |
|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| What is the difference between `@Component`, `@Service`, and `@Repository`? | All create beans; `@Service` for business logic, `@Repository` for data access with exception translation |
| Why prefer constructor injection over field injection?                      | Immutability, easier testing, clear dependencies, no need for `@Autowired`                                |
| What is `@RestControllerAdvice`?                                            | Global exception handler for REST APIs                                                                    |
| How to handle validation errors in Spring Boot?                             | Use `@Valid` + `@ControllerAdvice` with `MethodArgumentNotValidException` handler                         |
| What is the difference between `@RequestBody` and `@RequestParam`?          | `@RequestBody` for JSON payloads, `@RequestParam` for query parameters                                    |
| How to enable caching in Spring Boot?                                       | `@EnableCaching` + `@Cacheable`/`@CacheEvict`                                                             |
| What is Spring Boot Actuator?                                               | Production-ready features for monitoring and management                                                   |
| How to configure multiple environments?                                     | Use Spring profiles: `application-dev.yml`, `application-prod.yml`                                        |
| What is `@Transactional`?                                                   | Marks method/class to run within a database transaction                                                   |
| Why use DTOs instead of entities?                                           | Decouples API from database schema, prevents over-fetching, security                                      |
| How to version REST APIs?                                                   | URL versioning (`/api/v1/users`) or header versioning                                                     |
| What is `@ConfigurationProperties`?                                         | Type-safe configuration binding from `application.yml`                                                    |
| How to handle circular dependencies?                                        | Refactor, use `@Lazy`, or event-driven design                                                             |
| What is the difference between `@Component` and `@Bean`?                    | `@Component` for class-level auto-scanning, `@Bean` for method-level manual registration                  |
| How to enable async processing?                                             | `@EnableAsync` + `@Async` on methods                                                                      |

***

### Sources

- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Baeldung Spring Boot Guides](https://www.baeldung.com/spring-boot)
- [Spring Boot Best Practices (GitHub)](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide)
- [Production-Ready Spring Boot (Book)](https://www.manning.com/books/spring-boot-in-practice)
- [Perfect-Roadmap-To-Learn-Java-SpringBoot-In-2025-26](https://github.com/bhuvnesharya/Perfect-Roadmap-To-Learn-Java-SpringBoot-In-2024)
- [Reddit](https://www.reddit.com/r/SpringBoot/comments/1buv6hn/what_are_the_best_practices_in_spring_boot/)
- [Geeks for geeks](https://www.geeksforgeeks.org/springboot/best-way-to-master-spring-boot-a-complete-roadmap/)
- [Optimizing Spring Boot Applications: Tips for Peak Performance](https://www.javacodegeeks.com/2024/12/optimizing-spring-boot-applications-tips-for-peak-performance.html)
- [Spring Boot folder structure best practices](https://symflower.com/en/company/blog/2024/spring-boot-folder-structure/)

