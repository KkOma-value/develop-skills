---
name: mutation-testing
description: Mutation Testing 变异测试。Use when the user asks for mutation testing, mutation score, surviving mutants, 测试有效性/测试缺口分析, or "测试到底能不能发现真实 bug" for Python / JS / TS / TSX projects. 默认只分析不改代码。
---

回答一个问题：**测试看着绿，但真的能抓到 bug 吗？** 方法是注入 mutation，看哪些被测试 kill，哪些活下来（**survivor**）。survivor 就是测试缺口——coverage 正常却测不出错误的位置。默认只分析：不动测试代码、不修业务代码。只有用户明确要求 kill mutants / 补测试时，才修改测试代码。

## 定义

- **baseline**：mutation 之前原有测试必须全绿的状态。
- **survivor**：变异后测试仍然全绿 → 该测试发现不了这个改动。
- **mutation score** = killed ÷ (mutants - timeout)。目标是解释缺口，**不是刷到 100%**——等价 mutant 和低价值 mutant 会永远杀不完，追分是抓错目标。
- 完整跑一轮很慢（每个 mutant 跑一遍测试），所以 scope 先窄后宽（见「Scope」）。

## 步骤

1. **识别技术栈**：Python → engine `mutmut`；JS/TS/TSX → engine `StrykerJS`。测试框架复用项目现有的（pytest / Vitest / Jest / node test runner），看 `package.json` scripts 和配置确认；缺 engine 时只告诉用户装什么，不改依赖文件。
2. **圈定 scope**（优先级取第一个可用的）：① 用户指定的函数/文件 ② Git changed files ③ 高 CRAP Score 函数（有 `crap-analysis` 结果就用它）④ 核心业务逻辑 ⑤ 全仓库。前端项目优先：hooks、utils、stores、services、validation、API client；复杂 UI 渲染代码跳过。排除 `node_modules`、`dist`、`build`、`coverage`、`generated`、`.venv`。
3. **确认 baseline**：先运行原有测试，全绿才继续；红了就停下报告失败清单——baseline 不绿时，mutant 的生死没有意义。engine 配置优先复用项目现有测试配置，用临时命令行参数指向测试命令，不改项目配置文件。
4. **执行 mutation**：跑选定的 engine（配方见下）。engine 默认 operator 已覆盖边界类 mutation（比较符 `>`↔`>=`、`==`↔`!=`、逻辑符 `and`↔`or`、布尔字面量、返回值、算术运算符、null/optional 判断、调用删除、常量修改），不需要逐条检查，但解释 survivor 时按这些类别归因。
5. **分析 survivor**：每个 survivor 回答五问——mutation 在哪个函数、原代码变成了什么、为什么测试没抓到、缺什么测试场景、建议加什么测试。
6. **输出报告**（格式见下），每个 survivor 给 `Suggested Test`，命名写成可读的测试用例描述。

完成标准：baseline 已确认绿、scope 内每个文件的 mutants/killed/survived/timeout 四个数字齐全、每个 survivor 都有五问答案、报告已输出。

## 配方

**Python（mutmut）**：`mutmut run` 之前用 `--paths-to-mutate`（或 CLI 参数）限定 scope；测试命令走项目现有 pytest 配置；结果看 `mutmut results` 和 `mutmut show <id>` 拿到 diff。

**JS/TS（StrykerJS）**：项目已有 `stryker.conf.json` 就复用；没有则用命令行 `--mutate` / `--testRunner` / `--coverage analysis` 限定 scope，不落盘配置文件；报告读 mutation report JSON 的 survivor 列表（含 originalLine、mutatedCode）。

## 输出格式

先给统计：

```text
Mutation Testing

Target:
src/services/payment.ts

Engine:
StrykerJS

Mutants: 52
Killed: 43
Survived: 7
Timeout: 2

Mutation Score: 86%
```

再逐个展开 survivor：

```text
Survivor #1

Function:
PaymentService.retry()

Mutation:

retryCount > 3
→
retryCount >= 3

Reason:
没有测试 retryCount === 3。

Suggested Test:
should_not_retry_when_retry_count_equals_3
```

Reason 只写测试缺口的事实（哪个分支/边界没有断言踩到），Suggested Test 写一个具体场景名；缺的部分如实说"建议由用户确认优先级"，默认不写测试代码。
