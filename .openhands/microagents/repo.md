
# OpenHands Skills Registry

This is the official public registry of **Skills** for OpenHands - reusable guidelines that customize agent behavior across various technologies, frameworks, and best practices. Skills were previously called "Microagents", and the `.openhands/microagents/` folder path remains for backward compatibility.

## Project Overview

The registry contains domain-specific skills that provide OpenHands agents with expert knowledge for technologies like Java (Maven, Gradle, Spring Boot), React + TypeScript + Chakra-UI, Docker, Kubernetes, GitHub/GitLab workflows, and more. Each skill is triggered by relevant keywords and provides context-aware guidance.

## File Structure

```
skills/
├── skills/                    # All available skill directories
│   ├── maven/                 # Apache Maven build automation
│   ├── gradle/                # Gradle build tool
│   ├── spring-boot/           # Spring Boot framework
│   ├── java-testing/          # JUnit, Mockito, Testcontainers
│   ├── react-typescript/      # React + TypeScript patterns
│   ├── chakra-ui/             # Chakra-UI component library
│   ├── github/                # GitHub API and workflows
│   ├── gitlab/                # GitLab merge requests
│   ├── docker/                # Container management
│   ├── kubernetes/            # K8s orchestration
│   └── [other skills...]      
├── tests/                     # Skills validation tests
├── AGENTS.md                  # Detailed project guidelines
├── README.md                  # Repository overview
└── LICENSE                    # MIT License
```

Each skill directory contains a `SKILL.md` file with frontmatter (name, description, triggers) and detailed documentation.

## Running Tests

Skills are validated automatically to ensure each has a `README.md` file:

```bash
# Run all validation tests
npm test

# Or run directly with Python
python tests/test_skills_have_readme.py
```

## Key Information for Developers

- **Skill Format**: YAML frontmatter with `name`, `description`, `triggers` followed by markdown content
- **Creating Skills**: Add a new directory under `skills/` with a `SKILL.md` file
- **Triggers**: Define keywords that auto-activate the skill during conversations
- **Best Practices**: Skills should be specific, actionable, include examples, and follow industry standards
- **Documentation**: See `AGENTS.md` for comprehensive contribution guidelines and skill standards
