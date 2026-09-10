---
name: crap-analysis
description: CRAP Score 分析。Use when the user asks for CRAP score, function-level complexity + coverage analysis, or "哪些函数复杂且测试不足最值得优先处理" for Python / JS / TS / TSX projects. 只分析不改代码。
---

找出项目中 **CRAP 高**（复杂度高 × 测试不足）的函数，排序告知用户。只分析，不修改代码、不生成测试、不重构。

## 定义

- **complexity**：圈复杂度（Cyclomatic Complexity），函数/方法级。
- **coverage**：函数级覆盖率，范围 0~1（0% → 0，100% → 1）。
- **CRAP** = `complexity² × (1 - coverage)³ + complexity`
- **HIGH**：CRAP > 30。这是唯一的分级线，不要发明第二档。

## 步骤

1. **识别语言栈**：读仓库文件判断——Python（`.py` / `pyproject.toml` / `pytest.ini`）、Node/TS（`.js` / `.ts` / `.tsx` / `package.json` / `vitest.config.*` / `jest.config.*`）。Monorepo 允许两者并存，各走各的采集配方。
2. **圈定 scope**（按优先级取第一个可用的，先窄后宽）：① 用户指定的代码 ② Git changed files ③ 最近修改的文件 ④ 整个仓库。大仓库禁止默认全仓扫描——先问用户或从 ② 开始。排除 `node_modules`、`dist`、`build`、`coverage`、`.venv`、generated。
3. **采集 complexity + 函数行范围**：每个函数拿到 name、file、start/end line、complexity。配方见下方「Python 配方」「JS/TS 配方」。
4. **采集函数级 coverage**：把 coverage 工具输出的行级/区间级执行数据，按步骤 3 的 start/end line 归并到函数，得到 0~1 的值。没有 coverage 数据时先运行项目已有测试生成（见配方），跑不起来就如实降级——coverage 记为 0 并在报告中注明"未测得"。
5. **计算并排序**：套 CRAP 公式，筛出 HIGH，按 CRAP 降序。
6. **输出报告**（格式见「输出格式」），最后给 Summary 和 Top priorities。

完成标准：scope 内每个函数都有 complexity、coverage、CRAP 三个值，HIGH 名单完整，报告已输出。步骤 3、4 的采集缺一不可——缺 coverage 的"复杂度报告"不是 CRAP 分析。

## Python 配方

- 依赖：`pytest`、`pytest-cov`、`coverage.py`、`radon`。缺哪个只告诉用户需要装什么，未经允许不改依赖文件。
- complexity：`radon cc -s <path>`，输出含函数名、行范围、复杂度。
- coverage：用项目已有 pytest 配置跑 `pytest --cov --cov-report=json`，读 JSON 的行级执行数据归并到函数行范围。
- 优先复用项目现有测试配置；不为运行本 Skill 改 `pyproject.toml` / `pytest.ini`。

## JS/TS 配方

- 测试框架优先复用项目已有的：Vitest > Jest > Node test runner。看 `package.json` scripts 和配置文件确认，不要引入新框架。
- coverage：优先读已生成的 `coverage-final.json`（Istanbul，含 `statementMap` 行级数据）；没有则运行项目现有 coverage 命令生成；V8 coverage 输出按区间归并。
- complexity：ESLint `complexity` 规则（临时以命令行 `--rule 'complexity: ["error", 0]'` 方式跑，不改项目 `.eslintrc`），结合 AST（或 ESLint `estree` 输出）拿到函数行范围；没有 ESLint 的项目用 AST 分析。

## 输出格式

每个 HIGH 函数一段：

```text
CRAP Analysis

HIGH
src/services/payment.ts
PaymentService.process()

Complexity: 18
Coverage: 41%
CRAP: 85.4

Recommendation:
优先补测试，并考虑拆分该函数。
```

Recommendation 只写两件事：补测试 / 拆函数，按 CRAP 构成判断——coverage 极低时先说补测试，complexity 过高（>15）且 coverage 不低时先说拆分。

结尾：

```text
Summary

Functions: 143
High Risk: 8

Top priorities:
1. PaymentService.process
2. AgentRunner.execute
3. BrowserSession.handle
```
