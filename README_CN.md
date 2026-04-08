# Development Workflow Orchestrator

软件开发工作流编排器，协调从需求分析到部署的完整软件开发生命周期。

## 概述

本技能实现了一个 7 阶段工作流 pipeline，每个阶段有明确定义的角色：

```
[PM] → [Arch] → [Senior-Arch] → [Dev] → [QA] → [DevOps] → [PM]
  ↓        ↓            ↓              ↓         ↓          ↓
!pm-analyze !pm-design !pm-review    !pm-dev   !pm-test  !pm-deploy
                                                                         !pm-uat
```

## 触发命令

| 命令 | 角色 | 用途 |
|:---|:---|:---|
| `!pm-analyze` | [PM] | 需求分析，生成 PRD |
| `!pm-design` | [Arch] | 技术设计，生成 TECH_DESIGN.md + TASK_LIST.md |
| `!pm-review` | [Senior-Arch] | 架构评审与审批 |
| `!pm-dev` | [Dev] | TDD 开发（先后端，后前端）|
| `!pm-test` | [QA] | 集成测试，缺陷跟踪 |
| `!pm-deploy` | [DevOps] | 部署到测试/预发布/生产环境 |
| `!pm-uat` | [PM] | 用户验收测试 |
| `!pm-hotfix` | [Dev] | 生产环境 bug 紧急修复 |

## 工作流阶段

### 阶段 1：需求分析 (!pm-analyze)
- 调用：`product-manager` skill
- 输出：`docs/PRD.md`
- 验证：业务闭合性、数据完整性、状态有效性

### 阶段 2：技术设计 (!pm-design)
- 调用：`solution-architect` skill
- 输出：`docs/TECH_DESIGN.md`、`docs/TASK_LIST.md`
- UI 设计：通过 `/ui-ux-pro-max` 完成

### 阶段 3：架构评审 (!pm-review)
- 调用：`senior-architect` skill
- 决策：Approved 或 Rejected（Rejected 则返回阶段 2）

### 阶段 4：开发 (!pm-dev)
- 后端任务：通过 `backend-developer`
- 前端任务：通过 `frontend-developer`
- 前端 UI：通过 `/frontend-design`
- **顺序**：先后端 → 后前端
- **API 集成**：被前端组件 AND 后端 API 同时阻塞

### 阶段 5：测试 (!pm-test)
- 调用：`qa-engineer` skill
- 输出：`docs/ISSUE_LOG.md`
- Bug 生命周期：Open → Fixed → Verified

### 阶段 6：部署 (!pm-deploy)
- 调用：`devops-engineer` skill
- 顺序：测试环境 → 预发布环境 → 生产环境
- 健康检查失败时自动回滚

### 阶段 7：UAT (!pm-uat)
- 调用：`product-manager` skill
- 验证 PRD 中的 P0 验收标准

## 状态文件

| 文件 | 状态流转 | 负责角色 |
|:---|:---|:---|
| `docs/PRD.md` | Draft → Confirmed | [PM] |
| `docs/TECH_DESIGN.md` | Draft → Ready for Review → Rejected/Approved | [Arch] → [Senior-Arch] |
| `docs/TASK_LIST.md` | Pending → In-Progress → Blocked → To-Test → To-Deploy → Done | [Dev] |
| `docs/ISSUE_LOG.md` | Open → Fixed → Verified | [QA] |

## 核心原则

1. **强制 Skill 调用**：使用 `Skill` 工具委托给角色 skill
2. **顺序执行**：每个阶段必须完成后才能进入下一阶段
3. **后端优先**：后端完成后才能开始前端 API 集成
4. **迭代式开发**：大型项目拆分为多个迭代
5. **PRD 闭合验证**：每个功能必须有完整的开始→结束流程

## 外部依赖

本技能依赖以下外部 Claude Code skills：

| Skill | 用途 | 使用阶段 |
|:---|:---|:---|
| `ui-ux-pro-max` | UI/UX 设计系统、组件规格、设计评审 | 阶段 2：技术设计 |
| `frontend-design` | 前端开发，设计系统、组件实现 | 阶段 4：开发 |

## 使用方式

这是一个 Claude Code skill。当您使用 `!pm-analyze`、`!pm-design` 等触发命令时，会自动调用此 skill。

### 如何开始

1. **从需求开始**：告诉 Claude 您的项目需求或功能请求
2. **使用触发命令**：通过 `!pm-*` 命令调用工作流阶段
3. **遵循 pipeline**：每个阶段的输出作为下一阶段的输入

### 基本工作流示例

```
1. 用户："我需要一个用户登录功能"
2. 用户：!pm-analyze  → [PM] 生成 PRD
3. 用户：!pm-design    → [Arch] 生成 TECH_DESIGN + TASK_LIST
4. 用户：!pm-review   → [Senior-Arch] 评审并批准
5. 用户：!pm-dev      → [Dev] TDD 开发
6. 用户：!pm-test     → [QA] 测试并报告 Bug
7. 用户：!pm-deploy   → [DevOps] 部署
8. 用户：!pm-uat      → [PM] 验收
```

### 触发命令示例

#### 开始需求分析
```
!pm-analyze
```
Claude 将作为 [PM]，根据您的需求生成 `docs/PRD.md`。

#### 开始技术设计
```
!pm-design
```
Claude 将作为 [Arch]，生成 `docs/TECH_DESIGN.md` 和 `docs/TASK_LIST.md`。

#### 评审技术设计
```
!pm-review
```
Claude 将作为 [Senior-Arch] 评审技术设计。如果被拒绝，则返回设计阶段。

#### 开始开发
```
!pm-dev
```
Claude 将作为 [Dev]，读取 TASK_LIST 并以 TDD 方式实现（写测试 → 运行失败 → 写代码 → 测试通过）。

#### 执行测试
```
!pm-test
```
Claude 将作为 [QA]，运行集成测试并将 Bug 记录到 `docs/ISSUE_LOG.md`。

#### 部署到环境
```
!pm-deploy
```
Claude 将作为 [DevOps]，部署到测试环境 → 预发布环境 → 生产环境。

#### 用户验收测试
```
!pm-uat
```
Claude 将作为 [PM]，验证实现是否符合 PRD 验收标准。

### 典型项目流程

```
┌─────────────────────────────────────────────────────────────┐
│  阶段 1：需求分析                                            │
│  输入：用户的需求                                            │
│  输出：docs/PRD.md (Draft → Confirmed)                      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  阶段 2：技术设计                                            │
│  输入：已确认的 PRD                                          │
│  输出：docs/TECH_DESIGN.md, docs/TASK_LIST.md               │
│  外部依赖：ui-ux-pro-max skill 用于 UI 设计                  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  阶段 3：架构评审                                            │
│  输入：TECH_DESIGN.md                                       │
│  输出：Approved / Rejected                                  │
│  如果 Rejected → 返回阶段 2                                 │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  阶段 4：开发                                                │
│  输入：TASK_LIST.md                                         │
│  输出：已实现的代码，测试通过                                │
│  顺序：先后端 → 后前端                                       │
│  外部依赖：frontend-design skill 用于 UI                   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  阶段 5：测试                                                │
│  输入：已部署的代码                                          │
│  输出：测试报告，docs/ISSUE_LOG.md                          │
│  Bug 流程：Open → Fixed → Verified                          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  阶段 6：部署                                                │
│  顺序：测试环境 → 预发布环境 → 生产环境                      │
│  健康检查失败时自动回滚                                     │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  阶段 7：UAT                                                │
│  输入：已部署到预发布环境                                    │
│  输出：PM 验收（Done 或 Rejected）                          │
└─────────────────────────────────────────────────────────────┘
```
