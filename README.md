# ai-review-gates

Reusable GitHub Actions workflows that convert "review evidence" into a deterministic **pass/fail** check.

## What it does

This repo provides a reusable workflow that checks whether a PR has satisfied three categories:

- **AI Review** (e.g. Gemini Code Assist)
- **Static Analysis / Quality** (e.g. SonarCloud Quality Gate, CodeQL)
- **Security / Dependency** (e.g. Snyk)

It passes if:
- all required categories are satisfied **OR**
- the PR is **docs-only** (configurable whitelist)

The output is deterministic and machine-checkable, so an orchestrator (like `.clawdbot`) can gate “ready for human review”.

## Usage (in another repo)

Create `.github/workflows/ai-review-gate.yml`:

```yaml
name: AI Review Gate

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

jobs:
  gate:
    uses: commotionlabs/ai-review-gates/.github/workflows/pr-category-gate.yml@main
    with:
      docs_only_globs: '**/*.md,documentation/**,docs/**'
      require_categories: 'ai,static,security'
      ai_check_name_patterns: '(?i)gemini'
      static_check_name_patterns: '(?i)sonarcloud.*quality gate,(?i)codeql'
      security_check_name_patterns: '(?i)snyk'
```

## Notes

- This workflow currently detects categories using **successful check-run names** on the PR head SHA.
- Most GitHub Apps that integrate properly will produce check runs; if an app only comments, it won’t satisfy the gate.
- Adjust the regex patterns to match the exact check names your apps produce.
