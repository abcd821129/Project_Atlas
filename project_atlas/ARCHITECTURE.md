# 系统架构

## 边界

- 数据层只负责获取、标准化、质量判断。
- 扫描层只负责确定性特征、评分、排名和去相关。
- 分析层负责市场状态和多周期结构。
- 策略层负责产生策略评估，不负责仓位。
- AI层负责比较候选与策略，不直接调用下单。
- 风控层拥有最终否决权。
- 执行层仅执行已经通过校验和风控的指令。
- 复盘层不能修改正在运行的Champion。

## 关键接口

`DataProvider -> NormalizedMarketData`

`Scanner -> ScanCandidate[]`

`RegimeClassifier -> RegimeAssessment`

`Strategy -> StrategyEvaluation`

`StrategyRouter -> StrategySelection | NO_TRADE`

`DecisionValidator -> ValidatedDecision | NO_TRADE`

`RiskGate -> ApprovedOrderIntent | STOP_NEW_ENTRIES`

`ExecutionBridge -> ExecutionReport`

## 故障原则

默认失败关闭：数据源、AI、网络、MT5、时间同步或状态恢复任何关键组件异常时，不开新仓；已有持仓由EA本地安全规则管理。
