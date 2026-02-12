# Java Development Skills

This directory contains comprehensive skills for Java development, covering build tools, frameworks, testing, and code quality.

## Available Skills

### 1. Maven (`maven/`)
**Purpose**: Build automation and dependency management for Java projects using Apache Maven.

**Triggers**: `maven`, `pom.xml`, `mvn`, `maven build`, `maven dependency`

**What it covers**:
- Maven project structure and conventions
- Essential Maven commands (build, test, package, install, deploy)
- Dependency management (scopes, conflict resolution)
- Understanding pom.xml configuration
- Maven lifecycle phases
- Best practices and common issues
- Useful Maven plugins

**When to use**: Working with Maven projects, managing dependencies, running builds, or handling Maven lifecycle phases.

---

### 2. Gradle (`gradle/`)
**Purpose**: Modern build automation tool for Java projects using Gradle.

**Triggers**: `gradle`, `build.gradle`, `settings.gradle`, `gradlew`, `gradle build`, `gradle task`

**What it covers**:
- Gradle project structure (Groovy DSL and Kotlin DSL)
- Essential Gradle commands and tasks
- Dependency configurations (implementation, api, compileOnly, etc.)
- Multi-project builds
- Gradle Wrapper usage
- Performance optimization (parallel builds, caching)
- Common issues and solutions
- Integration with IDEs

**When to use**: Working with Gradle projects, managing dependencies, running Gradle tasks, or setting up build automation.

---

### 3. Spring Boot (`spring-boot/`)
**Purpose**: Spring Boot framework for building production-ready applications.

**Triggers**: `spring-boot`, `spring boot`, `@SpringBootApplication`, `@RestController`, `@Autowired`, `@Service`, `@Repository`, `@Entity`, `application.properties`, `application.yml`

**What it covers**:
- Spring Boot project structure
- Core annotations (@RestController, @Service, @Repository, @Entity)
- Configuration management (application.yml, profiles)
- Dependency injection (constructor, setter, field injection)
- REST API development
- Exception handling (@ControllerAdvice)
- Data access with Spring Data JPA
- Validation (Bean Validation API)
- Testing strategies
- Common dependencies and best practices

**When to use**: Developing Spring Boot applications, configuring beans, working with Spring annotations, or setting up REST APIs.

---

### 4. Java Testing (`java-testing/`)
**Purpose**: Java testing frameworks and best practices.

**Triggers**: `junit`, `mockito`, `@Test`, `@Mock`, `unit test`, `integration test`, `test case`, `testing java`, `testcontainers`

**What it covers**:
- JUnit 5 basics (annotations, assertions, lifecycle hooks)
- Parameterized tests
- Mockito (mocking, stubbing, verification)
- Integration testing with Testcontainers
- AssertJ fluent assertions
- Testing best practices (AAA pattern, naming conventions)
- Spring Boot testing (@WebMvcTest, @DataJpaTest, @SpringBootTest)
- Performance testing
- Maven configuration for testing

**When to use**: Writing unit tests, integration tests, or setting up test infrastructure for Java applications.

---

### 5. Java Style (`java-style/`)
**Purpose**: Java code style conventions and coding standards.

**Triggers**: `code style`, `coding standards`, `java convention`, `checkstyle`, `spotbugs`, `code formatting`, `code review`

**What it covers**:
- Naming conventions (classes, methods, variables, constants)
- Class organization and structure
- Code formatting (indentation, braces, line length)
- Best practices (immutability, null safety, exception handling)
- String and collection handling
- Method design principles
- Comments and documentation
- Code quality tools (Checkstyle, SpotBugs)
- Common code smells and fixes
- Performance considerations
- IDE configuration

**When to use**: Reviewing Java code, setting up linting tools, or ensuring consistent code style across the project.

---

## Quick Start

### For Maven Projects
1. Use the `maven` skill when you see `pom.xml` or need to run Maven commands
2. Run `mvn clean install` to build and test
3. Use `mvn dependency:tree` to analyze dependencies

### For Gradle Projects
1. Use the `gradle` skill when you see `build.gradle` or need to run Gradle tasks
2. Run `./gradlew build` to build and test
3. Use `./gradlew dependencies` to view dependency tree

### For Spring Boot Applications
1. Use the `spring-boot` skill for Spring Boot specific guidance
2. Follow the layered architecture pattern (controller → service → repository)
3. Use constructor injection and proper exception handling

### For Testing
1. Use the `java-testing` skill when writing tests
2. Prefer JUnit 5 with AssertJ for better assertions
3. Use Testcontainers for integration tests with real databases
4. Follow AAA pattern (Arrange-Act-Assert)

### For Code Quality
1. Use the `java-style` skill for code review and style guidance
2. Set up Checkstyle and SpotBugs in your build process
3. Follow Google Java Style Guide or configure your own standards

## Integration with Development Workflow

These skills are designed to work together:

1. **Setup**: Use `maven` or `gradle` to set up your project
2. **Development**: Use `spring-boot` for application development
3. **Testing**: Use `java-testing` to write comprehensive tests
4. **Quality**: Use `java-style` to ensure code quality and consistency

## Contributing

To add more Java-related skills or improve existing ones:

1. Create a new directory under `/skills/`
2. Add a `SKILL.md` file with proper frontmatter
3. Include practical examples and best practices
4. Test the triggers to ensure they work as expected

## Additional Resources

- [Apache Maven Documentation](https://maven.apache.org/guides/)
- [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html)
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
