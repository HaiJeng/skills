---
name: spring-boot
description: Spring Boot framework for building production-ready applications with Spring ecosystem. Use when developing Spring Boot applications, configuring beans, working with Spring annotations, or setting up REST APIs.
triggers:
- spring-boot
- spring boot
- "@SpringBootApplication"
- "@RestController"
- "@Autowired"
- "@Service"
- "@Repository"
- "@Entity"
- application.properties
- application.yml
---

# Spring Boot Development Guide

## Spring Boot Project Structure

Standard Spring Boot application layout:

```
my-spring-boot-app/
├── pom.xml / build.gradle                    # Build configuration
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/myapp/
│   │   │       ├── MyApplication.java        # Main application class
│   │   │       ├── controller/               # REST controllers
│   │   │       ├── service/                  # Business logic
│   │   │       ├── repository/               # Data access layer
│   │   │       ├── model/                    # Domain models/entities
│   │   │       ├── dto/                      # Data transfer objects
│   │   │       ├── config/                   # Configuration classes
│   │   │       └── exception/                # Custom exceptions
│   │   └── resources/
│   │       ├── application.yml               # Application configuration
│   │       ├── application.properties         # Alternative configuration
│   │       ├── application-dev.yml            # Dev environment config
│   │       ├── application-prod.yml           # Prod environment config
│   │       └── static/                       # Static web resources
│   └── test/
│       └── java/
│           └── com/example/myapp/
│               ├── controller/               # Controller tests
│               ├── service/                  # Service tests
│               └── repository/               # Repository tests
└── src/main/resources/
    └── application.yml
```

## Essential Spring Boot Annotations

### Core Annotations

```java
// Mark main application class
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// REST Controller
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    @PostMapping
    public User createUser(@RequestBody @Valid UserDto userDto) {
        return userService.create(userDto);
    }
    
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody UserDto userDto) {
        return userService.update(id, userDto);
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}

// Service layer
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}

// Repository layer (Spring Data JPA)
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByLastName(String lastName);
}

// Entity
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String firstName;
    
    @Column(nullable = false)
    private String lastName;
    
    // Getters and setters
}
```

## Configuration

### application.yml

```yaml
# Server configuration
server:
  port: 8080
  servlet:
    context-path: /api

# Application name
spring:
  application:
    name: my-spring-boot-app
  
  # Database configuration (H2 for dev)
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  
  # JPA configuration
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
        format_sql: true
  
  # H2 console (for development)
  h2:
    console:
      enabled: true
      path: /h2-console

# Logging configuration
logging:
  level:
    com.example.myapp: DEBUG
    org.springframework.web: INFO
    org.hibernate: INFO
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"

# Custom application properties
app:
  name: My Spring Boot Application
  version: 1.0.0
  security:
    jwt:
      secret: mySecretKey
      expiration: 86400000
```

### Profile-Specific Configuration

**application-dev.yml:**
```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
  jpa:
    show-sql: true

logging:
  level:
    com.example.myapp: DEBUG
```

**application-prod.yml:**
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: produser
    password: ${DB_PASSWORD}
  jpa:
    show-sql: false

logging:
  level:
    com.example.myapp: INFO
```

### Using Properties in Code

```java
@Component
public class AppProperties {
    
    @Value("${app.name}")
    private String appName;
    
    @Value("${app.version}")
    private String appVersion;
    
    @Value("${app.security.jwt.secret}")
    private String jwtSecret;
    
    @Value("${app.security.jwt.expiration}")
    private Long jwtExpiration;
    
    // Getters
}

// Or using @ConfigurationProperties (recommended)
@Configuration
@ConfigurationProperties(prefix = "app.security.jwt")
public class JwtProperties {
    private String secret;
    private Long expiration;
    
    // Getters and setters
}
```

## Dependency Injection

### Field Injection (NOT recommended)
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

### Constructor Injection (RECOMMENDED)
```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    // With Spring 4.3+, @Autowired is optional if只有一个构造函数
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// Or using Lombok (RECOMMENDED)
@Service
@RequiredArgsConstructor
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
}
```

### Setter Injection
```java
@Service
public class UserService {
    
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

## REST API Development

### Request Handling

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // GET all users with pagination
    @GetMapping
    public Page<UserDto> getAllUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy
    ) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return userService.findAll(pageable);
    }
    
    // GET user by ID
    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getUserById(@PathVariable Long id) {
        UserDto user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
    
    // POST create new user
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserDto createUser(@RequestBody @Valid UserDto userDto) {
        return userService.create(userDto);
    }
    
    // PUT update user
    @PutMapping("/{id}")
    public UserDto updateUser(
        @PathVariable Long id,
        @RequestBody @Valid UserDto userDto
    ) {
        return userService.update(id, userDto);
    }
    
    // DELETE user
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
    
    // Search users
    @GetMapping("/search")
    public List<UserDto> searchUsers(@RequestParam String keyword) {
        return userService.search(keyword);
    }
}
```

### Exception Handling

```java
// Custom exception
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}

// Global exception handler
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            errors
        );
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

## Data Access with Spring Data JPA

### Entity Definition

```java
@Entity
@Table(name = "users")
@EntityListeners(AuditingEntityListener.class)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String password;
    
    @Column(nullable = false)
    private String firstName;
    
    @Column(nullable = false)
    private String lastName;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserRole role;
    
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    // Relationships
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Order> orders;
    
    // Getters and setters
}
```

### Repository Interface

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
    
    // Query methods
    Optional<User> findByEmail(String email);
    List<User> findByLastName(String lastName);
    Page<User> findByRole(UserRole role, Pageable pageable);
    
    // Custom query with @Query
    @Query("SELECT u FROM User u WHERE u.email LIKE %:email%")
    List<User> searchByEmail(@Param("email") String email);
    
    // Native query
    @Query(value = "SELECT * FROM users WHERE created_at > :date", nativeQuery = true)
    List<User> findUsersCreatedAfter(@Param("date") LocalDateTime date);
    
    // Modifying query
    @Modifying
    @Query("UPDATE User u SET u.password = :password WHERE u.id = :id")
    int updateUserPassword(@Param("id") Long id, @Param("password") String password);
}
```

### Service Layer

```java
@Service
@Transactional
@RequiredArgsConstructor
public class UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    @Transactional(readOnly = true)
    public Page<UserDto> findAll(Pageable pageable) {
        return userRepository.findAll(pageable)
            .map(this::convertToDto);
    }
    
    @Transactional(readOnly = true)
    public UserDto findById(Long id) {
        return userRepository.findById(id)
            .map(this::convertToDto)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    public UserDto create(UserDto userDto) {
        User user = convertToEntity(userDto);
        user.setPassword(passwordEncoder.encode(userDto.getPassword()));
        User savedUser = userRepository.save(user);
        return convertToDto(savedUser);
    }
    
    public UserDto update(Long id, UserDto userDto) {
        User existingUser = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
        
        // Update fields
        existingUser.setFirstName(userDto.getFirstName());
        existingUser.setLastName(userDto.getLastName());
        
        return convertToDto(userRepository.save(existingUser));
    }
    
    public void delete(Long id) {
        if (!userRepository.existsById(id)) {
            throw new UserNotFoundException(id);
        }
        userRepository.deleteById(id);
    }
    
    private UserDto convertToDto(User user) {
        // Conversion logic
        return new UserDto(user);
    }
    
    private User convertToEntity(UserDto dto) {
        // Conversion logic
        return new User(dto);
    }
}
```

## Validation

### DTO with Validation

```java
public class UserDto {
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
    
    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 50)
    private String firstName;
    
    @NotBlank(message = "Last name is required")
    @Size(min = 2, max = 50)
    private String lastName;
    
    // Getters and setters
}
```

### Enable Validation in Controller

```java
@PostMapping
@Validated
public UserDto createUser(@RequestBody @Valid UserDto userDto) {
    return userService.create(userDto);
}
```

## Testing

### Unit Test

```java
@SpringBootTest
class UserServiceTest {
    
    @Autowired
    private UserService userService;
    
    @MockBean
    private UserRepository userRepository;
    
    @Test
    void shouldFindUserById() {
        // Given
        User user = new User();
        user.setId(1L);
        user.setEmail("test@example.com");
        
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));
        
        // When
        UserDto result = userService.findById(1L);
        
        // Then
        assertNotNull(result);
        assertEquals("test@example.com", result.getEmail());
        verify(userRepository, times(1)).findById(1L);
    }
}
```

### Integration Test

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void shouldCreateUser() throws Exception {
        UserDto userDto = new UserDto();
        userDto.setEmail("test@example.com");
        userDto.setPassword("password123");
        userDto.setFirstName("John");
        userDto.setLastName("Doe");
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(userDto)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.email").value("test@example.com"));
    }
}
```

## Best Practices

1. **Use Constructor Injection**: Prefer constructor injection over field injection
2. **Layered Architecture**: Maintain clear separation between controller, service, and repository
3. **DTO Pattern**: Use DTOs for API requests/responses instead of exposing entities
4. **Exception Handling**: Use @ControllerAdvice for global exception handling
5. **Validation**: Always validate input using @Valid and javax.validation annotations
6. **Transaction Management**: Use @Transactional appropriately (read-only for queries)
7. **Configuration Management**: Use Spring profiles for environment-specific configuration
8. **Testing**: Write both unit tests (with mocks) and integration tests
9. **Logging**: Use SLF4J with proper logging levels
10. **Security**: Always hash passwords, never store plain text passwords
11. **OpenAPI/Swagger**: Document your API with SpringDoc OpenAPI
12. **Lombok**: Reduce boilerplate code with Lombok annotations (@Data, @RequiredArgsConstructor)

## Common Dependencies for pom.xml

```xml
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!-- H2 Database (for development) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- PostgreSQL (for production) -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <!-- SpringDoc OpenAPI (Swagger UI) -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.0.2</version>
    </dependency>
    
    <!-- Spring Boot Starter Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```
