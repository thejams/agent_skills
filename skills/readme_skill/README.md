# 📄 README Generator Skill

## 🎯 Overview
Analyzes any codebase and generates professional README.md files with badges, setup instructions, real examples, architecture diagrams, and deployment guides. Automatically adapts to detected tech stack (Node.js, Go, Python, Rust, etc.).

**When to use**: "generate README", "document this project", "improve README"

---

## 🚀 Quick Start
```bash
# Copy to your project
cp -r readme-generator/ .claude/skills/   # Claude Code
cp -r readme-generator/ .cursor/skills/   # Cursor
cp -r readme-generator/ .agent/skills/    # Antigravity

# Invoke:
# "/readme-generator" or "/readme-generator mode=enhance"
```

---
## 📋 Features
- Auto-detects: Tech stack, project type (lib/app/CLI/microservice)

- Preserves: Existing content from current READMEs

- Templates: 30+ predefined sections + real badges

- Real examples: Working code, no placeholders

- Multi-stack: Node.js, Go, Python, Rust, Java, Docker, K8s
---

## 📊 Workflow (6 Phases)

- Phase 0: Project Discovery → Detects stack + asks user
- Phase 1: Tech Stack Analysis → package.json, go.mod, etc.
- Phase 2: Architecture → MVC, microservices, CLI patterns
- Phase 3: README Template → Complete structure with badges
- Phase 4: Content Generation → Real examples + config
- Phase 5: Polish → Validates links, tests commands
