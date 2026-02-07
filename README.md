# 🧠 Agent Skills Repository

A curated collection of **reusable agent skills** designed to be plugged into modern AI-assisted development environments such as **Claude Code**, **Cursor**, **Antigravity**, and similar tools.

This repository acts as a **skill library**: each skill is self-contained, documented, and ready to be copied into your project.

---

## ✨ What are Agent Skills?

**Agent skills** are structured capability definitions that describe *what an AI agent can do*, *how it should behave*, and *how it should reason* when performing a specific task.

They can include:
- Problem-solving strategies
- Coding conventions
- Architectural decision rules
- Review and analysis workflows
- Domain-specific reasoning patterns

Each skill is documented in a human- and AI-readable format to ensure consistent behavior across tools and projects.

---

## 📂 Repository Structure

```text
your-main-repo/
├── claude/                 # For Claude Code
│   └── .claude/
│       └── skills/
│           └── <skill-name>/
│               ├── SKILL.md          # Core skill definition
│               ├── examples/         # (Optional) Sample inputs/outputs
│               └── tools/            # (Optional) Scripts/templates
│
├── cursor/                 # For Cursor IDE
│   └── .cursor/
│       └── skills/
│           └── <skill-name>/         # Identical content
│               └── SKILL.md
│
├── antigravity/           # For Google Antigravity
│   └── .agent/
│       └── skills/
│           └── <skill-name>/         # Identical content
│               └── SKILL.md
│
├── universal/             # Source of truth (single copy)
│   └── skills/
│       └── <skill-name>/
│           └── SKILL.md
└── README.md

