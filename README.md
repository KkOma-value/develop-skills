# develop-skills

跨语言 Agent Skills（Python / JavaScript / TypeScript / TSX）。

## Skills

| Skill | 用途 |
|---|---|
| [`crap-analysis`](crap-analysis/SKILL.md) | 函数级 CRAP Score 分析（复杂度² × (1-coverage)³ + 复杂度），找出"复杂度高 + 测试不足"最值得优先处理的函数 |
| [`mutation-testing`](mutation-testing/SKILL.md) | Mutation Testing，用 mutmut / StrykerJS 检查测试是否真能发现 bug，解释 survivor 测试缺口 |
| [`test-generation`](test-generation/SKILL.md) | 分析现有测试与覆盖率，找出缺失场景并生成高质量测试（pytest / unittest，Vitest / Jest / node:test），补真正能杀 mutant 的测试 |

三者配合使用：先用 `crap-analysis` 定位高危函数，再用 `test-generation` 补上缺失的测试，最后用 `mutation-testing` 验证这些测试是否真的有效。所有 skill 默认只分析不改代码；补测试 / 重构需要用户明确要求。

## 安装

```bash
npx skills@latest add KkOma-value/develop-skills --skill=crap-analysis
npx skills@latest add KkOma-value/develop-skills --skill=mutation-testing
npx skills@latest add KkOma-value/develop-skills --skill=test-generation
```
