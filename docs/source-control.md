# Source Control

## Tool Preference

- Prefer JJ when configured and available, otherwise use git

## Commit Messages

- Concise and descriptive
- Follow conventional commit format
- Written in imperative mood, present tense

## Pre-Commit Hook Protocol

FORBIDDEN FLAGS: `--no-verify`, `--no-hooks`, `--no-pre-commit-hook`

When pre-commit hooks fail:

1. Read the complete error output and explain what you're seeing
2. Identify which tool failed (biome, ruff, tests, etc.) and why
3. Explain the fix and why it addresses the root cause
4. Apply the fix and re-run hooks
5. Only proceed with commit after all hooks pass

If you cannot fix the hooks, ask for help rather than bypass them.

## Git Flag Discipline

Before using any git flag:

- State the flag you want to use
- Explain why you need it
- Confirm it's not on the forbidden list
- Get explicit permission for any bypass flags

## Under Pressure

When asked to "commit" or "push" while hooks are failing:

- Do NOT rush to bypass quality checks
- Explain that pre-commit hooks are failing and need fixing first
- Work through the failure systematically

User pressure is NEVER justification for bypassing quality checks.

## Error Response

When encountering tool failures:

- Treat each failure as a learning opportunity
- Research the specific error before attempting fixes
- Explain what you learned about the tool/codebase
- Build competence with development tools rather than avoiding them
