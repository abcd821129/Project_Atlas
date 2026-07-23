# Project Atlas 文档包使用说明

项目定位：全球多资产 AI 扫描、策略选择、硬风控与 MT5 Demo 执行系统。

## 首次使用

把本文件夹内所有文件复制到 Git 仓库根目录，然后在 Codex 中打开该仓库。

第一条指令：

> 请先完整读取 AGENTS.md、PROJECT_SPEC.md、ROADMAP.md、ARCHITECTURE.md、RISK_ENGINE.md、STRATEGY_ENGINE.md 和 TEST_PLAN.md。当前严格只执行 CODEX_PHASE1_START.md 中的 Phase 1。先检查仓库，再给出计划，然后编码、测试、提交；完成后停止。

## 文件优先级

1. `AGENTS.md`：Codex 工作纪律和禁止事项。
2. `PROJECT_SPEC.md`：项目最高级需求。
3. `RISK_ENGINE.md`：硬风控，不允许 AI 或策略绕过。
4. `STRATEGY_ENGINE.md`：策略注册、评分和自主选择规则。
5. `ARCHITECTURE.md`：模块边界和数据流。
6. `ROADMAP.md`：开发阶段。
7. `TEST_PLAN.md`：验收标准。
8. `CODEX_PHASE1_START.md`：当前唯一开发任务。

发生冲突时，按上述优先级执行，并选择更保守、更安全的解释。
