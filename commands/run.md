---
description: Comprehensive code review using specialized agents — orchestrates code, comments, tests, errors, types, and simplify agents sequentially.
scripts:
  sh: scripts/bash/detect-changed-files.sh
  ps: scripts/powershell/detect-changed-files.ps1
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Orchestrate a comprehensive PR review by running specialized review agents against the changed files. Collect all findings, de-duplicate, group by severity, and produce a single consolidated report with actionable next steps.

## Operating Constraints

**STRICTLY READ-ONLY**: Do **not** modify any files. Output a structured analysis report. Offer an optional remediation plan (user must explicitly approve before any follow-up editing commands would be invoked manually).

---

## Step 1: Git Diff Auto-Detection

Identify the changed files to review by running the `detect-changed-files` script.

Run the `detect-changed-files` script with `--json` and parse the JSON output to populate **CHANGED_FILES**. Also capture **MODE** (e.g., branch diff, working tree) for the Review Scope field in the final report.

---

## Step 2: Load Context

### Extension Configuration

Load the review extension config from `.specify/extensions/review/review-config.yml` (if it exists). If not found, fall back to the `defaults` section declared in the extension manifest (`extension.yml`).

Store the loaded values as **CONFIDENCE_THRESHOLD** and **AGENT_TOGGLES** for use in subsequent steps.

### Project Guidelines

Check if project-specific guidelines (typically in `.specify/memory/constitution.md`, `CLAUDE.md`, `.github/copilot-instructions.md` or equivalent) exist:
- If present, record its path as **GUIDELINES_PATH** for passing to agents
- If not present, set GUIDELINES_PATH to empty (agents will skip guideline checks)

---

## Step 3: Determine Applicable Agents

Check `$ARGUMENTS` for user-specified review aspects:
- Parse arguments to see if user requested specific review aspects
- Default: Run all applicable reviews
- **Config filtering**: If the user did not specify agents, exclude any agent where `AGENT_TOGGLES.{agent}` is `false`

### **Available Review Aspects:**
   - **code** - General code quality review — project guideline compliance, bug detection, code quality analysis (`/speckit.review.code`)
   - **comments** - Code comment accuracy verification, documentation completeness assessment, comment rot detection (`/speckit.review.comments`)
   - **tests** - Test coverage quality analysis — behavioral coverage, critical gap identification, test resilience evaluation (`/speckit.review.tests`)
   - **errors** - Error handling review — silent failure detection, catch block analysis, error logging (`/speckit.review.errors`)
   - **types** - Type design analysis — encapsulation, invariant expression, usefulness, and enforcement. Auto-skips for dynamically-typed languages (`/speckit.review.types`)
   - **simplify** - Code simplification suggestions — clarity, unnecessary complexity, redundant abstractions. Advisory only (`/speckit.review.simplify`)
   - **all** - Run all applicable reviews (default)

---

## Step 4: Agent Orchestration

For each applicable agent execute the agent's command as a focused sub-task. Provide it with:
- The **CHANGED_FILES** list
- The **GUIDELINES_PATH** (if present)
- The **CONFIDENCE_THRESHOLD** value

### Collect Results

After each agent completes, extract from its output:
- All findings (severity, file, line, description, recommendation)
- Files analyzed count
- Agent status (success, error, skipped)

### De-Duplicate Findings

Before generating the consolidated report, de-duplicate findings across agents using these rules:

1. **Match criteria**: Two findings are duplicates when they target the **same file** AND the **same line range** (within ±3 lines) AND describe the **same underlying issue** (semantically equivalent, even if worded differently).
2. **Resolution — keep the specialist**: When a duplicate is found, keep the finding from the **more specialized agent** (e.g., prefer `errors` over `code` for error-handling issues, prefer `tests` over `code` for test-coverage issues).
3. **Severity conflict**: If the duplicate findings have different severities, keep the **higher** severity.
4. **Genuine disagreement**: If two agents flag the same location but for **genuinely different concerns** (e.g., `code` flags a naming issue on a line where `errors` flags a missing catch), these are **not** duplicates — include both.
5. **Cross-agent note**: When a finding is de-duplicated, append "(also flagged by `<other-agent>`)" to the kept finding's description for traceability.

---

## Step 5: Generate Consolidated Report

Output the final report in the following Markdown format:

### Report Template

```markdown
# PR Review Report

**Files Analyzed**: <count>
**Review Scope**: <describe how changed files were determined using MODE — e.g., feature branch diff, working directory changes, user-specified files>

## Agent Summary

| Agent | Status | Findings |
|-------|--------|----------|
<one row per agent: name, success/error/skipped, finding count>

## Critical Findings

| # | Agent | File | Line | Finding | Recommendation |
|---|-------|------|------|---------|----------------|
<critical findings rows>

## Important Findings

| # | Agent | File | Line | Finding | Recommendation |
|---|-------|------|------|---------|----------------|
<important findings rows>

## Suggestions

| # | Agent | File | Line | Finding | Recommendation |
|---|-------|------|------|---------|----------------|
<suggestion findings rows>

## Next Actions

   1. Fix critical issues first
   2. Address important issues
   3. Consider suggestions
   4. Re-run review after fixes
```

