# CLAUDE.md

## Tool usage strategy

### Default to action

Implement changes by default instead of just providing suggestions. If the user's intention is unclear, infer the most useful possible operation and continue running, using tools to discover any missing details instead of guessing.

### Use parallel tool calls

If you plan to call multiple tools and there are no dependencies between them, please execute all independent tool calls in parallel. For example, when reading three files, running three tool calls in parallel simultaneously reads three files.

## Git commit specification

### Submit Format Instructions

```
<type>(<scope>): <subject>
```

### Format description

- **type**: commit type (required)
- **scope**: scope of impact (optional)
- **subject**: simple description (required, not exceeding 50 characters)

---

### Type definition

|   type   | description                             | example                                   |
| :------: | :-------------------------------------- | :---------------------------------------- |
|   feat   | new feature                             | feat: add user login                      |
|   fix    | fix bug                                 | fix: login token expire                   |
|   docs   | docs changes                            | docs: update README.md descriptions       |
|  style   | code style (not effect feature)         | style: change indent the number of spaces |
| refactor | refactor (not a new feature or fix bug) | refactor: MyClass structure               |
|   perf   | performance relate                      | perf: optimize list query                 |
|   test   | test relate                             | test: add login form test                 |
|  build   | build system, tools, dependency relate  | build: update package.json dependency     |

---

### Scope range

Customize according to project modules, for example:

- **core** - core module
- **ui** - ui components
- **api** - api
- **auth** - auth module
- **db** - database related
- **config** - configuration related

---

### Subject

- Use English (en)
- Not ending with a period
- Use imperative sentence tone

---

## Decision framework

When there are various methods, decide according to these conditions:

1. **Testability** - can I easily test it?
2. **Readability** - will anyone understand this in 6 months?
3. **Consistency** - does this match project pattern?
4. **Simplicity** - is this the easiest solution?
5. **Reversibility** - how difficult is it to change in the future?

## File management

### Clean temporary files

If you create any temporary new files, scripts, or auxiliary files for iteration, please delete these files for cleaning at the end of the task.

## Accuracy Principle

### Investigate First, Speak Later

Before answering or coding: read relevant files first to ensure decisions are based on actual
codebase state.

### Only Do What's Required

Follow KISS and DRY principles. Only complete tasks explicitly requested. Do not add
unauthorized features or refactoring. Keep code simple.

### Error Handling Standards

All async/await operations must be wrapped in try/catch. Use .catch() for Promise chains.

## Experience & Pitfall Learning Mechanism

After solving complex tricky issues, **rigorously verified** experiences must be documented in a dedicated pitfall guide to avoid repeated mistakes.

### Core Process

1. **Review History**: When encountering tricky issues, first check `.claude/rules/learn-experience.md`.
2. **Record Failures**: During the resolution process, record failed operations, error messages, and context.
3. **Extract Insights**: After successful resolution, analyze the root cause and key solutions.
4. **Update Knowledge Base**: Write verified experiences to `.claude/rules/learn-experience.md`.

### Trigger Conditions for Recording

Recording is mandatory when any of the following conditions are met:

- **Repeated Trial-and-Error**: An issue that required ≥3 attempts to finally resolve.
- **Runtime Pitfalls**: Issues that pass static code checks but only surface during runtime/build time (e.g., async timing issues, memory leaks, environment-specific compatibility).
- **Black-box Config / Magic Parameters**: Configuration items not documented officially, or requiring specific parameter combinations to take effect.
- **Third-party Library Implicit Behaviors**: Undocumented side effects or required special handling logic in dependency packages.

### Write Format Specification (to learn-experience.md)

```
## [Issue Summary / Pitfall Name]

- **Tags**: `#runtime` / `#configuration` / `#third-party-library` / `#environment` / `#tricky-issue`
- **Trigger Context**: [e.g., Windows environment, Vite bundling, specific component unmounting]
- **Symptoms**: [Specific runtime errors, or unexpected behaviors]
- **Root Cause**: [Why did this happen? What is the underlying mechanism?]
- **Verified Solution**:
    // Must be code or configuration snippets that have actually been tested and verified
- **Prevention Recommendations**: [How to avoid in future code, or consolidate into project standards]
```

### Prohibitions

- [Prohibited] Writing before the issue is completely resolved and **verified through actual runtime**.
- [Prohibited] Recording baseless guesses, untested assumptions, or AI hallucinations.
- [Prohibited] Recording vague descriptions; must include specific error messages or code snippets.
- [Prohibited] Piling up these specific experiences in `CLAUDE.md`; must write to `.claude/rules/learn-experience.md`.
