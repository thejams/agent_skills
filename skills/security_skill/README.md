# 🛡️ Security & Code Quality Analysis Skill

## 🎯 Overview
Comprehensive security audit + code quality analysis: detects thundering herd, SQL injection, memory leaks, race conditions. Generates severity-ranked reports with specific fixes.

**When to use**: "security audit", "check vulnerabilities", "code review"

## 🚀 Quick Start
```bash
cp -r security-audit/ .claude/skills/
# Test: "/security-audit scope=changed" (for PR reviews)
```
---

## 📋 Features
Critical patterns: Self-DDoS, SQL injection, hardcoded secrets

Cross-stack: Frontend+backend mismatches (retry vs rate-limit)

False positive prevention: Verifies safeguards before reporting

Dependency scan: npm audit, pip-audit integration

Merge recommendation: BLOCK/WARNING/SAFE

---
## 🚨 Critical Patterns Detected

| Pattern            | Severity    | Fix Time |
| ------------------ | ----------- | -------- |
| No retry backoff   | 🔴 CRITICAL | 15min    |
| SQL concatenation  | 🔴 CRITICAL | 30min    |
| Hardcoded secrets  | 🔴 CRITICAL | 5min     |
| Missing rate limit | 🔴 CRITICAL | 1h       |
| Memory leaks       | 🟠 HIGH     | 2h       |

---

## 📊 Report Structure

- 🔴 CRITICAL: 3 (BLOCK DEPLOYMENT)
- 🟠 HIGH: 2 (REVIEW REQUIRED)

- ❌ CURRENT CODE: [vulnerable snippet]
- ✅ REQUIRED FIX: [working solution]
- 📋 IMPLEMENTATION CHECKLIST
- 🎚️ Confidence: High

---

## 💡 Example Report


- 🔍 Found: WebSocket retry without jitter/backoff
- 📁 websocket.js:45
- ⚠️ RISK: Self-DDoS with 1000+ clients
- ✅ FIX:
```js
// Add exponential backoff + jitter
const delay = (attempt) => 
  Math.min(1000 * 2 ** attempt + Math.random() * 100, 30000);
```


## 🎯 Perfect For
- **Pre-deploy audits** of gRPC services
- **PR code reviews**
- **Production incident analysis**
- **Dependency vulnerability scans**
