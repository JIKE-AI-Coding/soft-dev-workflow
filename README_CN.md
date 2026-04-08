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

完整使用示例请参阅 `SKILL.md`。
