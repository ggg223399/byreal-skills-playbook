# Byreal Trading Strategy Playbook

> Byreal CLI + Agent Wallet + Perp 全策略图谱
>
> Date: 2026-03-06 | Version: 2.0

---

## 图例

| 标签 | 含义 |
|------|------|
| **风险** | `C` 保守 / `M` 中性 / `A` 激进 |
| **复杂度** | `*` 单步 / `**` 多步 / `***` 多腿组合 |
| **适用行情** | `T` 趋势 / `R` 震荡 / `V` 高波动 / `Q` 低波动 / `*` 全天候 |
| **时间框架** | `min` 分钟级 / `H` 小时级 / `D` 日级 / `W` 周级 |
| **最低资金** | 建议起步资金（USDC） |

---

## 一、基础设施积木

| 积木 | 能力 |
|------|------|
| **Spot (Byreal DEX)** | 池子、Token、仓位、Swap、CopyFarmer、Launchpad、Kline、Incentive |
| **Perp (Hyperliquid)** | 开/平仓、限价单、止盈止损、持仓、资金费率、清算价、PnL、Vault/Copy、跨链划转 |
| **Agent Wallet** | 自动签名、风控策略、跨链资金管理（Solana <> Hyperliquid） |
| **NL 配置** | Claude 解析自然语言 -> 结构化策略参数 |
| **执行引擎** | Cron 轮询 / 事件驱动、风控校验、技能插件化 |
| **分发渠道** | CLI / MCP / TG Bot / Web Dashboard |

---

## 二、策略总览

> 120 条策略，按**底层资产 x 风险层级**组织。每条标注可操作参数。

### 2A. LP 策略（Byreal CLMM） — 26 条

#### 范围管理 — 流动性的"仓位管理"

| # | 策略 | 触发条件 / 入场 | 出场 / 风控 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|----------------|------------|------|--------|------|------|------|
| 1 | **Auto Range Rebalance** | 价格偏离当前 range X% | 撤出 -> 重新部署新 range | C | ** | * | H-D | 1k |
| 2 | **Volatility-Adaptive Range** | ATR/布林带宽度变化触发 | 高波动自动加宽，低波动收窄 | C | ** | * | H | 1k |
| 3 | **Narrow Range Sniper** | 预期大波动事件前（如 CPI、merge） | 事件结束后即撤 | A | * | V | min-H | 5k |
| 4 | **Multi-Range Split** | 入场即部署多层 range | 核心窄区 60% + 两侧宽区各 20% | M | ** | R | D | 3k |
| 5 | **Range Trailing** | 价格偏离 range 中心 > Y% | 中心跟随价格移动，trailing 逻辑 | M | ** | T | H | 2k |
| 6 | **Mean-Reversion Range** | 价格偏离均值 > Z% 时不跟 | 价格回归时吃满手续费；IL > 阈值止损 | M | ** | R | D | 2k |
| 7 | **Breakout Range** | 突破关键支撑/阻力 | 在新价格区间快速部署，跌回原区间撤出 | A | ** | T | H | 3k |
| 8 | **Single-Sided LP** | 只提供单边，等价格进入 range | 价格穿越 range 后自动转换 | M | * | T | D | 1k |

#### 收益优化 — 最大化 LP 回报

| # | 策略 | 触发条件 / 入场 | 出场 / 风控 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|----------------|------------|------|--------|------|------|------|
| 9 | **Auto Compound** | Fee 累积 > 阈值自动复投 | IL > 阈值暂停复投 | C | * | * | D | 500 |
| 10 | **Incentive Harvester** | Incentive 奖励 > Gas 成本 | 领取 -> 换目标 Token -> 复投/提取 | C | * | * | D-W | 500 |
| 11 | **Fee Tier Optimizer** | 监控不同 Fee Tier 实际 APR | APR 差异 > 阈值时迁移 | C | ** | * | W | 2k |
| 12 | **Pool Rotation** | 多池 APR 排序，前 N 池 | APR 跌出前 N 时轮换 | M | ** | * | D-W | 3k |
| 13 | **APR Target** | 设定目标年化（如 30%） | 自动调节 range 宽窄逼近目标 fee | M | ** | * | D | 2k |
| 14 | **Incentive + Fee Max** | Fee APR + Incentive APR 综合最优 | 最优 range/池子组合 | M | ** | * | D | 2k |
| 15 | **JIT Liquidity** | 检测大额 swap mempool 信号 | 瞬间提供流动性，赚 fee 后即撤 | A | *** | * | min | 10k |

#### 生命周期管理

| # | 策略 | 触发条件 / 入场 | 出场 / 风控 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|----------------|------------|------|--------|------|------|------|
| 16 | **LP DCA（渐进入场）** | 分 N 批入场，每批间隔 T | 全部入场后转其他管理策略 | C | * | * | D-W | 1k |
| 17 | **LP Gradual Exit** | 分批撤出，避免滑点 | 按时间/条件分 N 批退出 | C | * | * | D | 1k |
| 18 | **IL Monitor + Auto Exit** | IL > 阈值（如 -5%） | 自动撤出并平仓 | C | * | V | H | 500 |
| 19 | **LP Position Aging** | 新仓位激进窄 range | 存续 > T 天后自动转宽 range 保守模式 | M | ** | * | W | 2k |
| 20 | **LP Migration** | 池子激励变化 / 池子升级 | 旧池撤出 -> 新池部署 | C | ** | * | W | 1k |

#### CopyFarmer 增强

| # | 策略 | 触发条件 / 入场 | 出场 / 风控 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|----------------|------------|------|--------|------|------|------|
| 21 | **Smart Copy** | Top Farmer 开仓 | 复制仓位 + 自动 Perp 对冲 delta | M | *** | * | D | 3k |
| 22 | **Selective Copy** | 只跟特定池/Token 对操作 | Farmer 撤出时跟撤 | M | ** | * | D | 1k |
| 23 | **Copy with Range Mod** | 复制 Farmer range 中心 | 自定义宽度（比 Farmer 宽/窄 X%） | M | ** | * | D | 1k |
| 24 | **Multi-Farmer Blend** | 多个 Farmer 加权组合 | 任一 Farmer 撤出触发 rebalance | M | ** | * | D | 5k |
| 25 | **Farmer Alpha Tracking** | Farmer 异常操作（突然撤出/新开） | 预警通知 / 自动跟随 | C | * | * | H | 1k |
| 26 | **Anti-Copy** | 只跟历史胜率 > X% 的操作 | 胜率下降时自动停跟 | M | ** | * | D | 2k |

---

### 2B. Spot 策略 — 30 条

#### 执行优化 — 大单拆分与 MEV 防护

| # | 策略 | 触发条件 / 入场 | 出场 / 风控 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|----------------|------------|------|--------|------|------|------|
| 27 | **TWAP** | 大单 -> 按时间均匀拆分 | 全部执行完毕 | C | * | * | H | 5k |
| 28 | **VWAP** | 大单 -> 按历史成交量分布拆分 | 全部执行完毕 | C | * | * | H | 5k |
| 29 | **Iceberg Order** | 只显示部分量，分批暗盘执行 | 全部成交 | C | * | * | H | 10k |
| 30 | **MEV-Aware Execution** | 检测三明治攻击风险 | 动态调整 Jito tip + 执行时机 | C | ** | * | min | 1k |
| 31 | **Smart Router** | 跨多池拆单路由 | 最小化总滑点 | C | ** | * | min | 1k |

#### 趋势 / 动量 / 均值回归

| # | 策略 | 入场信号 | 出场信号 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|---------|---------|------|--------|------|------|------|
| 32 | **Grid Trading** | 部署价格区间 [P_low, P_high]，等分 N 格 | 价格跌破/突破区间止损 | M | ** | R | H-D | 2k |
| 33 | **Smart DCA** | 低于 MA 加倍买，高于 MA 减量 | 达到目标持仓量 / 手动止盈 | C | * | * | D-W | 500 |
| 34 | **DCA Sell** | 设定卖出计划，分批止盈 | 全部卖出 / 价格反转暂停 | C | * | T | D-W | 500 |
| 35 | **Momentum Breakout** | 突破关键价位 / MA 金叉 | Trailing stop X% 回撤 | A | ** | T | H-D | 1k |
| 36 | **Mean Reversion** | 偏离均值 > X% | 回归均值 / 偏离加倍止损 | M | ** | R | H-D | 2k |
| 37 | **Trailing Buy** | 价格从高点回落 X% | 买入后设 stop loss | M | * | * | H | 1k |
| 38 | **Trailing Sell** | 价格从低点反弹 X% | 卖出后观望 | M | * | * | H | 1k |
| 39 | **Bollinger Band** | 触及下轨买 / 上轨卖 | 回归中轨平仓 / 突破 2 倍宽止损 | M | ** | R | H-D | 1k |
| 40 | **RSI Strategy** | RSI < 30 买 / RSI > 70 卖 | RSI 回归中性区 | M | * | R | H-D | 1k |
| 41 | **MA Crossover** | 快线上穿慢线做多 / 下穿做空 | 反向交叉 / trailing stop | M | * | T | D | 1k |

#### 组合 / 指数 / 板块轮动

| # | 策略 | 入场 | 出场 / 再平衡 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|------|--------------|------|--------|------|------|------|
| 42 | **Index Tracking** | 自定义 Token 篮子，按权重买入 | 偏离权重 > X% 触发 rebalance | C | ** | * | W | 3k |
| 43 | **Basket DCA** | 一笔钱按比例定投多个 Token | 周期性执行 | C | * | * | W | 500 |
| 44 | **Sector Rotation** | 强势板块（Meme/DeFi/Infra）轮入 | 板块动量衰减时轮出 | A | ** | T | W | 3k |
| 45 | **Token Graduation Sniper** | Pump.fun 毕业 -> Byreal 首笔买入 | 利润 > X% 止盈 / -Y% 止损 | A | ** | * | min-H | 1k |
| 46 | **Accumulation Bot** | 设定目标持仓量，智能分批吸筹 | 达到目标量 | C | * | * | D-W | 1k |
| 47 | **Auto Rebalance** | 组合偏离目标权重 > X% | 自动买卖恢复目标权重 | C | ** | * | W | 2k |

#### Launchpad

| # | 策略 | 入场 | 出场 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|------|------|------|--------|------|------|------|
| 48 | **Launchpad Sniper** | 新项目上线 -> 自动参与 Auction + 首买 | 预设止盈/止损 | A | ** | * | min | 1k |
| 49 | **Launch -> LP** | 打新获得 Token -> 自动部署 LP | IL > 阈值撤出 | A | ** | * | H | 2k |
| 50 | **Launch + Hedge** | 打新 + 同时 Perp 空单锁定利润 | 目标利润达到后双平 | A | *** | * | H-D | 3k |
| 51 | **Whitelist Monitor** | 检测白名单资格 -> 自动参与 | 按打新策略出场 | M | * | * | D | 1k |

#### 链上信号

| # | 策略 | 信号源 | 执行 / 出场 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|--------|------------|------|--------|------|------|------|
| 52 | **Whale Tracker** | 大户链上转账/交易 | 跟单买入 / 大户卖出时跟卖 | A | ** | * | H | 2k |
| 53 | **New Pool Alert + Auto Enter** | 新池创建事件 | 自动分析并首笔入场 / 预设止损 | A | ** | * | min | 1k |
| 54 | **TVL Drain Detector** | 池子 TVL 急降 > X% | 自动撤出所有相关仓位 | C | * | * | min | 500 |
| 55 | **Hot Token Momentum** | Hot token 列表 + 链上动量 | 趋势衰减时退出 | A | ** | T | H-D | 1k |
| 56 | **Multiplier Chaser** | 高倍数 Token 信号 | 入场 -> trailing stop 止盈 | A | ** | T | H | 1k |

---

### 2C. Perp 策略（Hyperliquid） — 25 条

#### 基础交易

| # | 策略 | 入场 | 出场 / 风控 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|------|------------|------|--------|------|------|------|
| 57 | **Perp DCA** | 分 N 批建仓，每批间隔 T | 全部建仓后设 TP/SL | C | * | * | D | 1k |
| 58 | **Perp Grid** | 永续合约区间 [P_low, P_high] 网格 | 区间突破止损 | M | ** | R | H-D | 2k |
| 59 | **Auto TP/SL** | 开仓即设止盈止损 + trailing stop | 触发 TP/SL/trailing | C | * | * | H | 500 |
| 60 | **Limit Order Chain** | 首单触发后自动挂下一单 | 级联到最后一单 / 手动中止 | M | ** | T | H | 1k |
| 61 | **Scale In/Out** | 分批加仓/减仓到目标仓位 | 达到目标仓位 | M | * | * | H-D | 1k |
| 62 | **多空反转** | 趋势信号反转（MA/MACD/RSI） | 自动翻仓，反向建仓 | A | ** | T | H-D | 2k |

#### Funding Rate 套利

| # | 策略 | 入场条件 | 出场 / 风控 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|---------|------------|------|--------|------|------|------|
| 63 | **Funding Rate Farm** | 正费率做空 / 负费率做多 | 费率反转 > N 期 | C | * | * | D | 3k |
| 64 | **Funding Rate Arb** | 跨交易所 funding 差异 > X bp | 差异收敛 / 固定持有期 | M | *** | * | H-D | 10k |
| 65 | **Funding Prediction** | 模型预测下期 funding 方向 | 提前布仓，结算后退出 | A | ** | * | H | 3k |
| 66 | **Funding + Spot Hedge (Cash-and-Carry)** | Spot 多 + Perp 空，正 funding | Funding 转负 / 达到目标收益 | C | *** | * | D-W | 5k |

#### 仓位管理 / 风控

| # | 策略 | 触发条件 | 执行 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|---------|------|------|--------|------|------|------|
| 67 | **Liquidation Guard** | 清算距离 < X% | 自动减仓 / 补保证金 | C | * | * | min | 1k |
| 68 | **Auto Deleverage** | 组合杠杆 > 阈值 | 自动降杠杆到目标水平 | C | * | * | H | 1k |
| 69 | **Dynamic Position Sizing** | 每笔交易前计算 | 根据波动率 / Kelly 公式定仓位 | M | ** | * | H | 2k |
| 70 | **Correlation Monitor** | 持仓间相关性 > 阈值 | 预警 / 自动减仓高相关资产 | M | ** | * | D | 5k |
| 71 | **Drawdown Breaker** | 累计亏损 > X% | 暂停所有策略，全仓平仓 | C | * | * | min | 500 |

#### Copy Trading

| # | 策略 | 入场 | 出场 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|------|------|------|--------|------|------|------|
| 72 | **Perp Copy Trading** | Top trader 开仓 -> 跟 | Trader 平仓 -> 跟平 | M | * | * | H-D | 1k |
| 73 | **Multi-Trader Blend** | 多 trader 加权跟单 | 加权平仓 | M | ** | * | D | 3k |
| 74 | **Selective Copy** | 只跟特定 Token/方向/杠杆范围 | 条件不符不跟 | M | ** | * | D | 1k |
| 75 | **Inverse Copy** | 反向跟踪长期亏损交易员 | 与被跟对象操作相反 | A | ** | * | D | 2k |
| 76 | **Vault Auto-Invest** | 自动投入高表现 Vault | Vault 表现下滑撤出 | C | * | * | W | 1k |

#### 高级 / 量化

| # | 策略 | 入场 | 出场 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|------|------|------|--------|------|------|------|
| 77 | **Volatility Trading** | 隐含 vs 实际波动率偏差 > X | 偏差收敛 | A | *** | V | H-D | 5k |
| 78 | **OI Divergence** | OI 变化与价格走势背离 | 背离修复 / 止损 | A | ** | * | H | 3k |
| 79 | **Liquidation Cascade Catcher** | 检测大规模清算链 | 提前布仓吃反弹 / 快速止损 | A | ** | V | min | 5k |
| 80 | **Market Making** | 挂双边单赚 bid-ask spread | 单边暴露 > 阈值平仓 | A | *** | R-Q | min | 10k |
| 81 | **Statistical Arb** | Token 间统计偏离 > N 标准差 | 均值回归 / 止损 | A | *** | * | H-D | 10k |

---

### 2D. Spot x Perp 组合策略 — 39 条（核心差异化）

> **这是 Byreal 最大的护城河。** Spot 工具和 Perp 工具各有很多，但跨场景自动组合策略几乎无人产品化。

#### 对冲 — Delta 管理

| # | 策略 | 构建方式 | 目标收益来源 | 主要风险 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|---------|------------|---------|------|--------|------|------|------|
| 82 | **Cash-and-Carry** | Spot 多 + Perp 空 | Funding rate | Funding 反转 | C | *** | * | D-W | 5k |
| 83 | **Reverse Cash-and-Carry** | Spot 卖 + Perp 多 | 负 Funding rate | Funding 转正 | M | *** | * | D-W | 5k |
| 84 | **Hedged LP** | LP + Perp 对冲 IL | Fee - 对冲成本 | 极端行情对冲不足 | M | *** | * | D | 3k |
| 85 | **Delta-Neutral LP** | 精确计算 delta -> Perp 完美对冲 | 纯 Fee 收益 | 再平衡成本、极端行情 | M | *** | R | D | 5k |
| 86 | **Portfolio Insurance** | 现货组合 + 跌破阈值开 Perp 空单 | 下跌保护 | 保险成本（Funding） | C | ** | * | D | 3k |
| 87 | **Spot 吸筹 + Perp 保护** | DCA Spot + Perp 短期空单 | 低成本建仓 | 上涨时空单亏损 | M | ** | * | D-W | 3k |
| 88 | **Hedged Launchpad** | 打新 + Perp 空 | 锁定打新利润 | 项目无 Perp 标的 | A | *** | * | H | 3k |
| 89 | **Collar Strategy** | Spot 多 + Perp 上下限 | 限制波动区间 | 放弃上行空间 | C | *** | * | D-W | 5k |

#### 合成金融工具

| # | 策略 | 构建方式 | 等效工具 | Payoff 特征 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|---------|---------|------------|------|--------|------|------|------|
| 90 | **Synthetic Long Call** | Spot 多 + 部分 Perp 空 | 类看涨期权 | 上涨无限盈利，下跌有限损失 | M | *** | T | D-W | 3k |
| 91 | **Synthetic Long Put** | 卖 Spot + Perp 空 | 类看跌期权 | 下跌获利，上涨有限损失 | M | *** | T | D-W | 3k |
| 92 | **Synthetic Straddle** | Spot + Perp 组合 | 做多波动率 | 大幅波动时获利 | A | *** | V | H-D | 5k |
| 93 | **Leveraged Spot** | Perp 多 + Spot 多 | 杠杆化现货 | 放大收益/损失 | A | ** | T | H-D | 2k |
| 94 | **Synthetic Yield** | Spot 持有 + Perp 循环做空/做多 | 合成固收 | 稳定 Funding 收益 | C | ** | * | D-W | 5k |

#### 多腿协同 — 收益再循环

| # | 策略 | 操作流 | 目标 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|-------|------|------|--------|------|------|------|
| 95 | **Pairs Trading** | Spot 多 A + Perp 空 B | A/B 相关性回归 | M | *** | R | D | 5k |
| 96 | **Cross-Venue Rebalance** | Spot + Perp 组合统一再平衡 | 保持目标敞口 | M | ** | * | D-W | 3k |
| 97 | **Smart Exit** | 卖 Spot 同时平对应 Perp hedge | 无敞口残留 | C | ** | * | H | 1k |
| 98 | **Carry + LP Reinvest** | Funding 收益 -> 自动复投 LP | 双重复利 | M | *** | * | D-W | 5k |
| 99 | **LP Fee -> Perp Capital** | LP Fee -> 桥接 HL -> 增加保证金 | 扩大 Perp 仓位 | M | ** | * | W | 3k |
| 100 | **Perp PnL -> Spot Accumulation** | Perp 盈利 -> 桥回 Solana -> 买入 Spot | 利润变现+再投资 | M | ** | * | W | 3k |
| 101 | **Momentum Spot + Hedge Perp** | 趋势做多 Spot + Perp trailing stop | 上涨时盈利，下跌保护 | M | ** | T | D | 2k |
| 102 | **LP + Funding Double Yield** | Byreal LP Fee + HL Funding Farm | 双重收益 | M | *** | * | D-W | 5k |

#### 跨链协同 — Agent Wallet 优势

| # | 策略 | 操作流 | 目标 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|-------|------|------|--------|------|------|------|
| 103 | **Cross-Chain Arb** | Solana Spot 价 vs HL Perp 价差 | 价差套利 | M | *** | * | min | 10k |
| 104 | **Smart Fund Router** | 机会出现 -> 自动桥接至更高收益链 | 资金利用率最大化 | M | ** | * | D | 5k |
| 105 | **Unified Position Manager** | 一个 Agent 管双链 | 统一 P&L + 风控 | M | *** | * | D | 5k |
| 106 | **Cross-Chain Rebalance** | 双链资金按目标比例调配 | 保持分配平衡 | C | ** | * | W | 3k |
| 107 | **Bridge Timing Optimizer** | 检测桥接费/拥堵 | 最优时机跨链 | C | * | * | H | 1k |

#### LP x Perp 深度联动

| # | 策略 | 操作流 | 目标 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|-------|------|------|--------|------|------|------|
| 108 | **LP Range + Perp Grid 联动** | LP range 与 Perp Grid 共享价格区间 | 双场景收益互补 | M | *** | R | D | 5k |
| 109 | **IL Recovery via Perp** | LP 产生 IL -> Perp 盈利 offset | 对冲 IL 损失 | M | *** | * | D | 3k |
| 110 | **LP Exit -> Perp Continuation** | 撤出 LP -> Perp 延续方向敞口 | 保持市场暴露 | M | ** | T | H | 2k |
| 111 | **Perp Close -> LP Enter** | 平 Perp 盈利仓 -> 利润开 LP | 利润转被动收益 | M | ** | * | H | 2k |
| 112 | **Dynamic Hedge Ratio** | 根据 LP 实时 delta 动态调 Perp 对冲 | 精确中性 | M | *** | * | H | 5k |
| 113 | **CopyFarmer + Perp Hedge** | 复制 Farmer LP + 自动 Perp 对冲 | Copy + 风控 | M | *** | * | D | 5k |

---

### 2E. Meta 策略 — 策略的策略

> 管理和优化上述所有策略的运行方式。

| # | 策略 | 触发 | 执行 | 风险 | 复杂度 | 行情 | 周期 | 资金 |
|---|------|------|------|------|--------|------|------|------|
| 114 | **Strategy Allocator** | 按 Sharpe ratio 重新分配资金 | 动态调整子策略权重 | M | *** | * | W | 10k |
| 115 | **Regime Detector** | 识别市场状态（趋势/震荡/恐慌） | 自动切换策略组合 | M | *** | * | D | 5k |
| 116 | **Strategy A/B Test** | 同资金分两份跑不同参数 | 保留胜出方 | M | ** | * | W | 5k |
| 117 | **Kill Switch** | 全局 drawdown > X% | 一键全平所有仓位+暂停策略 | C | * | * | min | 500 |
| 118 | **Profit Recycler** | 所有策略盈利汇总 | 按规则再分配到其他策略 | C | ** | * | W | 3k |
| 119 | **Sleep Mode** | 低波动 / 非交易时段 | 暂停高频策略，节省 gas | C | * | Q | H | 500 |
| 120 | **Calendar Strategy** | Token unlock / Fed meeting 等事件 | 预设策略切换 | M | ** | * | D-W | 2k |

---

## 三、策略选择矩阵

### 按资金规模

| 资金 | 推荐策略类型 | 代表策略 |
|------|-------------|---------|
| **< 1k** | 单腿简单策略 | #33 Smart DCA, #59 Auto TP/SL, #43 Basket DCA |
| **1k - 5k** | 单资产进阶 + 入门组合 | #32 Grid, #1 Auto Range, #84 Hedged LP |
| **5k - 20k** | 多腿组合策略 | #82 Cash-and-Carry, #85 Delta-Neutral LP, #102 Double Yield |
| **> 20k** | Meta 策略 + 全套组合 | #114 Allocator, #105 Unified Manager, #80 Market Making |

### 按风险偏好

| 风险偏好 | 策略范围 | 预期年化 | 最大回撤 |
|----------|---------|---------|---------|
| **保守** (C) | DCA、Funding Farm、Auto Compound、Portfolio Insurance | 10-30% | < 5% |
| **中性** (M) | Grid、Hedged LP、Copy Trading、Pairs Trading | 20-60% | 5-15% |
| **激进** (A) | Momentum、Launchpad Sniper、Leveraged Spot、Volatility Trading | 50-200%+ | 15-50%+ |

### 按行情适配

| 行情 | 最佳策略 |
|------|---------|
| **牛市趋势 (T)** | #35 Momentum Breakout, #93 Leveraged Spot, #101 Momentum + Hedge |
| **震荡盘整 (R)** | #32 Grid, #58 Perp Grid, #85 Delta-Neutral LP, #81 Stat Arb |
| **高波动 (V)** | #3 Narrow Sniper, #92 Synthetic Straddle, #79 Liquidation Catcher |
| **低波动 (Q)** | #82 Cash-and-Carry, #9 Auto Compound, #63 Funding Farm |
| **熊市下跌** | #86 Portfolio Insurance, #89 Collar, #91 Synthetic Put |

---

## 四、实施优先级

### P0 — 核心差异化（立即做）

> 这些策略定义了 Byreal 的独特价值，竞品无法轻易复制。

| 策略 | 理由 |
|------|------|
| #82 Cash-and-Carry | 经典套利，风险低，用户易理解 |
| #84 Hedged LP | LP 用户的第一需求 |
| #85 Delta-Neutral LP | Byreal LP + HL Perp 独家 |
| #105 Unified Position Manager | 跨链管理的入口 |
| #117 Kill Switch | 安全底线，必须有 |

### P1 — 高用户需求

| 策略 | 理由 |
|------|------|
| #33 Smart DCA | 用户基数最大的需求 |
| #32 Grid Trading | 经典策略，用户认知成本低 |
| #63 Funding Rate Farm | Perp 用户核心需求 |
| #1 Auto Range Rebalance | LP 基础功能 |
| #59 Auto TP/SL | Perp 基础风控 |
| #9 Auto Compound | LP 基础收益优化 |

### P2 — 差异化增强

> CopyFarmer 增强、跨链协同、合成金融工具

### P3 — 生态扩展

> Meta 策略、Strategy Marketplace、高级量化

---

## 五、产品 & 商业化

### 分发矩阵

| 渠道 | Spot 策略 | LP 策略 | Perp 策略 | 组合策略 | Meta 策略 |
|------|-----------|---------|-----------|---------|-----------|
| **CLI** | 全量 | 全量 | 全量 | 全量 | 全量 |
| **MCP (AI IDE)** | 全量 | 全量 | 全量 | 全量 | 全量 |
| **Web Dashboard** | 可视化配置 | 可视化配置 | 可视化配置 | 可视化配置 | 可视化配置 |
| **TG Bot** | 简化版 | 监控 | 简化版 | 预设模板 | Kill Switch |
| **Strategy Marketplace** | — | — | — | 用户共创 | — |

### 收入来源

| 模式 | 适用阶段 | 预估占比 |
|------|---------|---------|
| **Performance Fee** — 策略利润 X% | P0 上线即启 | 40% |
| **Execution Fee** — 每笔自动执行 | P0 上线即启 | 20% |
| **Bridge Fee** — 跨链划转 | P0 跨链策略 | 10% |
| **Subscription** — 月费解锁高级策略 | P1 | 15% |
| **Marketplace Commission** — 策略交易抽成 | P3 | 10% |
| **White-label / API** — B2B | P3 | 5% |

---

## 六、关键洞察

### 1. 最大护城河：Spot x Perp 自动组合

单独的 Spot 或 Perp 工具很多。但**跨场景自动组合**（Hedged LP、Cash-and-Carry、Delta-Neutral Farming）几乎无人产品化。**2D 章节的 39 条策略是核心竞争力。**

### 2. 最大杠杆：Agent Wallet + 跨链

Agent Wallet 让用户不需要手动桥接、不需要在两链分别操作。一个入口，Agent 自动决策资金分配和执行路径。

### 3. 最大机会：策略即服务

120 条策略不需全部自建。通过 Strategy Composition Engine + Marketplace，让社区贡献策略，Byreal 提供执行基础设施。

### 4. NL 配置的终极形态

```
"帮我用 $5000，一半 DCA 买 SOL 现货，另一半在 Hyperliquid 做空 SOL 2x 对冲，
如果 funding rate 连续 3 天为正就平掉空单让多头裸跑"
```

这种多腿、多链、条件触发的策略，只有 NL + Agent Wallet 能让普通用户触达。
