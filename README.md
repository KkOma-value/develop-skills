# develop-skills

**[English](#english) | [中文](#中文)**

Cross-language Agent Skills focused on test **quality** over test **quantity**: locate high-risk functions → generate effective tests → verify the tests actually catch bugs. Supports Python / JavaScript / TypeScript / TSX.

---

## English

### What is this

A set of Agent Skills (for Claude Code and other tools in the [skills](https://github.com/vercel/skills) ecosystem) that combine into a complete test-quality workflow:

```
crap-analysis ──► test-generation ──► mutation-testing
  rank risky fns    fill test gaps     verify effectiveness
```

### Skills

| Skill | Purpose | Trigger phrases |
|---|---|---|
| [`crap-analysis`](crap-analysis/SKILL.md) | Function-level **CRAP score** analysis (`complexity² × (1-coverage)³ + complexity`) to rank the functions most worth fixing — high complexity + poor test coverage | CRAP score, complexity + coverage analysis |
| [`test-generation`](test-generation/SKILL.md) | Analyze existing tests and coverage, identify missing scenarios (branches, boundaries, failure paths), and generate high-quality tests (pytest / unittest, Vitest / Jest / node:test) | write tests, improve coverage |
| [`mutation-testing`](mutation-testing/SKILL.md) | **Mutation testing** with mutmut / StrykerJS — checks whether tests actually catch bugs, and explains the coverage gap behind each surviving mutant | mutation score, test effectiveness |

### Design principles

- **Measure real quality**: the goal of a test is to kill mutants, catch real bugs, and survive refactors — not to chase coverage numbers.
- **Reuse the project's own setup**: test framework, coverage commands, ESLint / pytest configs are all reused. No new frameworks, no dependency-file changes.
- **Analysis-only by default**: writing tests, refactoring, or killing mutants requires an explicit user request.
- **Explicit completion criteria**: e.g. `crap-analysis` isn't done until every function has complexity, coverage, and CRAP values — a "complexity report" without coverage is not a CRAP analysis.

### Installation

```bash
npx skills@latest add KkOma-value/develop-skills --skill=crap-analysis
npx skills@latest add KkOma-value/develop-skills --skill=mutation-testing
npx skills@latest add KkOma-value/develop-skills --skill=test-generation
```

### Recommended workflow

1. **`crap-analysis`**: run against changed files or the whole repo to get a HIGH-risk function list, sorted by CRAP score.
2. **`test-generation`**: fill the gaps for the top functions — review the missing-scenario plan, then generate the minimal necessary test set.
3. **`mutation-testing`**: run mutation testing on the same functions; if the score is low, keep adding tests guided by the survivor reports.

Each skill also works standalone — just ask your AI assistant to "run a CRAP analysis on this file", "add tests for this function", or "mutation test this module".

---

## 中文

### 这是什么

一组可安装到 AI 编程助手中的 Skill(如 Claude Code、支持 [skills](https://github.com/vercel/skills) 生态的工具),组合起来形成一条完整的测试质量工作流:

```
crap-analysis ──► test-generation ──► mutation-testing
   找出高危函数        补缺失的测试         验证测试有效性
```

### Skills

| Skill | 用途 | 关键词 |
|---|---|---|
| [`crap-analysis`](crap-analysis/SKILL.md) | 函数级 **CRAP Score** 分析(`complexity² × (1-coverage)³ + complexity`),找出「复杂度高 + 测试不足」最值得优先处理的函数 | CRAP score、复杂度 + 覆盖率分析 |
| [`test-generation`](test-generation/SKILL.md) | 分析现有测试与覆盖率,按分支/边界/异常路径找出缺失场景,生成高质量测试(pytest / unittest,Vitest / Jest / node:test) | 补测试、提高覆盖率 |
| [`mutation-testing`](mutation-testing/SKILL.md) | 用 mutmut / StrykerJS 做**变异测试**,检查测试是否真能发现 bug,逐个解释 survivor(存活变异体)对应的测试缺口 | mutation score、测试有效性 |

### 设计原则

- **只度量真实质量**:测试的目标是杀 mutant、抓真实 bug、在重构中存活——不是刷覆盖率数字。
- **复用项目现有配置**:测试框架、coverage 命令、ESLint / pytest 配置全部复用,不引入新框架、不改依赖文件。
- **默认只分析不改代码**:补测试 / 重构 / kill mutants 需要用户明确要求。
- **每个 skill 有明确的完成标准**:如 crap-analysis 要求每个函数都有 complexity、coverage、CRAP 三个值,缺 coverage 的报告不算完成。

### 安装

```bash
npx skills@latest add KkOma-value/develop-skills --skill=crap-analysis
npx skills@latest add KkOma-value/develop-skills --skill=mutation-testing
npx skills@latest add KkOma-value/develop-skills --skill=test-generation
```

### 推荐用法

1. **`crap-analysis`**:对改动文件或整个仓库跑一遍,拿到 HIGH 风险函数清单(按 CRAP 降序)。
2. **`test-generation`**:针对 Top 函数补测试,先看缺失场景计划,再生成最小必要的测试集。
3. **`mutation-testing`**:对同一批函数跑变异测试,若 score 不高,按 survivor 报告继续补测试。

三个 skill 也可以独立使用,直接对 AI 助手说「对这个文件跑 CRAP 分析」「给这个函数补测试」「对这个模块做 mutation testing」即可。
