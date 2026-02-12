# OpenHands Skills Registry

This repository contains community-shared skills for OpenHands agents.

## Repository Overview

The **OpenHands Skills Registry** is the official global repository for sharing and distributing skills that enhance OpenHands agent capabilities across various technologies, frameworks, and best practices.

## What Are Skills?

Skills are specialized prompts that provide OpenHands agents with:
- **Domain-specific knowledge** - Technical expertise for specific technologies
- **Best practices** - Industry-standard coding conventions and patterns
- **Automated guidance** - Context-aware instructions triggered by keywords

## Repository Structure

```
skills/
├── skills/                    # All available skills
│   ├── java-skills-README.md  # Java skills overview
│   ├── react-typescript-skills-README.md  # React skills overview
│   ├── maven/                 # Java build tools
│   ├── gradle/                # Java build tools
│   ├── spring-boot/           # Java framework
│   ├── java-testing/          # Java testing frameworks
│   ├── java-style/            # Java code conventions
│   ├── react-typescript/      # React + TypeScript
│   ├── chakra-ui/             # UI component library
│   ├── react-hooks/           # React patterns
│   ├── react-components/      # Component design
│   └── ts-react-patterns/     # Architecture patterns
├── AGENTS.md                  # This file - project guidelines
├── README.md                  # Repository documentation
└── tests/                     # Skills validation tests
```

## Available Skills

### Java Development Skills

#### 1. Maven (`skills/maven/`)
**Purpose**: Build automation and dependency management with Apache Maven
**Triggers**: `maven`, `pom.xml`, `mvn`, `maven build`

**Covers**:
- Maven project structure and lifecycle
- Dependency management and resolution
- Plugin configuration
- Best practices and common issues

#### 2. Gradle (`skills/gradle/`)
**Purpose**: Modern build automation with Gradle
**Triggers**: `gradle`, `build.gradle`, `gradlew`

**Covers**:
- Gradle DSL (Groovy and Kotlin)
- Multi-project builds
- Dependency configurations
- Performance optimization

#### 3. Spring Boot (`skills/spring-boot/`)
**Purpose**: Spring Boot framework for production-ready applications
**Triggers**: `spring-boot`, `@RestController`, `application.yml`

**Covers**:
- Spring Boot project structure
- Core annotations and patterns
- REST API development
- Data access with Spring Data JPA
- Configuration management

#### 4. Java Testing (`skills/java-testing/`)
**Purpose**: Testing frameworks and best practices
**Triggers**: `junit`, `mockito`, `@Test`, `testcontainers`

**Covers**:
- JUnit 5 testing framework
- Mockito mocking framework
- Testcontainers for integration tests
- Testing patterns and best practices

#### 5. Java Style (`skills/java-style/`)
**Purpose**: Code style conventions and quality standards
**Triggers**: `code style`, `checkstyle`, `spotbugs`

**Covers**:
- Naming conventions
- Code formatting standards
- Best practices
- Checkstyle and SpotBugs configuration

### React + TypeScript Skills

#### 1. React + TypeScript (`skills/react-typescript/`)
**Purpose**: React development with TypeScript best practices
**Triggers**: `react`, `typescript`, `.tsx`, `react component`

**Covers**:
- Component type definitions
- Advanced props patterns
- Type-safe event handlers
- Custom hooks with TypeScript
- Context and state patterns

#### 2. Chakra-UI (`skills/chakra-ui/`)
**Purpose**: Chakra-UI component library
**Triggers**: `chakra`, `@chakra-ui`, `useToast`, `<Box>`, `<Flex>`

**Covers**:
- Component priority (project components first)
- Layout primitives
- Theme customization
- Form handling
- Responsive design

**Key Principle**: Always prioritize project components over Chakra primitives

#### 3. React Hooks (`skills/react-hooks/`)
**Purpose**: Built-in and custom React hooks patterns
**Triggers**: `usehook`, `usestate`, `useeffect`, `custom hook`

**Covers**:
- Built-in hooks (useState, useEffect, useContext, etc.)
- Custom hooks (useFetch, useForm, useLocalStorage, etc.)
- Best practices and common patterns

#### 4. React Components (`skills/react-components/`)
**Purpose**: Component design patterns and composition
**Triggers**: `component composition`, `compound components`, `render props`

**Covers**:
- Container/Presentational pattern
- Compound components
- Render props
- Higher-Order Components
- Performance optimization

#### 5. TypeScript + React Patterns (`skills/ts-react-patterns/`)
**Purpose**: Architectural patterns for scalable applications
**Triggers**: `project structure`, `architecture patterns`, `type-safe react`

**Covers**:
- Feature-based architecture
- Type-safe API layer
- Form patterns
- Context patterns
- Utility types

## How to Use These Skills

### For OpenHands Users

When working with OpenHands, skills are automatically activated based on keywords in your conversation:

```bash
# Java examples
"Create a Maven project"           # Activates Maven skill
"Build a Spring Boot API"          # Activates Spring Boot skill
"Write JUnit tests"                # Activates Java Testing skill

# React examples
"Create a React component"         # Activates React+TypeScript skill
"Use Chakra-UI for layout"         # Activates Chakra-UI skill
"Implement a custom hook"          # Activates React Hooks skill
```

### For Project Integration

#### Option 1: Use Directly from This Repository

Clone this repository and reference skills in your project:

```bash
# Clone the skills registry
git clone https://github.com/OpenHands/skills.git ~/openhands-skills

# Copy skills to your project
cp -r ~/openhands-skills/skills/maven .agents/skills/
cp -r ~/openhands-skills/skills/react-typescript .agents/skills/
```

#### Option 2: Add to Your Project

Create `.agents/skills/` in your project and copy needed skills:

```bash
your-project/
├── .agents/
│   └── skills/
│       ├── maven/
│       ├── react-typescript/
│       └── chakra-ui/
├── AGENTS.md
└── src/
```

## Skill Format

Each skill follows the AgentSkills standard:

```yaml
---
name: skill-name
description: Brief description of the skill
triggers:
  - keyword1
  - keyword2
---

# Skill Content

Detailed documentation and examples...
```

## Contributing

We welcome contributions! Please:

1. **Fork this repository**
2. **Create a skill directory**: `mkdir skills/your-skill`
3. **Add SKILL.md** with proper frontmatter
4. **Include examples** and best practices
5. **Submit a pull request**

### Skill Guidelines

- ✅ **Specific and actionable** - Clear, focused instructions
- ✅ **Include examples** - Practical code samples
- ✅ **Define triggers** - Relevant keywords for auto-activation
- ✅ **Best practices** - Industry-standard patterns
- ✅ **Well-documented** - Clear explanations

## Testing Skills

Skills are validated automatically:

```bash
# Run skill validation tests
npm test

# Check specific skill
npm test -- skills/maven
```

## Documentation

- **Main README**: See [README.md](README.md) for repository information
- **Java Skills**: See [java-skills-README.md](skills/java-skills-README.md)
- **React Skills**: See [react-typescript-skills-README.md](skills/react-typescript-skills-README.md)

## License

See [LICENSE](LICENSE) file for details.

## Support

- **Issues**: [GitHub Issues](https://github.com/OpenHands/skills/issues)
- **Discussions**: [GitHub Discussions](https://github.com/OpenHands/skills/discussions)
- **Documentation**: [OpenHands Docs](https://docs.openhands.dev)

---

**Repository**: https://github.com/OpenHands/skills  
**Community**: Join us in building the best skills library for OpenHands agents!
