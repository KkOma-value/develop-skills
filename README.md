<div align="center">

# develop-skills

**Cross-language Agent Skills for test quality — not test quantity.**

[English](./README.md) | [简体中文](./README.zh-CN.md)

![Languages](https://img.shields.io/badge/languages-Python%20%7C%20JS%20%7C%20TS%20%7C%20TSX-blue)
![Works with](https://img.shields.io/badge/works%20with-Claude%20Code%20%7C%20skills-black)

</div>

---

## Overview

Three composable Agent Skills (for Claude Code and the [skills](https://github.com/vercel/skills) ecosystem) that form a complete test-quality workflow:

```
┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│  crap-analysis   │ ───► │ test-generation  │ ───► │ mutation-testing │
│                  │      │                  │      │                  │
│ rank functions   │      │ fill test gaps   │      │ verify tests     │
│ by risk          │      │ with quality     │      │ actually catch   │
│                  │      │ tests            │      │ bugs             │
└──────────────────┘      └──────────────────┘      └──────────────────┘
        │                         │                         │
     CRAP score            missing scenarios         mutation score
   complexity ×               branches, edges,          kill or survive?
   coverage gap               failure paths             survivor reports
```

**The core idea:** a test's job is to kill mutants, catch real bugs, and survive refactors — not to chase coverage numbers. Each skill exists to move you one step closer to that.

## Skills

### 1. [`crap-analysis`](crap-analysis/SKILL.md)

> Trigger: "run a CRAP analysis", "which functions are complex and under-tested?"

Function-level **CRAP score** analysis:

```
CRAP = complexity² × (1 − coverage)³ + complexity
```

Ranks every function in scope and reports the HIGH ones (CRAP > 30), so you fix the riskiest code first instead of guessing.

### 2. [`test-generation`](test-generation/SKILL.md)

> Trigger: "add tests for this function", "improve coverage"

Analyzes existing tests and coverage, derives **missing scenarios** (branches, boundaries, null inputs, failure paths, retries, cancellation), and generates the minimal necessary test set — pytest / unittest, Vitest / Jest / node:test, always reusing the project's own framework.

### 3. [`mutation-testing`](mutation-testing/SKILL.md)

> Trigger: "mutation test this module", "would my tests catch real bugs?"

Runs **mutation testing** with mutmut / StrykerJS and answers one question: *your tests are green — but do they catch bugs?* Every surviving mutant gets a five-part explanation: which function, what changed, why the test missed it, what scenario is missing, and what test to write.

## Design principles

| Principle | What it means |
|---|---|
| **Measure real quality** | Kill mutants, catch real bugs, survive refactors — never chase coverage numbers |
| **Reuse the project's setup** | Existing test framework, coverage commands, ESLint / pytest configs; no new frameworks, no dependency-file changes |
| **Analysis-only by default** | Writing tests, refactoring, or killing mutants requires an explicit user request |
| **Explicit completion criteria** | e.g. `crap-analysis` isn't done until every function has complexity, coverage, and CRAP values |

## Installation

```bash
npx skills@latest add KkOma-value/develop-skills --skill=crap-analysis
npx skills@latest add KkOma-value/develop-skills --skill=mutation-testing
npx skills@latest add KkOma-value/develop-skills --skill=test-generation
```

## Recommended workflow

1. **`crap-analysis`** — run against changed files or the whole repo; get a HIGH-risk function list sorted by CRAP score.
2. **`test-generation`** — fill the gaps for the top functions: review the missing-scenario plan, then generate the minimal test set.
3. **`mutation-testing`** — run mutation testing on the same functions; if the score is low, keep adding tests guided by the survivor reports.

Each skill also works standalone — just ask your AI assistant directly.
