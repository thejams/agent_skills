# Security & Code Quality Analysis Skill

## Skill Activation Triggers

Use this skill when the user's request matches ANY of these patterns:
- "analyze security"
- "check for vulnerabilities"
- "security audit"
- "code review"
- "find security issues"
- "check code quality"
- "look for bugs"
- "security scan"
- "find race conditions"
- "check for memory leaks"
- "review this code"
- References security, vulnerabilities, or code quality

## Skill Objectives

This skill performs comprehensive security and code quality analysis across any technology stack. It:
1. Automatically detects the project's tech stack
2. Applies context-appropriate security checks
3. Identifies critical patterns (DDoS risks, injection flaws, memory leaks)
4. Provides actionable fixes with code examples
5. Generates severity-ranked reports

## Critical Framework Rules

### Rule 1: Always Start with Context Discovery
**Before analyzing ANY code, Claude MUST:**
1. Check for configuration files (`.claude-security.md`, `CLAUDE.md`, `.security-config.md`)
2. Scan project structure to detect tech stack
3. Identify critical paths and areas of concern
4. Read dependency files to check for known vulnerabilities

### Rule 2: Adaptive Analysis Based on Context
**Different code requires different checks:**
- Frontend code → XSS, memory leaks, client-side secrets
- Backend code → SQL injection, auth bypass, rate limiting
- Infrastructure → Exposed secrets, privilege escalation
- Database → Missing indexes, unbounded queries

### Rule 3: Cross-Stack Analysis
**Always consider interactions:**
- Frontend retry logic + Backend rate limiting
- Client-side validation + Server-side validation
- Authentication flow end-to-end

### Rule 4: Actionable Output Only
**Every finding MUST include:**
- Specific vulnerable code snippet
- Clear explanation of the risk
- Working fix with code example
- Severity level with justification

### Rule 5: Minimize False Positives
**Before reporting, verify:**
- Is this actually exploitable in context?
- Are there safeguards elsewhere?
- Is this test/mock code?
- Could this be intentional?

## Workflow: Security Analysis Process

### Phase 0: Context Discovery (MANDATORY FIRST STEP)

**Claude must execute these steps:**

1. **Look for configuration files** (read in this order):
   ```bash
   # Project-specific security config
   view .claude-security.md
   view .security-config.md
   view CLAUDE.md
   
   # Code ownership and critical paths
   view .github/CODEOWNERS
   view CODEOWNERS
   ```

2. **Detect tech stack** by examining:
   ```bash
   # Check for package managers
   view package.json       # Node.js/JavaScript
   view requirements.txt   # Python
   view go.mod            # Go
   view Cargo.toml        # Rust
   view pom.xml           # Java
   view Gemfile           # Ruby
   
   # Check for frameworks
   ls -la | grep -E "\.jsx?|\.tsx?|\.vue|\.py|\.go|\.rs|\.java"
   ```

3. **Identify critical paths:**
   - Authentication/authorization code
   - API endpoints
   - Database queries
   - Connection/retry logic
   - Payment processing
   - User input handling
   - File operations

4. **Ask user for scope** (if not specified):
   ```
   I'll perform a security analysis. What would you like me to focus on?
   
   1. Full codebase audit (comprehensive)
   2. Specific files/directories (targeted)
   3. Recent changes only (git diff)
   4. Specific vulnerability types (e.g., just SQL injection)
   
   Select one or specify custom scope.
   ```

**Wait for user confirmation before proceeding.**

---

### Phase 1: Tech Stack Detection

**Automatically identify technology context:**

**Detection logic:**

```
IF files contain (.jsx, .tsx, React imports, DOM APIs):
  → Frontend (JavaScript/TypeScript/React)
  → Focus: XSS, memory leaks, client secrets, retry logic

IF files contain (Express, Fastify, Koa, HTTP server):
  → Backend (Node.js)
  → Focus: SQL injection, auth, rate limiting, input validation

IF files contain (Flask, Django, FastAPI, SQLAlchemy):
  → Backend (Python)
  → Focus: SQL injection, CSRF, auth, deserialization

IF files contain (http package, database/sql, gorm, gorilla/mux):
  → Backend (Go)
  → Focus: Race conditions, goroutine leaks, SQL injection

IF files contain (Dockerfile, docker-compose, k8s yaml):
  → Infrastructure
  → Focus: Secrets in configs, privilege escalation, resource limits

IF files contain (SQL, migrations, schema):
  → Database
  → Focus: Missing indexes, unbounded queries, injection
```

**Show user:**
```
🔍 Tech Stack Detected:
- Frontend: React + TypeScript
- Backend: Node.js + Express
- Database: PostgreSQL
- Infrastructure: Docker

Focus areas:
- XSS vulnerabilities in React components
- SQL injection in database queries
- Rate limiting on API endpoints
- Retry logic in WebSocket connections

Proceed with analysis?
```

---

### Phase 2: Critical Pattern Detection

**Scan for high-severity patterns in priority order:**

#### Priority 1: CRITICAL Patterns (Block Deployment)

**Pattern 1: Thundering Herd / Self-DDoS**

**Detection criteria:**
```javascript
// Look for retry/reconnect patterns:
- setTimeout/setInterval in retry logic
- while loops with retries
- Reconnection after disconnect
- No jitter/randomization
- No exponential backoff
- No maximum attempts
- Fixed delays
```

**Scan with:**
```bash
rg "retry|reconnect|attempt" -A 10 | rg -v "jitter|backoff|maxRetries"
```

**If found, check for safeguards:**
- [ ] Exponential backoff present?
- [ ] Jitter/randomization present?
- [ ] Maximum retry limit?
- [ ] Circuit breaker?
- [ ] Cleanup on success/failure?

**If ANY safeguard missing → Report as CRITICAL**

---

**Pattern 2: Unhandled Promise Rejections**

**Detection criteria:**
```javascript
// Look for:
- new Promise with reject()
- Promises in loops/retries
- No .catch() handler
- No try-catch around await
- Multiple reject calls possible
```

**Scan with:**
```bash
rg "new Promise" -A 20 | rg -v "\.catch|try.*catch"
```

---

**Pattern 3: Memory Leaks**

**Detection criteria:**
```javascript
// Look for:
- addEventListener/on() without removeEventListener/off()
- setInterval/setTimeout not cleared
- Closures capturing large objects
- Event listeners in retry loops
- WebSocket/SSE without cleanup
```

**Scan with:**
```bash
# Event listeners without cleanup
rg "addEventListener|\.on\(" -A 10 | rg -v "removeEventListener|\.off\("

# Timers without cleanup
rg "setTimeout|setInterval" -A 10 | rg -v "clearTimeout|clearInterval"
```

---

**Pattern 4: Hardcoded Secrets**

**Detection criteria:**
```regex
(password|secret|api[-_]?key|private[-_]?key|token|auth[-_]?token|access[-_]?token|client[_-]?secret|bearer)\s*[:=]\s*['"][^'"]{8,}['"]
```

**Scan with:**
```bash
rg -i "password|api_?key|secret|token" --type-not test -g '!*.md' -g '!*.example'
```

**Exceptions:**
- Example files (*.example, *.sample)
- Test files with dummy data
- Documentation
- Environment variable names (not values)

---

**Pattern 5: SQL Injection**

**Detection criteria:**
```javascript
// Look for:
- String concatenation with SQL keywords
- Template literals in queries
- No parameterized queries
- Dynamic table/column names from user input
```

**Language-specific patterns:**

**JavaScript/TypeScript:**
```bash
rg "query.*\+|execute.*[\+\`]|SELECT.*\$\{|INSERT.*\$\{" -t js -t ts
```

**Python:**
```bash
rg "execute.*%|query.*\.format|f\".*SELECT|f\".*INSERT" -t py
```

**Go:**
```bash
rg "Query.*\+|Exec.*\+|fmt\.Sprintf.*SELECT" -t go
```

---

**Pattern 6: Missing Authentication/Authorization**

**Detection criteria:**
- API endpoints without auth middleware
- Direct database access without permission checks
- Admin functions accessible to regular users
- JWT validation disabled
- Session management issues

**Scan strategy:**
1. Find all route definitions
2. Check each for auth middleware
3. Verify permission checks in handlers
4. Check for privilege escalation paths

---

**Pattern 7: Missing Rate Limiting**

**Detection criteria:**
- Public API endpoints without rate limiting
- Authentication endpoints without brute force protection
- Resource-intensive operations unlimited
- WebSocket connections unthrottled

**Context matters:**
- Public endpoints → CRITICAL
- Authenticated → HIGH  
- Admin-only → MEDIUM

---

**Pattern 8: Resource Exhaustion**

**Detection criteria:**
```javascript
// Look for:
- Unbounded arrays/collections
- No pagination (SELECT without LIMIT)
- File uploads without size limits
- Infinite loops possible
- No timeouts on operations
- Memory allocation without bounds
```

**Database queries:**
```bash
rg "SELECT.*FROM" | rg -v "LIMIT|OFFSET|TOP"
```

**Array operations:**
```bash
rg "\.push\(|\.concat\(|\.map\(" -A 5 | rg -v "\.slice\(|\.splice\(|length.*<"
```

---

#### Priority 2: HIGH Severity Patterns

**Quick scan for:**
- Missing input validation
- Improper error handling (silent failures)
- Race conditions in async code
- Missing CSRF protection
- XSS vulnerabilities
- Weak cryptography (MD5, SHA1 for passwords)
- Missing audit logs
- Insecure dependencies

---

#### Priority 3: MEDIUM Severity Patterns

**Quick scan for:**
- Code duplication
- High complexity (>10 cyclomatic)
- Missing logging
- Poor naming
- Type safety issues (`any` in TypeScript)
- Deprecated APIs
- Missing indexes
- Inefficient algorithms

---

### Phase 3: Cross-Stack Analysis

**Check for issues spanning multiple layers:**

**Frontend + Backend Coordination:**

1. **Retry logic mismatch:**
   ```
   Frontend: Unlimited retries with 5s delay
   Backend: No rate limiting
   → Result: Self-DDoS vulnerability
   ```

2. **Validation mismatch:**
   ```
   Frontend: Input validation
   Backend: No validation
   → Result: Validation bypass via API
   ```

3. **Authentication mismatch:**
   ```
   Frontend: CSRF token generated
   Backend: Token not verified
   → Result: CSRF vulnerability
   ```

**Claude should:**
1. Identify related frontend/backend files
2. Check both sides for complementary protections
3. Report mismatches as vulnerabilities

---

### Phase 4: Dependency Vulnerability Scan

**Check for known vulnerabilities:**

```bash
# Node.js
npm audit --json

# Python
pip-audit --format json

# Go
go list -json -m all | nancy sleuth

# Rust  
cargo audit --json

# Check for outdated packages
npm outdated
pip list --outdated
```

**Report format:**
```
📦 Dependency Vulnerabilities:

🔴 CRITICAL: lodash@4.17.15
- CVE-2020-8203: Prototype Pollution
- Affected: All versions < 4.17.19
- Fix: npm install lodash@4.17.21
```

---

### Phase 5: Generate Report

**Structure findings by severity:**

#### Report Template

```markdown
# 🛡️ Security Analysis Report

**Project:** [Auto-detected name]
**Tech Stack:** [Detected technologies]
**Scanned:** [Date/Time]
**Files Analyzed:** [Count]
**Lines of Code:** [Count]

---

## 📊 Executive Summary

- 🔴 Critical Issues: X (BLOCK DEPLOYMENT)
- 🟠 High Issues: X (REQUIRE REVIEW)
- 🟡 Medium Issues: X (SHOULD FIX)
- 🟢 Low Issues: X (SUGGESTIONS)

**Merge Recommendation:** ❌ BLOCK / ⚠️ REVIEW REQUIRED / ✅ SAFE TO MERGE

**Top 3 Risks:**
1. [Most critical issue]
2. [Second critical issue]
3. [Third critical issue]

---

## 🔴 CRITICAL ISSUES (Must Fix)

[For each critical issue, use detailed template below]

---

## 🟠 HIGH PRIORITY ISSUES

[Brief format for high-priority issues]

---

## 🟡 MEDIUM PRIORITY ISSUES

[Summary format]

---

## 🟢 LOW PRIORITY SUGGESTIONS

[List format]

---

## 📈 Metrics

- Security Score: X/100
- Code Quality Score: X/100
- Technical Debt: [Low/Medium/High]
- Most Vulnerable Areas:
  1. [Path]
  2. [Path]
  3. [Path]

---

## ✅ Remediation Roadmap

### Immediate (Critical):
1. [Action item with file:line]
2. [Action item with file:line]

### Short-term (High):
1. [Action item]
2. [Action item]

### Medium-term (Medium):
1. [Action item]
2. [Action item]

### Long-term (Low):
1. [Action item]
2. [Action item]

---

## 📚 Resources

- [Link to OWASP guidelines]
- [Link to framework security best practices]
- [Link to relevant CVE database]
```

---

## Issue Report Template

**For each issue, use this format:**

```markdown
🔴 CRITICAL: [Issue Name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📁 File: [file path]:[line number]
🔍 Pattern: [Detection pattern name]
🎯 Category: [Security/Performance/Reliability/Code Quality]

❌ CURRENT CODE:
```[language]
[Show problematic code with 5 lines before/after for context]
```

⚠️  RISK:
[Clear explanation of what can go wrong]

💥 POTENTIAL IMPACT:
- Normal Operation: [What breaks in normal use]
- Under Load: [What happens at scale]
- Security: [How it can be exploited]
- User Experience: [How users are affected]

✅ REQUIRED FIX:
```[language]
[Show corrected code with explanatory comments]
```

📋 IMPLEMENTATION CHECKLIST:
- [ ] [Specific step 1]
- [ ] [Specific step 2]
- [ ] [Specific step 3]
- [ ] Add tests to verify fix
- [ ] Update documentation

📚 REFERENCES:
- [OWASP guideline if applicable]
- [Framework documentation]
- [Related CVE if applicable]
- [Best practice resource]

🎚️ CONFIDENCE: [High/Medium/Low]
❓ FALSE POSITIVE LIKELIHOOD: [Low/Medium/High]
[If not Low, explain conditions where this might be acceptable]

ESTIMATED FIX TIME: [5min/30min/2hours/1day]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Severity Classification Rules

### 🔴 CRITICAL (Block Deployment)

**Conditions:**
- Can cause production outage
- Self-inflicted DDoS possible
- Data breach possible
- Authentication bypass
- SQL injection in production code
- Hardcoded production secrets
- No input validation on public endpoints

**In critical paths (auth, payments, connections):**
- Upgrade HIGH → CRITICAL
- Upgrade MEDIUM → HIGH

**With past incidents:**
- Upgrade severity by 1 level

---

### 🟠 HIGH (Require Review)

**Conditions:**
- Security vulnerability (not immediately exploitable)
- Performance degradation under load
- Data corruption possible
- Missing CSRF protection
- XSS vulnerability
- Weak cryptography
- Missing audit logs
- Race conditions

---

### 🟡 MEDIUM (Should Fix)

**Conditions:**
- Code quality issues
- Minor performance issues
- Missing validation (non-critical data)
- High complexity
- Missing logging
- Poor error handling
- Deprecated API usage
- Missing indexes

---

### 🟢 LOW (Suggestions)

**Conditions:**
- Code style issues
- Minor optimizations
- Missing documentation
- Unused code
- Console logs in production

---

## Adaptive Severity Table

| Pattern | Base Severity | Critical Path | Past Incident | Public API | Final |
|---------|--------------|---------------|---------------|------------|-------|
| No retry backoff | 🟠 HIGH | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 |
| SQL injection | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 |
| Missing validation | 🟡 MEDIUM | 🟠 HIGH | 🟠 HIGH | 🟠 HIGH | 🟠 |
| Hardcoded secret | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 |
| Memory leak | 🟠 HIGH | 🔴 CRITICAL | 🔴 CRITICAL | 🟠 HIGH | 🔴 |
| No rate limiting | 🟠 HIGH | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 |
| Missing auth | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 CRITICAL | 🔴 |
| Code duplication | 🟢 LOW | 🟡 MEDIUM | 🟢 LOW | 🟢 LOW | 🟡 |

---

## False Positive Prevention

**Before reporting an issue, verify:**

### 1. Context Check
```
Question: Is this actually exploitable in this context?

Check:
- Is user input actually reaching this code?
- Are there protections in earlier layers?
- Is this a controlled/internal-only path?
- Is this dead/unreachable code?
```

### 2. Safeguards Check
```
Question: Are there safeguards elsewhere?

Check:
- Frontend validation + backend validation?
- Rate limiting at load balancer level?
- Web Application Firewall (WAF) rules?
- Database-level constraints?
```

### 3. Test Code Check
```
Question: Is this test/mock code?

Indicators:
- In __tests__ directory
- File ends with .test.js/.spec.js
- Contains test framework imports (jest, mocha, pytest)
- Mock/stub/fake in variable names

Action: Lower severity or skip
```

### 4. Intentional Pattern Check
```
Question: Could this be intentional?

Examples:
- Admin debug endpoints (should still be flagged but with context)
- Intentionally unbounded queries (with good reason)
- Test/development configurations

Action: Report but note it might be intentional
```

### 5. Uncertainty Handling

**If Claude is uncertain:**

```markdown
❓ POTENTIAL ISSUE: [Pattern Name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📁 File: [path]:[line]
🎯 Confidence: LOW

🔍 PATTERN OBSERVED:
[Describe what was detected]

❓ QUESTIONS:
1. Is this actually a security concern in this context?
2. Are there safeguards I'm not seeing?
3. Is this intentional for valid reasons?

💭 ANALYSIS:
[Claude's reasoning about why this might or might not be an issue]

⚖️  SIMILAR TO:
[Reference to known patterns]

✅ IF THIS IS A PROBLEM:
[Suggested fix]

RECOMMENDATION: Manual review recommended
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Configuration File Support

### Primary: `.claude-security.md`

**If this file exists, read it for:**

```markdown
# Project Security Configuration

## Tech Stack
[Override auto-detection if needed]

## Critical Paths
[Paths requiring extra scrutiny]
- /src/auth/** - Authentication
- /src/api/payments/** - Payment processing

## Known Issue Areas
[Historical context]
- /src/connections/websocket.js - Had DDoS incident 2024-01

## Custom Rules
[Project-specific requirements]
1. All API endpoints MUST have rate limiting
2. All database queries MUST use ORM
3. No client-side secrets ever

## Exceptions
[Skip these]
- /src/legacy/** - Scheduled for refactor
- /vendor/** - Third-party
- **/*.test.js - Test files

## Severity Overrides
[Adjust based on context]
- SQL injection in /admin/** → Keep CRITICAL
- Missing validation in /internal/** → Downgrade to MEDIUM

## Alert Targets
- 🔴 Critical → @security-team, block merge
- 🟠 High → @code-owners
- 🟡 Medium → PR comments
- 🟢 Low → Optional review
```

### Fallback: `CLAUDE.md`

**Read for project context:**
- Project description
- Architecture notes
- Critical components
- Technology decisions

### Fallback: `CODEOWNERS`

**Read for:**
- Who owns which paths
- Critical/sensitive areas (often heavily owned)
- Team structure

---

## Analysis Scope Options

### Option 1: Full Codebase Audit

```bash
# Scan entire project
find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.go" \) \
  -not -path "*/node_modules/*" \
  -not -path "*/vendor/*" \
  -not -path "*/.git/*"
```

**Use when:**
- Initial security assessment
- Pre-production audit
- Compliance requirements
- After major refactor

---

### Option 2: Changed Files Only (PR Review)

```bash
# Get files changed in PR
git diff --name-only origin/main...HEAD

# Get detailed changes
git diff origin/main...HEAD
```

**Use when:**
- Pull request review
- Incremental changes
- CI/CD integration
- Quick validation

---

### Option 3: Specific Paths

```bash
# User specifies paths
analyze /src/auth/**
analyze /src/api/payments/**
```

**Use when:**
- Targeted review
- Specific concern
- Refactoring specific area
- Following up on incident

---

### Option 4: Specific Vulnerability Types

```bash
# User requests specific checks
check for SQL injection
find memory leaks
audit retry logic
```

**Use when:**
- Known vulnerability type
- Specialized audit
- Learning/training
- Following security advisory

---

## Scan Commands Reference

**JavaScript/TypeScript:**
```bash
# Find retry/reconnect logic
rg "(retry|reconnect|attempt)" -t js -t ts -A 10

# Find unhandled promises
rg "new Promise" -t js -t ts -A 20 | rg -v "\.catch"

# Find event listeners without cleanup
rg "(addEventListener|\.on\()" -t js -t ts -A 10 | rg -v "(removeEventListener|\.off\()"

# Find SQL concatenation
rg "(query|execute).*(\+|`)" -t js -t ts

# Find hardcoded secrets
rg -i "(password|api_?key|secret|token)\s*[:=]\s*['\"][^'\"]{8,}" -t js -t ts
```

**Python:**
```bash
# Find SQL injection
rg "(execute|query).*(%|\.format|f\")" -t py

# Find hardcoded secrets
rg -i "(password|api_?key|secret|token)\s*=\s*['\"][^'\"]{8,}" -t py

# Find missing input validation
rg "request\.(GET|POST|args|form)" -t py -A 5 | rg -v "validate"
```

**Go:**
```bash
# Find SQL injection
rg "(Query|Exec).*(\+|Sprintf)" -t go

# Find missing error handling
rg "err\s*:?=" -t go -A 2 | rg -v "if.*err"

# Find goroutine leaks
rg "go func" -t go -A 10 | rg -v "defer|cancel|Done"
```

**Database:**
```bash
# Find queries without LIMIT
rg "SELECT.*FROM" | rg -v "(LIMIT|TOP|OFFSET)"

# Find missing indexes
rg "WHERE.*=" -t sql -B 2 | rg -v "INDEX"
```

---

## Learning Mode

**When encountering unfamiliar patterns:**

```markdown
🎓 LEARNING: New Pattern Detected
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📁 File: [path]:[line]

🔍 OBSERVED PATTERN:
```[language]
[code snippet]
```

❓ ANALYSIS QUESTIONS:
1. What is this code trying to do?
2. What are the failure modes?
3. Could this be exploited?
4. What happens at scale (1000+ users)?
5. Are there safer alternatives?

💭 MY REASONING:
[Claude's analysis based on known patterns]

⚖️  SIMILAR TO:
- [Known pattern 1]: Similar because...
- [Known pattern 2]: Different because...

🎯 PRELIMINARY ASSESSMENT:
Severity: [CRITICAL/HIGH/MEDIUM/LOW/UNKNOWN]
Confidence: [Low]
Recommendation: [Flag for manual review / Safe / Needs more context]

📚 WOULD BENEFIT FROM:
- Context about [specific aspect]
- Information about [specific safeguard]
- Confirmation that [specific assumption]

Please provide feedback to improve detection accuracy.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Core Principles for Analysis

### 1. Prevention Over Detection
**Catch issues before they reach production**
- Integrate into PR review process
- Block merges on critical issues
- Provide fixes, not just complaints

### 2. Context Awareness
**Same code can be safe or critical depending on context**
- Test code vs production code
- Internal endpoints vs public APIs
- Admin functions vs user functions
- Critical paths vs auxiliary features

### 3. Scale Awareness
**What works for 10 users may fail for 10,000**
- Test assumptions at scale
- Consider concurrent users
- Think about edge cases
- Simulate failure modes

### 4. Cross-Stack Thinking
**Security is only as strong as the weakest layer**
- Check frontend AND backend
- Verify end-to-end flows
- Look for bypasses across layers
- Consider the whole system

### 5. Actionable Feedback
**Always provide specific, working fixes**
- Show the vulnerable code
- Explain the risk clearly
- Provide working fixed code
- Include implementation steps

### 6. Efficiency
**Minimize false positives**
- Verify before alerting
- Check for safeguards
- Consider context
- Ask when uncertain

### 7. Clarity
**Explain WHY, not just WHAT**
- Explain the attack vector
- Show the impact
- Describe the failure mode
- Educate the developer

### 8. Prioritization
**Focus on production-impacting issues first**
- Critical → High → Medium → Low
- Security → Stability → Performance → Style
- Public → Authenticated → Internal

### 9. Empathy
**Be constructive, not judgmental**
- Focus on the code, not the developer
- Provide learning opportunities
- Acknowledge trade-offs
- Suggest improvements

### 10. Continuous Learning
**Improve detection based on feedback**
- Track false positives
- Learn from incidents
- Adapt to new patterns
- Update rules based on results

---

## Execution Reminders for Claude

**Before starting analysis:**
- [ ] Read configuration files first
- [ ] Auto-detect tech stack
- [ ] Identify critical paths
- [ ] Confirm scope with user
- [ ] Set severity thresholds

**During analysis:**
- [ ] Scan in priority order (Critical → High → Medium → Low)
- [ ] Check for cross-stack issues
- [ ] Verify findings (minimize false positives)
- [ ] Gather context (surrounding code)
- [ ] Generate fixes for every issue

**When reporting:**
- [ ] Structure by severity
- [ ] Provide executive summary
- [ ] Include specific code fixes
- [ ] Give implementation checklists
- [ ] Provide remediation roadmap
- [ ] Include confidence scores

**After analysis:**
- [ ] Provide merge recommendation
- [ ] Suggest preventive measures
- [ ] Offer to help implement fixes
- [ ] Ask for feedback on accuracy

---

## Example Interaction Flow

**User:** "Analyze the security of my WebSocket connection code"

**Claude Response:**

```
I'll analyze your WebSocket connection code for security and reliability issues.

First, let me gather context:

🔍 Checking for configuration files...
✅ Found .claude-security.md
✅ Found CLAUDE.md

📋 Configuration Summary:
- Critical Path: /src/connections/** (past DDoS incident)
- Tech Stack: Node.js + Express + Socket.io
- Custom Rule: All retry logic must have exponential backoff + jitter

🎯 Analysis Scope:
Focusing on:
- Connection/retry logic
- Memory leaks
- Self-DDoS vulnerabilities
- Error handling

Scanning files...
```

[Performs analysis following all phases]

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# 🛡️ Security Analysis Results

## 📊 Summary
- 🔴 Critical: 2 (BLOCK DEPLOYMENT)
- 🟠 High: 1
- 🟡 Medium: 3
- 🟢 Low: 2

❌ MERGE RECOMMENDATION: BLOCK - Critical issues must be fixed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Detailed findings with fixes...]

Would you like me to:
1. Show how to implement the fixes?
2. Analyze other parts of the codebase?
3. Generate tests for the fixed code?
```

---

## End of Skill

This skill should be used for all security and code quality analysis tasks across any technology stack. The skill adapts to the project's specific context while maintaining consistent, thorough analysis standards.
