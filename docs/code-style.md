# Code Style

## Philosophy

- Prefer simple, clean, maintainable solutions over clever or complex ones
- Readability and maintainability are primary concerns
- Match the style and formatting of surrounding code, even if it differs from standard style guides
- Consistency within a file is more important than strict adherence to external standards

## Comments

- NEVER remove code comments unless you can prove they are actively false
- All code files should start with a brief 2-line comment explaining what the file does
- Each line should start with "ABOUTME: " to make it easy to grep for
- Comments should be evergreen - avoid referring to temporal context about refactors or recent changes

## Naming

- NEVER name things as 'improved', 'new', 'enhanced', etc.
- Code naming should be evergreen - what is "new" today will be "old" someday

## Implementation Rules

- NEVER implement mock mode for testing or any purpose - always use real data and real APIs
- NEVER throw away old implementation and rewrite without explicit permission
- NEVER make code changes unrelated to the current task - document unrelated issues separately
- NEVER disable functionality instead of fixing the root cause
- NEVER create duplicate files to work around issues - fix the original

## Problem-Solving Approach

- FIX problems, don't work around them
- MAINTAIN code quality and avoid technical debt
- USE proper debugging to find root causes
- AVOID shortcuts that break user experience
