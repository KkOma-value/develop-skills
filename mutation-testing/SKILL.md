---
name: mutation-testing
description: Mutation Testing 变异测试。Use when the user asks for mutation testing, mutation score, surviving mutants, 测试有效性/测试缺口分析, or "测试到底能不能发现真实 bug" for Python / JS / TS / TSX projects. 默认只分析不改代码。
---

回答一个问题：**测试看着绿，但真的能抓到 bug 吗？** 注入 mutation，看哪些被测试 kill、哪些活下来。活下来的 **survivor** 就是测试缺口——coverage 正常却测不出错误的位置。默认只分析：不动测试代码、不修业务代码；只有用户明确要求 kill mutants / 补测试时，才修改测试代码。

## 定义

- **baseline**：mutation 之前原有测试必须全绿的状态。全绿以**引擎 dry-run** 为准，普通测试命令的绿不算数（见步骤 3）。
- **survivor**：变异后测试仍然全绿 → 该测试发现不了这个改动。
- **mutation score** = killed ÷ (mutants - timeout)。目标是解释缺口，**不是刷到 100%**——等价 mutant 和低价值 mutant（日志文本、装饰性字符串）永远杀不完，追分是抓错目标。
- **NoCoverage**：scope 测试子集覆盖不到的 mutant，不参与评分。正向利用它：把 scope 缩到改动级。

## 步骤

1. **选 engine**：Python → `mutmut`；JS/TS/TSX → `StrykerJS`（测试框架复用项目现有的，看 `package.json` scripts 和配置确认）。缺 engine 时 `npm i --no-save @stryker-mutator/core @stryker-mutator/vitest-runner` 临时安装——不碰 package.json/lockfile。注意：`--no-save` 装的插件不会被 Stryker 自动发现，配置里显式声明 `"plugins": ["@stryker-mutator/vitest-runner"]`。
   ✅ engine 可运行。
2. **圈定 scope**（优先级取第一个可用）：① 用户指定 ② git changed files ③ 高 CRAP 函数（有 `crap-analysis` 结果就用它）④ 核心业务逻辑 ⑤ 全仓库。前端优先 hooks、utils、stores、services、validation、API client；复杂 UI 渲染跳过。排除 `node_modules`、`dist`、`build`、`coverage`、`generated`。
   ✅ 文件（或行范围）清单 + 覆盖它们的测试子集。
3. **确认 baseline（绿门）**：过两道门——先用项目普通测试命令确认全绿，再让引擎 dry-run 跑一遍并同样全绿。引擎比测试命令严格：Stryker 把 unhandled rejection 判为失败，而 vitest CLI 对其退出 0；jsdom 项目里 fire-and-forget 的网络上报、XHR 噪音都会在普通跑里隐身。dry-run 红了先修环境噪音：优先用 vitest 别名把真实网络上报的模块重定向到静默 stub——只改测试环境，断言与业务代码一字不动。baseline 不绿时，mutant 的生死没有意义。
   ✅ 引擎 dry-run 全绿（0 unhandled）。
4. **执行 mutation**，跑得快靠三件事：
   - `--coverageAnalysis perTest`：每个 mutant 只跑覆盖它的测试，未被覆盖的标 NoCoverage 退出评分。
   - `--mutate` 行范围 `file:startLine-endLine` 缩到改动级；多段逗号分隔。单点行写 `:583-583`，裸 `:583` 会静默匹配不到任何文件——启动日志出现 `did not result in any files` 即是线索。
   - CLI 表达不了的配置写临时文件 `*.tmp.conf.json`（跑完删 `reports/` 与 `.stryker-tmp/`，要复跑补测则保留并注明）。例如 Stryker 限制 vitest 测试子集：临时 vitest 配置（`mergeConfig` 项目现有配置 + `test.include` 子集），stryker 配置里 `"vitest": { "configFile": "./vitest.tmp.ts", "related": false }` 指向它。
   ✅ scope 内每个文件四个数字齐全：killed / survived / timeout / no-coverage。
5. **五问每个 survivor**：① mutation 在哪个函数；② 原代码变成了什么；③ 为什么测试没抓到；④ 缺什么测试场景；⑤ 建议什么测试——命名写成可读的测试用例描述（如 `should_not_retry_when_retry_count_equals_3`）。engine 自带 operator 已覆盖边界类变异（比较符翻转、逻辑符、布尔字面量、算术、返回值、调用删除、常量修改），解释时按类别归因。等价或低价值 mutant 如实标注「不追」，不硬凑测试。
   ✅ 每个 survivor 五问齐全。
6. **输出报告**（格式见下）。

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
NoCoverage: 5

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

Reason 只写测试缺口的事实（哪个分支/边界没有断言踩到）；需要用户定优先级的缺口如实说，默认不写测试代码。
