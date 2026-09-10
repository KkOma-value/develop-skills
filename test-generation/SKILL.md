---
name: test-generation
description: Analyze existing tests, coverage reports, and target functions to find missing test scenarios and generate high-quality tests for Python, JavaScript, TypeScript, TSX, Node.js, and frontend business logic. Use when the user asks in any language to 补测试, write tests, generate tests, increase coverage, improve coverage for a function or file, test recently modified code, or add tests for top CRAP-score functions. Reuses the project's existing test framework (pytest/unittest, Vitest/Jest/node:test). Do not use to run mutation testing itself or to write non-test code.
---

Generate tests that verify real behavior and boundary conditions — not coverage numbers. The goal: tests that would kill mutants, catch real bugs, and survive refactors.

## Language & framework detection

Detect the framework from the environment before writing anything; reuse it, never replace it.

- **Python**: pytest first, unittest if that is what exists. Check `pyproject.toml` / `setup.cfg` / existing test imports.
- **JS / TS / TSX / Node**: Vitest first, Jest second, `node:test` if the project uses it. Check `package.json` devDependencies, config files (`vitest.config.*`, `jest.config.*`), existing test imports.
- Confirm the framework by finding at least one existing test file importing it. If the project has no tests at all, default to pytest (Python) or Vitest (JS/TS), and say so.

## Workflow

Execute in order; each step gates the next.

1. **Read the target code** — understand the function's behavior before listing anything.
2. **Read existing tests** — find the conventions (imports, fixtures, mock style, naming) and the existing scenarios, so new tests fill gaps instead of duplicating them.
3. **Get coverage** — prefer existing coverage data (`coverage.xml`, `.coverage`, LCOV, `coverage-summary.json`); run the framework's coverage command only if absent.
4. **Identify uncovered lines and branches** — Lines, Branches, Functions. When branch coverage is available, prioritize it over line coverage: a line executed once may still hide untested branches (e.g. `retry_count < max_retry` vs `== max_retry` vs `> max_retry`).
5. **List missing scenarios** as a plan (format below). Wait for user confirmation only when the user asked for review; otherwise proceed.
6. **Generate the minimal set of tests** for that plan.
7. **Run the target tests**, then related tests, then the full suite if the change touches shared fixtures or helpers.
8. **Re-check coverage** for the target.
9. **Report** in the format below.

## Scenario analysis

Derive scenarios from the function's logic, not from uncovered lines alone. For every function, check the applicable cases:

- normal path
- boundary conditions: empty string, empty array / object, zero, max / min values
- None / null input
- boolean conditions, each `if` / `else` branch, early returns
- exceptions and failure paths: API errors, malformed input, unexpected responses, timeouts, retries reaching their limit, fallbacks
- state changes the function causes

For **agent code** specifically, cover:

- tool call success / failure / timeout / exception
- retry and max-retry exhaustion
- context limit exceeded
- model returns empty content or a malformed tool call
- subagent success / failure / timeout
- cancellation
- state recovery

## Test generation principles

Generate the **minimum necessary** tests. Prioritize tests that:

- cover a new branch
- verify business behavior (input → behavior → output / state change)
- could catch a real bug
- a mutation-testing pass could kill

Skip tests that:

- only execute a line for the number
- duplicate an existing test
- assert only `not None` or only "does not throw"
- are snapshot tests unrelated to business behavior
- are fragile — coupled to incidental implementation details

## Mocking

Mock only external boundaries: HTTP, database, filesystem, model API, tool API, clock, random, third-party services. Test through the public interface:

```
input → behavior → output / state change
```

Avoid asserting on internals (how many times an internal function was called) unless the call count itself is a business requirement. Reuse the project's existing fixtures and mock helpers.

## Permissions

Creating or modifying test files is allowed whenever the request is test generation (补测试 / write tests / generate tests / increase coverage). Never modify business code to make a test pass.

If a test reveals a likely implementation bug, surface it before deciding:

```
Potential implementation bug

Expected: ...
Actual: ...
```

Then keep the test expressing the real business requirement — do not weaken it to match the buggy behavior without the user's decision.

## Verification & report

After every change, run target tests → related tests → full suite when warranted, then re-measure coverage. Report:

```
Test Generation Result

Target:
src/agent/runner.py

Before:
Line Coverage: 42%
Branch Coverage: 28%

Added:
5 tests

Covered scenarios:
- tool success
- tool exception
- timeout
- max retry
- empty model response

After:
Line Coverage: 81%
Branch Coverage: 72%

Tests:
18 passed
0 failed
```

If a plan was requested first, output it before generating tests:

```
Missing test scenarios

AgentRunner.run()

1. tool execution success
2. tool raises exception
3. tool timeout
4. retry reaches max_retry
5. model returns empty response
```

## With CRAP / mutation testing

When the input comes from the CRAP skill, prioritize the highest-CRAP functions. Recommended pipeline:

```
CRAP → top high-risk functions → test generation → coverage / branch coverage up → mutation testing → verify the tests actually kill mutants
```

100% coverage is not the goal. The goal: tests for high-risk code that verify real behavior and boundaries.
