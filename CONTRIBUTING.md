# Contributing

Thanks for contributing to `ios-marketing-capture`.

This repository is intentionally small, but changes still affect real agent behavior. Most contributions here change either:

- `README.md`: how humans discover and install the skill
- `SKILL.md`: how coding agents actually behave
- `templates/`: the reference code patterns agents adapt for each project

## What Makes a Good Contribution

- Fixes a real workflow problem for people capturing iOS marketing screenshots
- Documents a new gotcha discovered on a real project
- Improves support for different navigation patterns or app architectures
- Makes the skill easier to use across agents and environments
- Keeps the skill opinionated, but still project-agnostic

## Scope Guidelines

Good fit:

- New gotchas from real capture failures
- Better navigation pattern coverage (e.g. custom coordinators, UIKit hosts)
- Improved element rendering reliability
- Better shell script portability
- Clearer installation or usage docs

Usually not a fit:

- Project-specific assumptions that don't generalize
- Hardcoded app names, bundle IDs, or view types in templates
- Large framework additions outside the skill's purpose
- Changes that make the skill significantly more verbose without improving outcomes

## Before Opening a PR

1. Check open PRs to avoid overlapping work.
2. Read `README.md`, `SKILL.md`, and both templates.
3. Keep user-facing docs and skill behavior aligned when applicable.

## Testing Changes

There is no automated test suite in this repository, so use a manual smoke-test checklist.

### For README-only changes

- Confirm installation instructions still make sense
- Confirm example prompts and commands are copy-pasteable

### For `SKILL.md` changes

Validate the skill against at least one realistic scenario:

1. Start with a SwiftUI app that has no marketing capture system
2. Ask an agent to capture marketing screenshots
3. Confirm the skill:
   - asks the required discovery questions first
   - explores the codebase before writing code
   - produces a working `MarketingCapture.swift` adapted to the project
   - generates a shell script that runs end-to-end
   - handles at least one gotcha correctly (e.g. sheet dismiss, seed-once)

### For template changes

- Verify the template compiles when adapted to a real project
- Confirm placeholder comments are clear about what to replace

### Strongly Recommended

Include in your PR description:

- the prompt you used to test the skill
- what behavior changed
- what stayed intentionally unchanged

## Authoring Guidelines

- Prefer compact, high-signal instructions over long prose
- Keep examples realistic and production-oriented
- Avoid duplicating large blocks between `README.md` and `SKILL.md` unless the duplication helps different audiences
- Gotchas must include: what happens, why, and the fix

## Pull Request Checklist

Before submitting, verify:

- the change solves a concrete problem
- the wording is clear for both humans and agents
- README and SKILL instructions do not contradict each other
- the PR description explains why the change is useful
