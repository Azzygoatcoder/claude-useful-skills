---
name: reaudit
description: Use when the user types /reaudit or asks to verify that security fixes have been applied, re-audit a codebase after fixes, or check whether previous audit findings are resolved. Shortcut for code-security-audit Phase 4 (re-audit verification). Triggers on "/reaudit", "re-audit", "verify fixes", "再审计", "检查修复", "确认修复".
---

# /reaudit — Verify Security Fixes

Shortcut slash command. Immediately load and follow `code-security-audit` Phase 4:

1. Read the previous audit report at `docs/SECURITY_AUDIT.md`
2. For every finding, read the flagged file at the flagged location
3. Classify each as: Fixed, Not Fixed, Partially Fixed, or Mitigated
4. Check for regressions (new issues introduced by fixes)
5. Add a Re-Audit section at the top of the audit report

Read `code-security-audit/SKILL.md` Phase 4 for the detailed workflow.
