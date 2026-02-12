---
name: java-style
description: Java code style conventions, best practices, and coding standards including Checkstyle, SpotBugs, and Google Java Style Guide. Use when reviewing Java code, setting up linting tools, or ensuring consistent code style.
triggers:
- "code style"
- "coding standards"
- "java convention"
- checkstyle
- spotbugs
- "code formatting"
- "code review"
---

# Java Code Style and Best Practices

## Code Style Standards

### Naming Conventions

```java
// Classes: PascalCase
public class UserService {}
public class HttpClientConnection {}

// Interfaces: PascalCase, often start with 'I' or descriptive
public interface UserRepository {}
public interface Serializable {}

// Methods: camelCase, should be verbs
public void getUserById() {}
public boolean isValidEmail() {}
public void calculateTotal() {}

// Variables: camelCase, meaningful names
private String userEmail;
private int maximumRetryCount;
private final DatabaseConnection connection;

// Constants: UPPER_SNAKE_CASE
public static final String API_BASE_URL = "https://api.example.com";
public static final int MAX_CONNECTIONS = 100;
private static final Logger LOGGER = LoggerFactory.getLogger(MyClass.class);

// Packages: all lowercase, reverse domain notation
package com.example.service;
package org.apache.commons.lang3;

// Enums: PascalCase for enum, UPPER_SNAKE_CASE for values
public enum UserRole {
    ADMIN,
    USER,
    GUEST
}
```

### Class Organization

```java
// Standard order (Google Java Style):
// 1. Package statement
package com.example.service;

// 2. Imports (grouped and sorted)
import java.util.List;
import java.util.ArrayList;

import javax.annotation.Nullable;

import com.example.model.User;
import org.springframework.stereotype.Service;

// 3. Class documentation
/**
 * Service class for managing user-related operations.
 * Provides methods for creating, updating, and retrieving user data.
 *
 * @author John Doe
 * @since 1.0
 */
// 4. Class declaration
public class UserService {
    
    // 5. Static fields (constants first)
    private static final Logger LOGGER = LoggerFactory.getLogger(UserService.class);
    private static final int DEFAULT_PAGE_SIZE = 20;
    
    // 6. Instance fields
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // 7. Constructors
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    // 8. Static methods
    public static boolean isValidEmail(String email) {
        return email != null && email.contains("@");
    }
    
    // 9. Public instance methods
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    // 10. Protected methods
    protected void validateUser(User user) {
        // Validation logic
    }
    
    // 11. Package-private methods
    
    // 12. Private methods
    private void sendWelcomeEmail(User user) {
        emailService.sendEmail(user.getEmail(), "Welcome!");
    }
    
    // 13. Inner classes
    private static class UserValidator {
        // ...
    }
}
```

## Code Formatting

### Indentation and Braces

```java
// Use 4 spaces for indentation (NOT tabs)

// K&R style (opening brace on same line)
public void example() {
    if (condition) {
        doSomething();
    } else {
        doSomethingElse();
    }
    
    for (int i = 0; i < 10; i++) {
        process(i);
    }
}

// Always use braces, even for single statements
if (condition) {
    return true;  // Good
}

// Avoid this (even though it works):
if (condition)
    return true;  // Bad - hard to read and error-prone
```

### Line Length and Wrapping

```java
// Maximum line length: 100 characters (Google style) or 120 characters

// Wrap long method calls
String result = someObject.someMethod(
    firstArgument,
    secondArgument,
    thirdArgument
);

// Wrap chained methods
String result = someObject
    .methodOne()
    .methodTwo()
    .methodThree();

// Wrap long conditions
if (firstCondition && secondCondition && thirdCondition 
    && fourthCondition) {
    // ...
}

// Better: extract to boolean method
if (isValidUser() && hasPermission() && isNotExpired()) {
    // ...
}
```

### Spacing

```java
// Spaces around operators
int sum = a + b;
boolean isEqual = (x == y);

// Space after comma
method(arg1, arg2, arg3);
int[] array = {1, 2, 3};

// Space after keywords
if (condition) {
    // ...
}

while (condition) {
    // ...
}

// No space before parentheses
method();  // Good
method ();  // Bad

// Spacing in declarations
private int count;  // Good
private int   count;  // Bad
```

## Best Practices

### 1. Use Final for Immutability

```java
// Good: final for fields that won't change
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// Good: final for local variables that won't change
public void processUser() {
    final List<User> users = userRepository.findAll();
    final String adminEmail = "admin@example.com";
    
    for (final User user : users) {
        // user is effectively final in enhanced for loop
        System.out.println(user.getName());
    }
}
```

### 2. Avoid Magic Numbers and Strings

```java
// Bad: magic numbers
if (user.getAge() > 18) {
    // ...
}

// Good: use constants
private static final int LEGAL_ADULT_AGE = 18;

if (user.getAge() > LEGAL_ADULT_AGE) {
    // ...
}

// Bad: magic strings
if (user.getStatus().equals("ACTIVE")) {
    // ...
}

// Good: use enums
public enum UserStatus {
    ACTIVE,
    INACTIVE,
    PENDING,
    SUSPENDED
}

if (user.getStatus() == UserStatus.ACTIVE) {
    // ...
}
```

### 3. Null Safety

```java
// Good: return Optional instead of null
public Optional<User> findById(Long id) {
    User user = repository.findById(id);
    return Optional.ofNullable(user);
}

// Good: use Optional methods
Optional<User> user = findById(1L);
user.ifPresent(u -> System.out.println(u.getName()));

String email = user.map(User::getEmail).orElse("default@example.com");

// Good: validate parameters early
public void processUser(User user) {
    if (user == null) {
        throw new IllegalArgumentException("User cannot be null");
    }
    
    if (user.getEmail() == null) {
        throw new IllegalArgumentException("Email cannot be null");
    }
    
    // OR use Objects.requireNonNull
    Objects.requireNonNull(user, "User cannot be null");
    Objects.requireNonNull(user.getEmail(), "Email cannot be null");
}
```

### 4. Exception Handling

```java
// Good: specific exceptions
try {
    processUser(user);
} catch (UserNotFoundException e) {
    LOGGER.error("User not found: {}", e.getMessage());
} catch (DatabaseException e) {
    LOGGER.error("Database error: {}", e.getMessage());
    throw new ServiceException("Failed to process user", e);
}

// Good: try-with-resources for AutoCloseable
try (Connection connection = dataSource.getConnection();
     PreparedStatement statement = connection.prepareStatement(query)) {
    
    ResultSet resultSet = statement.executeQuery();
    // Process results
} catch (SQLException e) {
    LOGGER.error("Database error", e);
}

// Bad: catching generic Exception
try {
    // ...
} catch (Exception e) {  // Too broad
    e.printStackTrace();
}

// Bad: empty catch block
try {
    // ...
} catch (Exception e) {
    // Swallowing exceptions - BAD
}
```

### 5. String Handling

```java
// Good: use StringBuilder for concatenation in loops
public String buildReport(List<String> items) {
    StringBuilder sb = new StringBuilder();
    for (String item : items) {
        sb.append(item).append("\n");
    }
    return sb.toString();
}

// Good: use String.format for complex strings
String message = String.format("User %s (ID: %d) has %d items", 
    user.getName(), user.getId(), user.getItems().size());

// Good: use text blocks (Java 15+) for multi-line strings
String json = """
    {
        "name": "%s",
        "age": %d,
        "active": %b
    }
    """.formatted(user.getName(), user.getAge(), user.isActive());
```

### 6. Collection Best Practices

```java
// Good: specify initial capacity when known
List<User> users = new ArrayList<>(1000);
Set<String> names = new HashSet<>(500);

// Good: use interfaces as types
List<String> items = new ArrayList<>();
Map<String, User> userMap = new HashMap<>();

// Good: use Collections.unmodifiableList for immutability
public List<String> getItems() {
    return Collections.unmodifiableList(items);
}

// Good: use diamond operator
List<String> list = new ArrayList<>();  // Java 7+

// Good: use empty collections instead of null
List<String> emptyList = Collections.emptyList();
Set<String> emptySet = Collections.emptySet();
Map<String, String> emptyMap = Collections.emptyMap();
```

### 7. Method Design

```java
// Good: methods should do one thing
public User createUser(UserDto dto) {
    validateUserDto(dto);
    User user = convertToEntity(dto);
    return userRepository.save(user);
}

// Bad: method doing too many things
public void createUserAndSendEmailAndLog(UserDto dto) {
    // Too many responsibilities!
}

// Good: keep parameter list short (max 3-4 parameters)
public void processOrder(Order order, Payment payment) {
    // ...
}

// If more parameters needed, use a parameter object
public void processOrder(OrderRequest request) {
    // request contains order, payment, shipping, etc.
}

// Good: use builder pattern for many parameters
User user = User.builder()
    .email("test@example.com")
    .firstName("John")
    .lastName("Doe")
    .age(30)
    .build();
```

### 8. Comments and Documentation

```java
// Good: Javadoc for public APIs
/**
 * Calculates the total price including tax for the given order.
 *
 * @param order the order to calculate price for, must not be null
 * @param taxRate the tax rate to apply (e.g., 0.07 for 7%), must be non-negative
 * @return the total price including tax
 * @throws IllegalArgumentException if order is null or taxRate is negative
 */
public BigDecimal calculateTotalPrice(Order order, BigDecimal taxRate) {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
    if (taxRate.compareTo(BigDecimal.ZERO) < 0) {
        throw new IllegalArgumentException("Tax rate cannot be negative");
    }
    
    return order.getSubtotal().multiply(BigDecimal.ONE.add(taxRate));
}

// Bad: obvious comments
// Increment the counter
count++;  // Unnecessary

// Good: explain WHY, not WHAT
// We retry 3 times because the external API is occasionally unstable
for (int i = 0; i < 3; i++) {
    try {
        return callApi();
    } catch (ApiException e) {
        if (i == 2) {
            throw e;
        }
    }
}
```

## Code Quality Tools

### Checkstyle Configuration (checkstyle.xml)

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">
    
<module name="Checker">
    <property name="charset" value="UTF-8"/>
    <property name="severity" value="warning"/>
    
    <module name="TreeWalker">
        <!-- Naming conventions -->
        <module name="ConstantName"/>
        <module name="LocalFinalVariableName"/>
        <module name="LocalVariableName"/>
        <module name="MemberName"/>
        <module name="MethodName"/>
        <module name="PackageName"/>
        <module name="ParameterName"/>
        <module name="StaticVariableName"/>
        <module name="TypeName"/>
        
        <!-- Imports -->
        <module name="AvoidStarImport"/>
        <module name="IllegalImport"/>
        <module name="RedundantImport"/>
        <module name="UnusedImports"/>
        
        <!-- Size violations -->
        <module name="LineLength">
            <property name="max" value="100"/>
        </module>
        <module name="MethodLength">
            <property name="max" value="150"/>
        </module>
        <module name="ParameterNumber">
            <property name="max" value="7"/>
        </module>
        
        <!-- Whitespace -->
        <module name="EmptyForIteratorPad"/>
        <module name="GenericWhitespace"/>
        <module name="MethodParamPad"/>
        <module name="NoWhitespaceAfter"/>
        <module name="NoWhitespaceBefore"/>
        <module name="OperatorWrap"/>
        <module name="ParenPad"/>
        <module name="TypecastParenPad"/>
        <module name="WhitespaceAfter"/>
        <module name="WhitespaceAround"/>
        
        <!-- Modifier checks -->
        <module name="ModifierOrder"/>
        <module name="RedundantModifier"/>
        
        <!-- Blocks -->
        <module name="AvoidNestedBlocks"/>
        <module name="EmptyBlock"/>
        <module name="LeftCurly"/>
        <module name="NeedBraces"/>
        <module name="RightCurly"/>
        
        <!-- Coding -->
        <module name="EmptyStatement"/>
        <module name="EqualsHashCode"/>
        <module name="IllegalInstantiation"/>
        <module name="InnerAssignment"/>
        <module name="MissingSwitchDefault"/>
        <module name="SimplifyBooleanExpression"/>
        <module name="SimplifyBooleanReturn"/>
        
        <!-- Annotations -->
        <module name="AnnotationLocation">
            <property name="allowSamelineMultipleAnnotations" value="false"/>
            <property name="allowSamelineSingleParameterlessAnnotation" value="false"/>
            <property name="allowSamelineParameterizedAnnotation" value="false"/>
        </module>
    </module>
</module>
```

### Maven Checkstyle Plugin

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <version>3.2.0</version>
            <configuration>
                <configLocation>checkstyle.xml</configLocation>
                <encoding>UTF-8</encoding>
                <consoleOutput>true</consoleOutput>
                <failsOnError>true</failsOnError>
                <linkXRef>false</linkXRef>
            </configuration>
            <executions>
                <execution>
                    <id>validate</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### SpotBugs Configuration

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.github.spotbugs</groupId>
            <artifactId>spotbugs-maven-plugin</artifactId>
            <version>4.7.3.0</version>
            <configuration>
                <effort>Max</effort>
                <threshold>Low</threshold>
                <failOnError>true</failOnError>
                <includeFilterFile>spotbugs-include.xml</includeFilterFile>
            </configuration>
            <executions>
                <execution>
                    <id>spotbugs-check</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## Common Code Smells and Fixes

### 1. Long Method

```java
// Bad: method doing too much
public void processUserData(String userId) {
    // 200 lines of code...
}

// Good: break into smaller methods
public void processUserData(String userId) {
    User user = findUser(userId);
    validateUser(user);
    enrichUserData(user);
    saveUser(user);
    sendNotification(user);
}
```

### 2. Feature Envy

```java
// Bad: method too interested in another class's internals
public void printUserReport(User user) {
    System.out.println("Name: " + user.getName());
    System.out.println("Email: " + user.getEmail());
    System.out.println("Age: " + user.getAge());
    // ... more user fields
}

// Good: move to User class
public void printUserReport(User user) {
    user.printReport();
}
```

### 3. Duplicate Code

```java
// Bad: same logic repeated
if (user.getType() == UserType.ADMIN) {
    sendEmail(user.getEmail(), "Admin login");
    logToFile("Admin logged in: " + user.getName());
}

if (user.getType() == UserType.SUPERADMIN) {
    sendEmail(user.getEmail(), "SuperAdmin login");
    logToFile("SuperAdmin logged in: " + user.getName());
}

// Good: extract to method
private void handleUserLogin(User user, String type) {
    sendEmail(user.getEmail(), type + " login");
    logToFile(type + " logged in: " + user.getName());
}
```

## Performance Considerations

### 1. Object Creation

```java
// Bad: creating unnecessary objects
String s = new String("hello");  // Creates 2 objects

// Good: use string literals
String s = "hello";

// Bad: concatenating strings in loop
String result = "";
for (Item item : items) {
    result += item.toString();  // Creates new String each iteration
}

// Good: use StringBuilder
StringBuilder sb = new StringBuilder();
for (Item item : items) {
    sb.append(item);
}
```

### 2. Collection Performance

```java
// Good: use ArrayList for frequent random access
List<String> list = new ArrayList<>();

// Good: use LinkedList for frequent insertions/deletions
List<String> list = new LinkedList<>();

// Good: use HashSet for O(1) lookups
Set<String> set = new HashSet<>();

// Good: use LinkedHashMap for predictable iteration order
Map<String, User> map = new LinkedHashMap<>();
```

## IDE Configuration

### IntelliJ IDEA Settings

1. **Code Style**: Settings → Editor → Code Style → Java
   - Select "Google Style Guide" or import custom style
2. **Imports**: Organize imports alphabetically
3. **Formatter**: Enable format on save
4. **CheckStyle**: Install CheckStyle-IDEA plugin
5. **SpotBugs**: Install SpotBugs plugin

### VS Code Settings

Install extensions:
- "Java Extension Pack"
- "Checkstyle for Java"
- "SonarLint"

Configure in `.vscode/settings.json`:
```json
{
    "java.format.settings.url": "https://raw.githubusercontent.com/google/styleguide/gh-pages/eclipse-java-google-style.xml",
    "java.saveActions.organizeImports": true,
    "java.format.enabled": true
}
```

## Review Checklist

When reviewing Java code, check for:

- [ ] Proper naming conventions (classes, methods, variables)
- [ ] Appropriate use of `final` keyword
- [ ] Null safety handled correctly
- [ ] Exceptions handled appropriately (specific, not swallowed)
- [ ] No magic numbers or strings
- [ ] Methods are focused and do one thing
- [ ] Proper use of collections and generics
- [ ] Documentation (Javadoc) for public APIs
- [ ] No code duplication
- [ ] Consistent code formatting
- [ ] Appropriate logging (not System.out.println)
- [ ] Resource management (try-with-resources)
- [ ] Thread safety considered (where applicable)
- [ ] No TODO comments without explanation
