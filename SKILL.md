---
name: factor-evaluate
description: 评价单个因子或信号 —— 双 IC（rank + Pearson）+ Sharpe + MDD + 单调性 + 换手 → 归一加权主分。给一个综合分判断"好不好、要不要保留"。触发词：评价因子、给因子打分、factor score、evaluate factor、rank IC、IC_IR、信号好不好。
---

# Factor Evaluate

> 给定一个截面信号 `[date × symbol]`，输出**完整体检报告**：截面有效性 / 风险调整收益 / 稳健性 / 换手成本 → 一个可比的主分。

## 核心规则

1. **七项指标全覆盖**：双 IC + IC_IR + Sharpe + Ann.Return + MDD + Monotonicity + Turnover（详见 `references/metrics.md`）
2. **永远算双 IC**：rank IC 和 Pearson IC 同时算并对照（细则：`references/dual-ic.md`）
3. **归一加权主分**：每个分量先归一到 ~[-2, +2]，再按权重加（公式：`references/primary-score.md`）
4. **项目已有 `primary_score()` 永远优先调用** —— 不要自己另写"近似版"，那叫绕过评估宪法
5. **单看一个数会被刷**：报告必须给六联拆解，不能只给一个 score 数字

## 工作流（标准 6 步）

```
1. 校验信号契约（截面规模 / 均值 / std / NaN 占比）
2. 算 rank IC + Pearson IC 时序
3. 算多头回测：T+1 开盘买 Top 10%、等权、双边 15bp、T+1+H 卖（详见 backtest skill）
4. 算分组单调性（5 或 10 分位）
5. 算换手率
6. 套主分公式 → 输出 ScoreReport
```

## 接口映射

| 本 skill 概念 | 你的项目对应 |
|---|---|
| 主分公式 | `evaluation.md` 锁定的 `primary_score()` |
| 信号 | `[date × symbol]` 浮点 DataFrame |
| 行情 panel | 含 `open` / `close` / `high` / `low` / `volume` |
| HORIZON | 必须等于信号声明的 H |

**调用项目内的 `primary_score()` 时**：传 `(signal, panel, horizon, label_kind)`，得到 `ScoreReport`。**不要自己重写**。

## 按需加载

| 何时读 | 文件 |
|---|---|
| 想知道每项指标算法 | `references/metrics.md` |
| rank vs Pearson IC 对照解读 | `references/dual-ic.md` |
| 主分公式细节 / 调权 | `references/primary-score.md` |
| 输出报告格式 | `references/report-format.md` |
| 反模式（刷 IC / 未来函数 / ...） | `references/anti-patterns.md` |

## QA 检查清单

- [ ] 同时算了 rank IC 和 Pearson IC？
- [ ] forward return 用 `open[T+1+H] / open[T+1] - 1`，不是 close？
- [ ] 涨跌停 / 停牌已剔除？
- [ ] 报告含六联拆解，不只一个数？
- [ ] 调用了项目内的 `primary_score()`，没有另写？

## 跨工具适配

- OpenAI Codex / Assistants → `agents/openai.yaml`
- Cursor → `agents/cursor-rule.mdc`
- 无原生 skill 机制 → `agents/portable-loader.md`

---

## 项目边界（量化研究合规声明）

> 按 QUANTSKILLS 社区规则 §8 声明。

- **数据来源**：本 skill 不附带任何市场数据；使用者需自行准备行情面板（OHLCV / 涨跌停 / 停牌状态等），数据合法性与许可由使用者负责。
- **假设与参数**：默认假设见各 references（T+1 开盘成交、Top 10% 等权、双边 15bp 手续费、A 股涨跌停规则等）。这些是**研究阶段的标准化假设**，不等同于真实交易。
- **已知限制**：
  - 不模拟市场冲击、不模拟集合竞价滑点、不模拟券池融券约束
  - 不处理分红除权除息 / 配股 / 重大事件停牌的复杂情形
  - 默认 pooled cross-section 范式，对单股序列建模、时序模型不适用
- **风险边界**：本 skill 输出的因子分数 / 回测净值 / IC 诊断结果，**仅反映在历史数据 + 假设条件下的统计表现**，不代表未来表现。
- **用途定位**：**仅供量化研究、教育与方法论参考**。不构成任何形式的投资建议、交易信号或获利保证。使用者据此进行实盘交易的全部后果由使用者自负。
