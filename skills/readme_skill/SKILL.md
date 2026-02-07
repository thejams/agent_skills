---
name: readme-generator
title: Professional README Generator
description: Analyzes codebase and generates comprehensive README.md with setup, examples, architecture, and badges
activation_regex: "(generate|create|write|update|improve|document) (README|docs?|documentation)"
allowed_tools:
  - view
  - ls
  - cat
  - tree
  - grep
parameters:
  mode:
    type: string
    description: "enhance|regenerate|analyze"
    default: "enhance"
  project_type:
    type: string
    description: "library|app|cli|microservice|framework"
    optional: true
---

## Skill Activation Triggers

Use this skill when the user's request matches ANY of these patterns:
- "generate README"
- "create README"
- "write a README"
- "update README"
- "improve README"
- "document this project"
- "create project documentation"
- "write documentation"
- "generate docs"
- References README.md or project documentation

## Skill Objectives

This skill analyzes a codebase and generates comprehensive, professional README.md files that:
1. Automatically detect project type and technology stack
2. Document architecture and design patterns
3. Provide clear setup and usage instructions
4. Include relevant badges and links
5. Follow best practices for open-source documentation

## Critical Framework Rules

### Rule 1: Always Start with Discovery
**Before writing ANY documentation, Claude MUST:**
1. Scan project structure to understand organization
2. Detect technology stack (languages, frameworks, tools)
3. Identify configuration files (package.json, go.mod, requirements.txt, etc.)
4. Find existing documentation (README, CONTRIBUTING, docs/)
5. Check for CI/CD, testing, and build configurations

### Rule 2: Adapt to Project Context
**Different projects need different documentation:**
- Library/Package → API reference, installation, usage examples
- Application → Setup, configuration, deployment
- Framework/Boilerplate → Architecture, patterns, extension guide
- CLI Tool → Installation, commands, examples
- Microservices → Service architecture, communication patterns

### Rule 3: Preserve Existing Content
**When updating existing README:**
- Read current README first
- Identify sections to preserve (badges, license, contributors)
- Merge new content with existing
- Maintain the project's voice and style
- Ask before removing significant content

### Rule 4: Include Working Examples
**Every README MUST include:**
- Actual code examples that work
- Real configuration snippets
- Concrete usage scenarios
- No placeholder text like "TODO" or "Coming soon"

### Rule 5: Make it Scannable
**Structure for quick comprehension:**
- Clear hierarchy (H1 → H2 → H3)
- Table of contents for long READMEs
- Code blocks with syntax highlighting
- Badges at the top
- Quick start section early

## Workflow: README Generation Process

### Phase 0: Project Discovery (MANDATORY FIRST STEP)

**Claude must execute these steps:**

1. **Check for existing README:**
   ```bash
   view README.md
   view README
   view readme.md
   ```

2. **If README exists, ask user:**
   ```
   I found an existing README.md. Would you like me to:
   
   1. Enhance it (preserve content, add missing sections)
   2. Regenerate it (start fresh, may incorporate existing content)
   3. Analyze and suggest improvements only
   
   Select an option.
   ```

3. **Scan project structure:**
   ```bash
   # Get directory tree (max 3 levels)
   ls -R | head -100
   
   # Or if tree is available:
   tree -L 3 -I 'node_modules|vendor|venv|.git'
   ```

4. **Detect project type and tech stack:**
   ```bash
   # Check for language/framework indicators
   ls -la | grep -E "package\.json|go\.mod|requirements\.txt|Cargo\.toml|pom\.xml|Gemfile|composer\.json"
   
   # Check for configuration files
   ls -la | grep -E "\.config\.|docker-compose|Dockerfile|Makefile|tsconfig|webpack|vite"
   
   # Check for documentation
   ls -la docs/ 2>/dev/null
   ls -la .github/ 2>/dev/null
   ```

5. **Read key configuration files:**
   ```bash
   view package.json       # Node.js/JavaScript
   view go.mod            # Go
   view requirements.txt  # Python
   view Cargo.toml        # Rust
   view pom.xml           # Java
   view Gemfile           # Ruby
   view composer.json     # PHP
   ```

6. **Show user the detection results:**
   ```
   📊 Project Analysis:
   
   Type: [Detected type]
   Language: [Primary language]
   Framework: [If detected]
   Build Tool: [Make, npm, cargo, etc.]
   
   Key Files Found:
   - [List important files]
   
   Existing Documentation:
   - [List found docs]
   
   Proceed with README generation?
   ```

**Wait for user confirmation before proceeding.**

---

### Phase 1: Technology Stack Detection

**Automatically identify the project context:**

**Detection Logic:**

```javascript
// Node.js/JavaScript/TypeScript
IF files contain (package.json):
  → Read package.json for dependencies
  → Check for React, Vue, Angular, Express, Next.js, etc.
  → Identify if it's a library, app, or CLI tool
  → Check for TypeScript (tsconfig.json)

// Go
IF files contain (go.mod):
  → Read go.mod for module name and dependencies
  → Check for common frameworks (Gin, Echo, Fiber, gRPC)
  → Identify if it's a CLI, service, or library
  → Look for main.go location

// Python
IF files contain (requirements.txt, setup.py, pyproject.toml):
  → Read dependencies
  → Check for Django, Flask, FastAPI
  → Identify if it's a library, app, or CLI
  → Check for virtual environment setup

// Rust
IF files contain (Cargo.toml):
  → Read Cargo.toml for package info
  → Check if it's a binary or library
  → Identify dependencies

// Java
IF files contain (pom.xml, build.gradle):
  → Detect Maven or Gradle
  → Read dependencies
  → Check for Spring Boot, Quarkus

// Ruby
IF files contain (Gemfile):
  → Check for Rails, Sinatra
  → Identify gem dependencies

// PHP
IF files contain (composer.json):
  → Check for Laravel, Symfony
  → Read dependencies

// Container/Infrastructure
IF files contain (Dockerfile, docker-compose.yml):
  → Note containerization
  → Check for multi-stage builds

// CI/CD
IF files contain (.github/workflows, .gitlab-ci.yml, .travis.yml):
  → Note automated testing/deployment
```

**Show user:**
```
🔍 Technology Stack Detected:

Primary Language: [Language + Version]
Framework/Library: [If applicable]
Key Dependencies:
- [Dep 1]
- [Dep 2]
- [Dep 3]

Build Tools:
- [npm, cargo, make, etc.]

Infrastructure:
- [Docker, Kubernetes, etc.]

Testing:
- [Test frameworks detected]
```

---

### Phase 2: Architecture Analysis

**Understand the project structure:**

**For Applications:**
```bash
# Identify architectural patterns
- MVC (Model-View-Controller)
- Microservices
- Monolithic
- Client-Server
- Layered Architecture
- Event-Driven

# Detect by structure:
IF has (controllers/, models/, views/) → MVC
IF has (services/, each with own dependencies) → Microservices
IF has (cmd/, pkg/, internal/) → Go standard layout
IF has (src/components/, src/pages/) → React/Vue app
IF has (api/, frontend/, backend/) → Full-stack separation
```

**For Libraries:**
```bash
# Check for:
- Public API surface (exported functions/classes)
- Module structure
- Plugin system
- Extension points
```

**For CLI Tools:**
```bash
# Check for:
- Command structure
- Subcommands
- Flag definitions
- Help text
```

**Document patterns found:**
```
📐 Architecture Pattern: [Detected pattern]

Project Structure:
```
[Directory tree highlighting key areas]
```

Key Components:
- [Component 1]: [Purpose]
- [Component 2]: [Purpose]

Design Patterns Used:
- [Pattern 1]
- [Pattern 2]
```

---

### Phase 3: README Structure Generation

**Generate README following this template structure:**

```markdown
# Project Title

[Badges: Build Status, Coverage, Version, License, etc.]

[One-line description]

[Optional: Screenshot, demo GIF, or architecture diagram]

## Table of Contents

- [Features](#features)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Architecture](#architecture) (if complex)
- [API Reference](#api-reference) (if library)
- [Development](#development)
- [Testing](#testing)
- [Deployment](#deployment) (if applicable)
- [Contributing](#contributing)
- [License](#license)

## Features

- ✨ Feature 1
- 🚀 Feature 2
- 📦 Feature 3

## Quick Start

[Minimal example to get running in < 5 minutes]

## Installation

### Prerequisites

- [Requirement 1]
- [Requirement 2]

### Install

[Platform-specific installation instructions]

## Usage

### Basic Example

[Working code example]

### Advanced Usage

[More complex examples]

## Configuration

[Environment variables, config files, options]

## Architecture

[High-level architecture explanation]
[Component diagram if complex]

## API Reference

[For libraries - document public API]

## Development

### Setup Development Environment

[How to set up for contributing]

### Project Structure

[Explain directory organization]

### Build

[How to build from source]

### Run Locally

[How to run for development]

## Testing

[How to run tests]

## Deployment

[How to deploy to production]

## Contributing

[How to contribute]

## License

[License information]
```

**Sections to include based on project type:**

| Project Type | Required Sections | Optional Sections |
|--------------|------------------|-------------------|
| Library | Installation, Usage, API Reference | Examples, Plugins |
| Application | Quick Start, Configuration, Deployment | Architecture |
| CLI Tool | Installation, Commands, Examples | Configuration |
| Framework | Architecture, Getting Started, Extension | Plugins, Recipes |
| Microservices | Architecture, Services, Deployment | Communication Patterns |

---

### Phase 4: Content Generation

**For each section, generate specific content:**

#### 4.1: Title and Badges

```markdown
# Project Name

[![Build Status](https://img.shields.io/github/workflow/status/user/repo/CI)](link)
[![Coverage](https://img.shields.io/codecov/c/github/user/repo)](link)
[![Version](https://img.shields.io/npm/v/package-name)](link)
[![License](https://img.shields.io/github/license/user/repo)](link)
[![Stars](https://img.shields.io/github/stars/user/repo)](link)
```

**Badges to include based on project:**
- Build/CI status (GitHub Actions, Travis, CircleCI)
- Code coverage (Codecov, Coveralls)
- Package version (npm, PyPI, crates.io, etc.)
- License
- Downloads/Installs
- Language version
- Dependencies status
- Documentation
- Chat/Community (Discord, Slack)

---

#### 4.2: Description and Features

**Extract from:**
- package.json: `description` field
- go.mod: Comments
- setup.py: `description` field
- Existing README
- Code comments in main files

**Features section:**
- Scan code for major functionality
- Check configuration files for capabilities
- Review dependencies for integrations
- List 5-10 key features with emojis

**Example:**
```markdown
A production-ready microservices framework built on Pydio Cells.

## Features

- 🚀 **Fast gRPC Communication** - High-performance inter-service calls
- 📦 **Auto-Discovery** - Services register automatically
- 🔍 **Full Observability** - Logging, tracing, and metrics built-in
- 🔐 **Enterprise Auth** - LDAP, SAML, OAuth support
- 🐳 **Container-Ready** - Docker and Kubernetes configs included
- 📊 **OpenAPI Spec** - Auto-generated REST documentation
```

---

#### 4.3: Quick Start

**Generate minimal working example:**

**For Applications:**
```bash
# Clone the repository
git clone https://github.com/user/repo.git
cd repo

# Install dependencies
[package manager install command]

# Run the application
[run command]

# Open in browser (if web app)
http://localhost:[port]
```

**For Libraries:**
```bash
# Install
[package manager add command]

# Use in code
[minimal code example]
```

**For CLI Tools:**
```bash
# Install
[installation method]

# Run
[command] --help
```

---

#### 4.4: Installation

**Platform-specific instructions:**

**Node.js:**
```markdown
### Prerequisites

- Node.js >= 16.x
- npm >= 8.x (or yarn/pnpm)

### Install via npm

\`\`\`bash
npm install package-name
\`\`\`

### Install from source

\`\`\`bash
git clone https://github.com/user/repo.git
cd repo
npm install
npm run build
\`\`\`
```

**Go:**
```markdown
### Prerequisites

- Go >= 1.21

### Install

\`\`\`bash
go install github.com/user/repo@latest
\`\`\`

### Build from source

\`\`\`bash
git clone https://github.com/user/repo.git
cd repo
make build
# or
go build -o binary ./cmd/main.go
\`\`\`
```

**Python:**
```markdown
### Prerequisites

- Python >= 3.9
- pip

### Install via pip

\`\`\`bash
pip install package-name
\`\`\`

### Install from source

\`\`\`bash
git clone https://github.com/user/repo.git
cd repo
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -e .
\`\`\`
```

**Rust:**
```markdown
### Prerequisites

- Rust >= 1.70

### Install via cargo

\`\`\`bash
cargo install package-name
\`\`\`

### Build from source

\`\`\`bash
git clone https://github.com/user/repo.git
cd repo
cargo build --release
\`\`\`
```

---

#### 4.5: Usage Examples

**Generate working code examples:**

**Scan for:**
- Example files (examples/, samples/)
- Test files (for usage patterns)
- Main entry points
- Documentation comments

**Create examples that:**
- Actually work (no placeholders)
- Show common use cases
- Progress from simple to complex
- Include expected output

**Example format:**
```markdown
## Usage

### Basic Example

\`\`\`javascript
const lib = require('package-name');

// Create an instance
const instance = new lib.Client({
  apiKey: 'your-api-key'
});

// Make a request
const result = await instance.doSomething();
console.log(result);
\`\`\`

### Advanced Example

\`\`\`javascript
// Configure with all options
const instance = new lib.Client({
  apiKey: process.env.API_KEY,
  timeout: 5000,
  retries: 3,
  logger: console
});

// Use middleware
instance.use(lib.middleware.logging());

// Handle errors
try {
  const result = await instance.doSomething();
} catch (error) {
  console.error('Operation failed:', error);
}
\`\`\`

**Output:**
\`\`\`
{ success: true, data: {...} }
\`\`\`
```

---

#### 4.6: Configuration

**Document all configuration options:**

**Scan for:**
- Environment variables (in code or .env.example)
- Config files (config.json, .yaml, etc.)
- Command-line flags
- Initialization options

**Format:**
```markdown
## Configuration

### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `API_KEY` | API authentication key | - | Yes |
| `PORT` | Server port | `8080` | No |
| `LOG_LEVEL` | Logging level (debug, info, error) | `info` | No |

### Configuration File

Create a \`config.yml\`:

\`\`\`yaml
server:
  port: 8080
  host: "0.0.0.0"

database:
  url: "postgresql://localhost/db"
  max_connections: 10

logging:
  level: "info"
  format: "json"
\`\`\`

### CLI Flags

\`\`\`bash
--port, -p     Server port (default: 8080)
--config, -c   Config file path
--verbose, -v  Enable verbose logging
--help, -h     Show help
\`\`\`
```

---

#### 4.7: Architecture (For Complex Projects)

**Generate architecture documentation:**

**Include:**
- High-level architecture diagram (ASCII or link to image)
- Component descriptions
- Communication patterns
- Data flow

**Example:**
```markdown
## Architecture

### Overview

This project follows a microservices architecture with the following components:

\`\`\`
┌─────────────────────────────────────────────────┐
│                    Gateway                      │
│          (REST → gRPC translation)             │
└────────────┬────────────────────────────────────┘
             │
             ├─────────────┬─────────────┬──────────
             │             │             │
        ┌────▼───┐    ┌────▼───┐   ┌────▼───┐
        │Service │    │Service │   │Service │
        │   A    │    │   B    │   │   C    │
        └────┬───┘    └────┬───┘   └────┬───┘
             │             │             │
             └─────────────┴─────────────┴──────────
                           │
                      ┌────▼───┐
                      │Database│
                      └────────┘
\`\`\`

### Components

#### Gateway Service
- **Purpose**: Exposes REST API to external clients
- **Tech**: gRPC-Gateway
- **Port**: 8080 (HTTPS), 8000 (gRPC)

#### Service A
- **Purpose**: User management and authentication
- **Tech**: Go + PostgreSQL
- **Communication**: gRPC

#### Service B
- **Purpose**: Business logic processing
- **Tech**: Go + Redis
- **Communication**: gRPC + Message Queue

#### Service C
- **Purpose**: Data analytics
- **Tech**: Go + TimescaleDB
- **Communication**: gRPC

### Communication Patterns

1. **Synchronous**: gRPC for direct service-to-service calls
2. **Asynchronous**: NATS for event publishing
3. **Caching**: Redis for shared state

### Data Flow

1. Client sends REST request to Gateway
2. Gateway translates to gRPC and routes to Service A
3. Service A processes and may call Service B
4. Service B publishes event to message queue
5. Service C consumes event asynchronously
6. Response flows back through Gateway to client
```

---

#### 4.8: API Reference (For Libraries)

**Auto-generate from code:**

**Scan for:**
- Exported functions/classes
- JSDoc/GoDoc/docstrings
- Type definitions
- Parameter descriptions

**Format:**
```markdown
## API Reference

### Class: Client

#### Constructor

\`\`\`javascript
new Client(options)
\`\`\`

**Parameters:**
- \`options\` (Object)
  - \`apiKey\` (string, required) - API authentication key
  - \`timeout\` (number, optional) - Request timeout in ms (default: 5000)
  - \`retries\` (number, optional) - Number of retries (default: 3)

**Returns:** Client instance

**Example:**
\`\`\`javascript
const client = new Client({ apiKey: 'abc123' });
\`\`\`

#### Methods

##### client.doSomething(params)

Performs an operation.

**Parameters:**
- \`params\` (Object)
  - \`id\` (string, required) - Resource ID
  - \`options\` (Object, optional) - Additional options

**Returns:** Promise<Result>

**Throws:** Error if operation fails

**Example:**
\`\`\`javascript
const result = await client.doSomething({ id: '123' });
\`\`\`
```

---

#### 4.9: Development Setup

```markdown
## Development

### Prerequisites

- [Tool 1] >= version
- [Tool 2] >= version

### Setup

\`\`\`bash
# Clone repository
git clone https://github.com/user/repo.git
cd repo

# Install dependencies
[install command]

# Set up development environment
[setup commands]
\`\`\`

### Project Structure

\`\`\`
project/
├── cmd/              # Command-line applications
├── pkg/              # Public libraries
├── internal/         # Private application code
├── api/              # API definitions (proto, OpenAPI)
├── web/              # Web UI assets
├── configs/          # Configuration files
├── scripts/          # Build and maintenance scripts
├── test/             # Integration tests
└── docs/             # Documentation
\`\`\`

### Build

\`\`\`bash
# Development build
[dev build command]

# Production build
[prod build command]
\`\`\`

### Run Locally

\`\`\`bash
# Start development server
[dev server command]

# The service will be available at:
# - REST API: http://localhost:8080
# - gRPC: localhost:8000
\`\`\`

### Code Style

We use [linter/formatter] for code quality:

\`\`\`bash
# Check code style
[lint command]

# Auto-fix issues
[format command]
\`\`\`
```

---

#### 4.10: Testing

```markdown
## Testing

### Run All Tests

\`\`\`bash
[test command]
\`\`\`

### Run Specific Tests

\`\`\`bash
# Unit tests only
[unit test command]

# Integration tests
[integration test command]

# End-to-end tests
[e2e test command]
\`\`\`

### Coverage

\`\`\`bash
# Generate coverage report
[coverage command]

# View coverage in browser
[coverage view command]
\`\`\`

### Test Structure

\`\`\`
test/
├── unit/             # Unit tests
├── integration/      # Integration tests
├── e2e/              # End-to-end tests
└── fixtures/         # Test data
\`\`\`

### Writing Tests

Example test:

\`\`\`[language]
[Example test code]
\`\`\`
```

---

#### 4.11: Deployment

```markdown
## Deployment

### Docker

\`\`\`bash
# Build image
docker build -t app-name:latest .

# Run container
docker run -p 8080:8080 app-name:latest
\`\`\`

### Docker Compose

\`\`\`bash
docker-compose up -d
\`\`\`

### Kubernetes

\`\`\`bash
# Apply configurations
kubectl apply -f k8s/

# Check status
kubectl get pods

# View logs
kubectl logs -f deployment/app-name
\`\`\`

### Environment Variables

Production environment variables:

\`\`\`bash
export API_KEY="your-production-key"
export DATABASE_URL="postgresql://prod-host/db"
export LOG_LEVEL="info"
\`\`\`

### Health Checks

- **Liveness**: \`GET /health\`
- **Readiness**: \`GET /ready\`

### Monitoring

The application exposes metrics at:
- Prometheus: \`GET /metrics\`
- Health: \`GET /health\`
```

---

### Phase 5: Polish and Finalize

**Review and enhance the README:**

1. **Check for completeness:**
   - [ ] All major sections present
   - [ ] Code examples work
   - [ ] Links are valid
   - [ ] Badges are correct
   - [ ] Table of contents is accurate

2. **Add polish:**
   - Emojis for visual interest (but not excessive)
   - Consistent formatting
   - Proper code block languages
   - Clear section hierarchy

3. **Validate:**
   - Test all commands
   - Verify links
   - Check for typos
   - Ensure consistent terminology

4. **Optimize for GitHub:**
   - Use relative links for internal docs
   - Add anchors for table of contents
   - Ensure images are accessible
   - Consider adding .github/README.md templates

---

### Phase 6: Generate Output

**Create the final README:**

```markdown
✅ Generated README.md

📄 Sections Included:
- Title and badges
- Description
- Features (X items)
- Quick Start
- Installation
- Usage (X examples)
- Configuration (X options)
- Architecture (if complex)
- API Reference (if library)
- Development Setup
- Testing
- Deployment
- Contributing
- License

📊 Statistics:
- Total lines: X
- Code examples: X
- Sections: X
- Estimated reading time: X min

💡 Recommendations:
- [Suggestion 1]
- [Suggestion 2]

Would you like me to:
1. Save this as README.md
2. Show you the full content first
3. Make adjustments to specific sections
```

---

## Template Library

### Minimal README (For Simple Projects)

```markdown
# Project Name

One-line description.

## Install

\`\`\`bash
[install command]
\`\`\`

## Usage

\`\`\`[language]
[minimal example]
\`\`\`

## License

[License]
```

---

### Standard README (For Most Projects)

```markdown
# Project Name

[Badges]

Description paragraph.

## Features

- Feature 1
- Feature 2

## Quick Start

[5-minute getting started]

## Installation

[Install instructions]

## Usage

[Examples]

## Configuration

[Config options]

## Development

[Dev setup]

## Testing

[Test commands]

## License

[License]
```

---

### Comprehensive README (For Complex Projects)

```markdown
# Project Name

[Badges]

[Tagline]

[Screenshot/Demo]

## Table of Contents

[Full TOC]

## Overview

[Detailed description]

## Features

[10+ features]

## Architecture

[Architecture diagram and explanation]

## Getting Started

### Prerequisites
### Installation
### Configuration

## Usage

### Basic Usage
### Advanced Usage
### Examples
### Common Patterns

## API Reference

[Full API docs]

## Development

### Setup
### Project Structure
### Build Process
### Code Style
### Testing

## Deployment

### Docker
### Kubernetes
### Cloud Platforms

## Performance

[Benchmarks, optimization tips]

## Security

[Security considerations]

## Contributing

[Contribution guide]

## Roadmap

[Future plans]

## FAQ

[Common questions]

## License

[License details]

## Acknowledgments

[Credits]
```

---

## Badge Generator

**Generate appropriate badges based on project:**

```markdown
<!-- Build Status -->
![Build](https://img.shields.io/github/actions/workflow/status/user/repo/ci.yml?branch=main)

<!-- Coverage -->
![Coverage](https://img.shields.io/codecov/c/github/user/repo)

<!-- Version -->
<!-- npm -->
![npm](https://img.shields.io/npm/v/package-name)
<!-- PyPI -->
![PyPI](https://img.shields.io/pypi/v/package-name)
<!-- Crates.io -->
![Crates.io](https://img.shields.io/crates/v/package-name)

<!-- License -->
![License](https://img.shields.io/github/license/user/repo)

<!-- Language -->
![Language](https://img.shields.io/github/languages/top/user/repo)

<!-- Downloads -->
![Downloads](https://img.shields.io/npm/dm/package-name)

<!-- Stars -->
![Stars](https://img.shields.io/github/stars/user/repo?style=social)

<!-- PRs Welcome -->
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)

<!-- Maintenance -->
![Maintained](https://img.shields.io/badge/Maintained%3F-yes-green.svg)
```

---

## Best Practices

### 1. Keep It Current
- Update README with every major change
- Keep examples working
- Update version numbers
- Verify links regularly

### 2. Make It Accessible
- Write for beginners
- Explain acronyms
- Include visual aids
- Provide multiple examples

### 3. Be Honest
- Don't oversell features
- Mention limitations
- Acknowledge known issues
- Set realistic expectations

### 4. Show, Don't Tell
- Use code examples over descriptions
- Include screenshots/GIFs
- Provide live demos if possible
- Link to real-world usage

### 5. Make It Scannable
- Use clear headings
- Break into short paragraphs
- Use bullet points
- Add table of contents for long docs

---

## Execution Reminders for Claude

**Before generating README:**
- [ ] Scan entire project structure
- [ ] Read key configuration files
- [ ] Check for existing documentation
- [ ] Identify project type and tech stack
- [ ] Understand architecture/patterns

**During generation:**
- [ ] Use appropriate template
- [ ] Generate working code examples
- [ ] Create accurate badges
- [ ] Document all configuration
- [ ] Include proper attribution

**After generation:**
- [ ] Validate all commands work
- [ ] Check all links
- [ ] Verify code examples
- [ ] Ensure consistent formatting
- [ ] Ask user for feedback

**Common user requests after generation:**
- "Add more examples" → Expand usage section
- "Explain architecture better" → Add diagrams
- "Make it simpler" → Use minimal template
- "Add deployment docs" → Expand deployment section

---

## End of Skill

This skill should be used for generating or updating README.md files for any project. The skill adapts to the project's specific context while maintaining professional documentation standards across all technology stacks.
