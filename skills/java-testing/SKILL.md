---
name: java-testing
description: Java testing frameworks and best practices including JUnit 5, Mockito, Testcontainers, and integration testing strategies. Use when writing unit tests, integration tests, or setting up test infrastructure for Java applications.
triggers:
- junit
- mockito
- "@Test"
- "@Mock"
- "unit test"
- "integration test"
- "test case"
- "testing java"
- testcontainers
---

# Java Testing Guide

## Testing Frameworks

### Core Testing Stack

- **JUnit 5**: Modern testing framework for unit tests
- **Mockito**: Mocking framework for isolating dependencies
- **AssertJ**: Fluent assertion library for better test readability
- **Testcontainers**: Integration testing with real containers (databases, message brokers)
- **Hamcrest**: Matcher-based assertions (included with JUnit)

## Maven Dependencies

```xml
<dependencies>
    <!-- JUnit 5 (Jupiter) -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.9.2</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Mockito -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.1.1</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Mockito JUnit 5 integration -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>5.1.1</version>
        <scope>test</scope>
    </dependency>
    
    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.24.2</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Testcontainers -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers</artifactId>
        <version>1.17.6</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Testcontainers JUnit 5 extension -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>1.17.6</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Testcontainers PostgreSQL -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <version>1.17.6</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Spring Boot Test (if using Spring Boot) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## JUnit 5 Basics

### Basic Test Structure

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("UserService Tests")
class UserServiceTest {
    
    @Test
    @DisplayName("Should create user successfully")
    void shouldCreateUser() {
        // Given
        String email = "test@example.com";
        
        // When
        User user = userService.create(email);
        
        // Then
        assertNotNull(user);
        assertEquals(email, user.getEmail());
    }
    
    @Test
    @DisplayName("Should throw exception when email is null")
    void shouldThrowExceptionWhenEmailIsNull() {
        // Given
        String email = null;
        
        // When & Then
        assertThrows(IllegalArgumentException.class, () -> {
            userService.create(email);
        });
    }
}
```

### Common Assertions

```java
import static org.junit.jupiter.api.Assertions.*;

@Test
void testAssertions() {
    // Equality assertions
    assertEquals(4, 2 + 2);
    assertNotEquals(5, 2 + 2);
    
    // Boolean assertions
    assertTrue(true);
    assertFalse(false);
    
    // Null checks
    assertNotNull(object);
    assertNull(nullObject);
    
    // Same instance check
    Object obj = new Object();
    assertSame(obj, obj);
    assertNotSame(new Object(), new Object());
    
    // Array/Collection assertions
    assertArrayEquals(new int[]{1, 2}, new int[]{1, 2});
    
    // Exception assertion
    assertThrows(IllegalArgumentException.class, () -> {
        throw new IllegalArgumentException("Invalid argument");
    });
    
    // Timeout assertion
    assertTimeout(Duration.ofSeconds(2), () -> {
        // Code that should complete within 2 seconds
    });
}
```

### Lifecycle Hooks

```java
import org.junit.jupiter.api.*;

class UserServiceTest {
    
    private UserService userService;
    
    @BeforeAll
    static void setupAll() {
        // Runs once before all tests
        System.out.println("Setting up test class");
    }
    
    @BeforeEach
    void setup() {
        // Runs before each test
        userService = new UserService();
        System.out.println("Setting up test");
    }
    
    @Test
    void test1() {
        // Test implementation
    }
    
    @AfterEach
    void tearDown() {
        // Runs after each test
        System.out.println("Tearing down test");
    }
    
    @AfterAll
    static void tearDownAll() {
        // Runs once after all tests
        System.out.println("Tearing down test class");
    }
}
```

### Parameterized Tests

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.api.Assertions.*;

@ParameterizedTest
@ValueSource(strings = {"racecar", "madam", "level"})
void shouldCheckPalindrome(String word) {
    assertTrue(PalindromeChecker.isPalindrome(word));
}

@ParameterizedTest
@CsvSource({
    "test@example.com, true",
    "invalid-email, false",
    "another@test.co, true"
})
void shouldValidateEmail(String email, boolean expected) {
    assertEquals(expected, EmailValidator.isValid(email));
}

@ParameterizedTest
@MethodSource("provideTestData")
void shouldTestWithMethodSource(int input, int expected) {
    assertEquals(expected, Calculator.square(input));
}

private static Stream<Arguments> provideTestData() {
    return Stream.of(
        Arguments.of(2, 4),
        Arguments.of(3, 9),
        Arguments.of(5, 25)
    );
}
```

## Mockito

### Basic Mocking

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldFindUserById() {
        // Given
        Long userId = 1L;
        User user = new User(userId, "test@example.com");
        
        when(userRepository.findById(userId))
            .thenReturn(Optional.of(user));
        
        // When
        User result = userService.findById(userId);
        
        // Then
        assertNotNull(result);
        assertEquals("test@example.com", result.getEmail());
        verify(userRepository, times(1)).findById(userId);
    }
    
    @Test
    void shouldThrowExceptionWhenUserNotFound() {
        // Given
        Long userId = 1L;
        
        when(userRepository.findById(userId))
            .thenReturn(Optional.empty());
        
        // When & Then
        assertThrows(UserNotFoundException.class, () -> {
            userService.findById(userId);
        });
        
        verify(userRepository, times(1)).findById(userId);
    }
}
```

### Stubbing Methods

```java
@Test
void testStubbing() {
    // Return specific value
    when(userRepository.findById(1L))
        .thenReturn(Optional.of(user));
    
    // Return different values on consecutive calls
    when(userRepository.getCount())
        .thenReturn(10)
        .thenReturn(20)
        .thenReturn(30);
    
    // Throw exception
    when(userRepository.findById(999L))
        .thenThrow(new UserNotFoundException(999L));
    
    // Return based on argument matcher
    when(userRepository.findByEmail(anyString()))
        .thenReturn(Optional.of(user));
    
    // Return based on custom condition
    when(userRepository.findById(argThat(id -> id > 100)))
        .thenReturn(Optional.of(premiumUser));
    
    // Call real method
    when(userRepository.process(any()))
        .thenCallRealMethod();
}
```

### Verification

```java
@Test
void testVerification() {
    // Perform some actions
    userService.createUser("test@example.com");
    userService.createUser("another@example.com");
    
    // Verify exact number of calls
    verify(userRepository, times(2)).save(any(User.class));
    
    // Verify at least/at most
    verify(userRepository, atLeast(1)).save(any(User.class));
    verify(userRepository, atMostOnce()).findById(anyLong());
    
    // Verify never called
    verify(userRepository, never()).delete(anyLong());
    
    // Verify with specific arguments
    verify(userRepository).save(argThat(user -> 
        user.getEmail().equals("test@example.com")
    ));
    
    // Verify in order
    InOrder inOrder = inOrder(userRepository, emailService);
    inOrder.verify(userRepository).save(any(User.class));
    inOrder.verify(emailService).sendWelcomeEmail(anyString());
    
    // Verify no more interactions
    verifyNoMoreInteractions(userRepository);
}
```

### Mocking Static Methods (Mockito 3.4.0+)

```java
import org.mockito.MockedStatic;

@Test
void testStaticMocking() {
    try (MockedStatic<UtilityClass> mocked = 
         mockStatic(UtilityClass.class)) {
        
        mocked.when(() -> UtilityClass.staticMethod())
            .thenReturn("mocked value");
        
        assertEquals("mocked value", UtilityClass.staticMethod());
    }
}
```

## Integration Testing with Testcontainers

### PostgreSQL Integration Test

```java
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
class UserRepositoryIntegrationTest {
    
    @Container
    private static final PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:15-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldSaveAndRetrieveUser() {
        // Given
        User user = new User();
        user.setEmail("test@example.com");
        user.setFirstName("John");
        user.setLastName("Doe");
        
        // When
        User savedUser = userRepository.save(user);
        
        // Then
        assertNotNull(savedUser.getId());
        Optional<User> foundUser = userRepository.findById(savedUser.getId());
        assertTrue(foundUser.isPresent());
        assertEquals("test@example.com", foundUser.get().getEmail());
    }
}
```

### Generic Container Usage

```java
@Testcontainers
class GenericContainerTest {
    
    @Container
    private static final GenericContainer<?> redis = 
        new GenericContainer<>("redis:7-alpine")
            .withExposedPorts(6379);
    
    @Test
    void shouldConnectToRedis() {
        String redisHost = redis.getHost();
        Integer redisPort = redis.getFirstMappedPort();
        
        Jedis jedis = new Jedis(redisHost, redisPort);
        jedis.set("key", "value");
        
        assertEquals("value", jedis.get("key"));
    }
}
```

## AssertJ Assertions

### Fluent Assertions

```java
import static org.assertj.core.api.Assertions.*;

@Test
void testAssertJ() {
    User user = new User(1L, "test@example.com");
    
    // Chain assertions
    assertThat(user)
        .isNotNull()
        .hasFieldOrProperty("id")
        .hasFieldOrPropertyWithValue("email", "test@example.com");
    
    // String assertions
    assertThat(user.getEmail())
        .isNotEmpty()
        .contains("@")
        .endsWith("@example.com")
        .matches("^[A-Za-z0-9+_.-]+@(.+)$");
    
    // Collection assertions
    List<String> emails = Arrays.asList(
        "test1@example.com", 
        "test2@example.com"
    );
    
    assertThat(emails)
        .hasSize(2)
        .contains("test1@example.com")
        .doesNotContain("invalid")
        .allSatisfy(email -> 
            assertThat(email).contains("@")
        );
    
    // Number assertions
    assertThat(user.getId())
        .isPositive()
        .isGreaterThan(0L)
        .isLessThan(1000L);
    
    // Exception assertion
    assertThatThrownBy(() -> {
        userService.findById(999L);
    })
        .isInstanceOf(UserNotFoundException.class)
        .hasMessageContaining("999");
}
```

## Testing Best Practices

### 1. Test Naming Convention

Use descriptive test names that explain what is being tested:

```java
// Good
void shouldReturnUserWhenValidIdIsProvided() {}
void shouldThrowExceptionWhenUserNotFound() {}
void shouldNotAllowDuplicateEmails() {}

// Avoid
void testUser() {}
void test1() {}
```

### 2. AAA Pattern (Arrange-Act-Assert)

```java
@Test
void shouldCalculateTotalPrice() {
    // Arrange (Given)
    ShoppingCart cart = new ShoppingCart();
    cart.addItem(new Item("Product", 100.0));
    
    // Act (When)
    double total = cart.calculateTotal();
    
    // Assert (Then)
    assertEquals(100.0, total);
}
```

### 3. Test Independence

Each test should be independent and not rely on other tests:

```java
@Test
void test1() {
    // Don't rely on state from previous tests
    User user = new User("test@example.com");
    // ...
}

@Test
void test2() {
    // Create fresh state, don't rely on test1
    User user = new User("another@example.com");
    // ...
}
```

### 4. Use Meaningful Assertions

```java
// Bad
assertNotNull(result);

// Good
assertNotNull(result.getId());
assertEquals("test@example.com", result.getEmail());
assertTrue(result.isActive());
```

### 5. Avoid Testing Third-Party Code

Don't test libraries or frameworks, test your business logic:

```java
// Bad - testing the List implementation
@Test
void testListAdd() {
    List<String> list = new ArrayList<>();
    list.add("item");
    assertEquals(1, list.size());
}

// Good - testing your business logic
@Test
void shouldAddItemToCart() {
    Cart cart = new Cart();
    cart.addItem(new Item("Product", 100.0));
    assertEquals(1, cart.getItemCount());
    assertEquals(100.0, cart.getTotal());
}
```

## Spring Boot Testing

### @WebMvcTest (Controller Layer)

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldGetUserById() throws Exception {
        User user = new User(1L, "test@example.com");
        
        when(userService.findById(1L)).thenReturn(user);
        
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.email").value("test@example.com"));
    }
}
```

### @DataJpaTest (Repository Layer)

```java
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldFindUserByEmail() {
        User user = new User();
        user.setEmail("test@example.com");
        entityManager.persist(user);
        
        Optional<User> found = userRepository.findByEmail("test@example.com");
        
        assertTrue(found.isPresent());
        assertEquals("test@example.com", found.get().getEmail());
    }
}
```

### @SpringBootTest (Integration Tests)

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void shouldCreateUser() throws Exception {
        UserDto userDto = new UserDto("test@example.com");
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(userDto)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.email").value("test@example.com"));
    }
}
```

## Performance Testing

```java
@Test
void shouldHandleLargeNumberOfRequests() {
    // Warm-up
    for (int i = 0; i < 1000; i++) {
        userService.findById(1L);
    }
    
    // Measure
    long startTime = System.nanoTime();
    for (int i = 0; i < 10000; i++) {
        userService.findById(1L);
    }
    long duration = System.nanoTime() - startTime;
    
    // Assert performance (10K requests in < 1 second)
    assertTrue(duration < 1_000_000_000);
}
```

## Continuous Testing Configuration

Add to `pom.xml`:

```xml
<build>
    <plugins>
        <!-- Surefire Plugin for Unit Tests -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0</version>
        </plugin>
        
        <!-- Failsafe Plugin for Integration Tests -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>3.0.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>integration-test</goal>
                        <goal>verify</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
