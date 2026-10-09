<div align="center">

# develop-skills

**跨语言 Agent Skills,聚焦测试质量,而非测试数量。**

[English](./README.md) | [简体中文](./README.zh-CN.md)

![Languages](https://img.shields.io/badge/languages-Python%20%7C%20JS%20%7C%20TS%20%7C%20TSX-blue)
![Works with](https://img.shields.io/badge/works%20with-Claude%20Code%20%7C%20skills-black)

</div>

---

## 概述

三个可组合的 Agent Skill(适用于 Claude Code 及 [skills](https://github.com/vercel/skills) 生态),构成一条完整的测试质量工作流:

```
┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│  crap-analysis   │ ───► │ test-generation  │ ───► │ mutation-testing │
│                  │      │                  │      │                  │
│ 按风险给函数排序    │      │ 补上高质量的       │      │ 验证测试           │
│                  │      │ 缺失测试          │      │ 能否真抓到 bug     │
└──────────────────┘      └──────────────────┘      └──────────────────┘
        │                         │                         │
     CRAP 分数                缺失场景                   mutation 分数
   复杂度 ×                  分支 / 边界 /               杀死还是存活?
   覆盖率缺口                 异常路径                    survivor 报告
```

**核心理念:** 测试的职责是杀死 mutant、抓住真实 bug、在重构中存活——而不是刷覆盖率数字。每个 skill 都是为把你往这个目标推近一步而存在。

## Skills

### 1. [`crap-analysis`](crap-analysis/SKILL.md)

> 触发说法:「对这个文件跑 CRAP 分析」「哪些函数复杂且测试不足最值得优先处理」

函数级 **CRAP Score** 分析:

```
CRAP = complexity² × (1 − coverage)³ + complexity
```

对 scope 内每个函数排序,报告 HIGH 风险名单(CRAP > 30),让你优先处理风险最高的代码,而不是凭感觉猜。

### 2. [`test-generation`](test-generation/SKILL.md)

> 触发说法:「给这个函数补测试」「提高覆盖率」

分析现有测试与覆盖率,推导**缺失场景**(分支、边界、null 输入、异常路径、重试、取消),生成最小必要的测试集——pytest / unittest,Vitest / Jest / node:test,始终复用项目自己的框架。

### 3. [`mutation-testing`](mutation-testing/SKILL.md)

> 触发说法:「对这个模块做 mutation testing」「我的测试能不能发现真实 bug」

用 mutmut / StrykerJS 运行**变异测试**,回答一个问题:*测试看着绿,但真的能抓到 bug 吗?* 每个存活变异体(survivor)都给出五问解释:在哪个函数、原代码变成了什么、为什么测试没抓到、缺什么场景、建议写什么测试。

## 设计原则

| 原则 | 含义 |
|---|---|
| **度量真实质量** | 杀 mutant、抓真实 bug、在重构中存活——绝不刷覆盖率数字 |
| **复用项目现有配置** | 现有测试框架、coverage 命令、ESLint / pytest 配置;不引入新框架、不改依赖文件 |
| **默认只分析不改代码** | 补测试、重构、kill mutants 需要用户明确要求 |
| **明确的完成标准** | 如 crap-analysis 要求每个函数都有 complexity、coverage、CRAP 三个值才算完成 |

## 安装

```bash
npx skills@latest add KkOma-value/develop-skills --skill=crap-analysis
npx skills@latest add KkOma-value/develop-skills --skill=mutation-testing
npx skills@latest add KkOma-value/develop-skills --skill=test-generation
```

## 推荐用法

1. **`crap-analysis`** —— 对改动文件或整个仓库跑一遍,拿到按 CRAP 降序排列的 HIGH 风险函数清单。
2. **`test-generation`** —— 针对 Top 函数补测试:先看缺失场景计划,再生成最小必要的测试集。
3. **`mutation-testing`** —— 对同一批函数跑变异测试;若分数不高,按 survivor 报告继续补测试。

三个 skill 也可以独立使用,直接对 AI 助手说即可。
