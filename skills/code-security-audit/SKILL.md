---
name: code-security-audit
description: Use when the user wants to audit a codebase for security vulnerabilities, perform a security review, do penetration testing, run a 代码审计 or 安全审查, check for security issues, or verify that security fixes have been applied. Triggers on phrases like "audit this repo", "security review", "find vulnerabilities", "安全审计", "代码审计", "再审计", "verify fixes", "安全扫描".
---

# Code Security Audit

## Overview

Systematic security audit of any codebase using parallel domain exploration. Launch independent review agents across three security domains simultaneously, then synthesize findings into a structured audit report with severity ratings and concrete remediation steps.

**Core principle:** Coverage through parallelization. Three focused agents catch more than one broad agent — each domain has different grep patterns, different mental models, and different blind spots.

## Slash Commands

| Command | Action |
|---------|--------|
| `/audit` | Full security audit — explore → verify → report (Phase 1–3) |
| `/reaudit` | Verify previous audit fixes were applied (Phase 4) |
| `/code-security-audit` | Same as `/audit` (canonical name) |

## When to Use

Trigger when the user asks to:
- Audit a repository or codebase for security issues
- Perform a security review or vulnerability assessment
- Check if security fixes were properly applied (re-audit)
- Find vulnerabilities before a release or deployment

Do NOT use for:
- Reviewing a single PR diff (use `/security-review` or manual review)
- Checking one specific function for bugs (use systematic-debugging)
- General code review for style/architecture (use `/code-review`)

## Audit Workflow

### Phase 1 — Parallel Exploration

Launch **3 Explore agents simultaneously** (single message, parallel tool calls). Each agent covers one security domain.

**Agent A: Secrets & Credentials**
Prompt template:
```
Explore the codebase at <path> for security issues related to secrets, credentials, and sensitive data handling:
1. Hardcoded API keys, tokens, passwords in source files
2. .env files, config files that might contain secrets
3. How credentials are stored, accessed, and written
4. Any logging or error messages that might leak sensitive data
5. Check .gitignore — are sensitive file patterns properly excluded?
6. Check git history for accidentally committed secrets
Report findings with specific file paths, line numbers, and code snippets.
```

**Agent B: Input Validation & Injection**
Prompt template:
```
Explore the codebase at <path> for security issues related to input validation, injection, and unsafe data handling:
1. Command injection: shell=True, os.system, subprocess with user input in command strings
2. SQL injection if database queries exist
3. Path traversal: user input used in file paths without validation
4. Insecure deserialization: pickle, yaml.load, marshal
5. XSS: innerHTML, document.write, unsanitized user data in HTML context
6. SSRF: user-controlled URLs in HTTP clients without allowlist validation
7. Unsafe eval/exec usage
8. Template injection (SSTI)
Report findings with specific file paths, line numbers, and code snippets.
```

**Agent C: Auth, Cryptography & Dependencies**
Prompt template:
```
Explore the codebase at <path> for security issues related to:
1. Authentication and authorization: missing auth checks, weak auth mechanisms
2. Cryptography: weak algorithms (MD5, SHA1, DES), hardcoded keys, improper TLS
3. Session management: insecure cookies, missing HttpOnly/Secure/SameSite
4. CSRF protections on state-changing endpoints
5. Dependency security: unpinned versions, known-vulnerable packages, missing lockfile
6. File permissions: credential files with weak permissions
7. Temp file handling: predictable names, missing cleanup
Report findings with specific file paths, line numbers, and code snippets.
```

**Customize the prompts** based on the codebase language and framework. Add language-specific patterns (e.g., for Python add `os.popen`, for JS add `eval`, for Go add `text/template` without escaping).

### Phase 2 — Deep-Dive Verification

After receiving all three agent reports:

1. **Read the flagged files** yourself. Agents provide summaries but you must verify each finding is real.
2. **Filter false positives**. Not everything an agent flags is exploitable. Check context: is user input actually reachable? Is the vulnerable function guarded?
3. **Assign severity** to each confirmed finding:

| Severity | Criteria |
|----------|----------|
| **Critical** | Remote unauthenticated access to secrets, arbitrary code execution, complete system compromise |
| **High** | Authenticated access to others' data, privilege escalation, injection with direct impact |
| **Medium** | Information disclosure, missing hardening, defense-in-depth gaps |
| **Low** | Best-practice violations with minimal direct risk, cosmetic issues |

4. **Merge overlapping findings** from different agents into single entries.

### Phase 3 — Report Compilation

Write the audit report to an agreed location. Use this exact structure:

```
# [Project Name] Security Audit Report

**Date:** [date]
**Scope:** [what was reviewed]
**Methodology:** Three parallel reviews covering (1) secrets & credentials, (2) input validation & injection, (3) authentication, cryptography & dependencies

## Executive Summary
One paragraph: total findings by severity, most critical issues, overall risk posture.

## Risk Distribution
Table: | Severity | Count | Key Findings |

## Critical Findings
One subsection per finding:
- **Finding:** title
- **Severity:** Critical
- **File:** path (line numbers)
- **Description:** what and why it matters
- **Vulnerable code:** fenced code block
- **Remediation:** concrete fix with corrected code

## High Findings
[Same format]

## Medium Findings
[Same format]

## Low Findings
[Same format]

## Cross-Cutting Recommendations
Themes spanning multiple findings.

## Remediation Priority Matrix
Table: | # | Finding | Effort | Impact | Priority (P1-P4) |

## Appendix
Commit hash, verification guidance per finding.
```

### Phase 4 — Re-Audit (When Fixes Are Applied)

When the user says fixes are applied and wants verification:

1. **Read each fixed file** at the flagged locations
2. **Classify every finding**: Fixed, Not Fixed, Partially Fixed, Mitigated
3. **Check for regressions** — did the fix introduce new issues?
4. **Add a Re-Audit section** above the original audit in the report document

Re-audit section structure:
```
## Re-Audit ([date])

### Fixed (N of total)
Table: | # | Finding | Fix Verified At |

### Not Fixed (N of total)
Table: | # | Finding | Status/reason |

### Partially Fixed (N of total)
Table: | # | Finding | What's done vs. remaining |

### New Findings
Any issues introduced by the fixes.

### Verdict
One paragraph summary.
```

## Quick Reference: Vulnerability Categories

See `references/vulnerability-patterns.md` for grep patterns and remediation templates for each category. Read that file before starting Phase 2 to know what to look for.

Key categories covered:
- Path Traversal, CSRF/CORS, Command Injection, XSS, SSRF
- SQL Injection, Insecure Deserialization, SSTI, Eval Injection
- File Permissions, Plaintext Credentials, Environment Leakage
- Dependency Management, Subprocess Injection
- Auth Bypass, Session Weaknesses, Cryptographic Weaknesses
- Temp File Handling, Log Forging, Race Conditions

## Common Mistakes

| Mistake | Correction |
|---------|------------|
| Running a single broad agent | Three focused agents find different things. Always use all three. |
| Accepting agent findings without reading code | Agents summarize; you must verify each finding's reality and severity. |
| Skipping Phase 4 because "fixes look right" | Always verify every finding against current code. Fixes can be incomplete or introduce new issues. |
| Writing report before verifying | Phase 3 (write) only after Phase 2 (verify). |
| Using severity labels inconsistently | Follow the criteria table above. Critical = remote + unauthenticated + secret access or RCE. |
| Not including concrete remediations | Every finding must have a fix code snippet, not just "fix this." |
| Forgetting to check .gitignore and git history | Secrets accidentally committed are still in history even if removed from HEAD. |

## After the Audit

- **Report location:** Place the report at `docs/SECURITY_AUDIT.md` by default. Add it to `.gitignore` so it stays local.
- **Follow-up:** Offer to fix the highest-priority findings (P1 items).
- **Pattern collection:** After the audit, review `references/vulnerability-patterns.md`. Add any new patterns you discovered. This is how the skill evolves.
