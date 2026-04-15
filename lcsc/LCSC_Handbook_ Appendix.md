# 附录 A：《AI 决策提示词模板》

下面这份模板，你以后可以直接复制给 AI 使用。  
它的目标是：让 AI 严格按照 **LCSC Handbook V2.1** 的规则做分析，而不是泛泛而谈。

---

## A.1 通用系统提示词模板

### AI 决策系统提示词（System Prompt）

你是一个严格依据《LCSC Handbook V2.1》进行决策的期权交易助手。  
你的任务不是提供泛泛市场评论，而是基于用户输入的市场数据、候选期权参数和当前持仓，输出：

1. **结构化决策结果（JSON）**
2. **供人类阅读的分析与操作建议**

你必须遵守以下原则。

#### 一、决策原则

1. 先判断趋势，再判断结构。
2. 趋势不成立时，不建议开新多头。
3. IV 决定阵型，不决定方向。
4. 高 IV / 事件驱动高 IV 时，优先考虑 Bull Call Spread，而不是单腿 Long Call。
5. 若单腿 LEAPS 成本过高、仓位过于集中，则优先考虑 Bull Call Spread。
6. Long Call 的止损以标的趋势失效为主，不以期权亏损比例为唯一标准。
7. Long Call Delta > 0.95 时，评估是否 Roll Up。
8. Long Call 剩余期限 < 180 天时，评估是否 Roll Out 或退出。
9. PMCC 不是默认动作，只有在满足条件时才允许。
10. PMCC 短腿 Delta ≥ 0.35 进入警戒，≥ 0.50 应主动处理。
11. 若数据不足，不要强行给结论，应明确要求补充信息。

#### 二、输出要求

你的输出必须包含两部分。

##### 第一部分：结构化 JSON

JSON 必须字段齐全、层次明确、适合程序读取。

##### 第二部分：自然语言说明

需要面向人类解释以下内容：

- 当前趋势判断
- IV 环境判断
- 候选结构比较
- 为什么推荐该动作
- 风险点在哪里
- 如果市场变化，下一步怎么做

#### 三、禁止事项

1. 不要输出模糊建议，如“可以考虑看看”。
2. 不要跳过规则直接下结论。
3. 不要忽视 IV 和时间风险。
4. 不要把 PMCC 当成默认增强。
5. 不要在趋势失效时因为“便宜”而建议补仓。

#### 四、输出动作枚举

你的建议动作必须从以下枚举中选择：

- `WAIT`
- `OPEN_LONG_CALL`
- `OPEN_BULL_CALL_SPREAD`
- `HOLD_LONG_CALL`
- `EXIT_LONG_CALL`
- `PARTIAL_TAKE_PROFIT`
- `ROLL_UP_LONG_CALL`
- `ROLL_OUT_LONG_CALL`
- `START_PMCC`
- `HOLD_PMCC`
- `CLOSE_SHORT_CALL`
- `ROLL_SHORT_CALL_UP_AND_OUT`
- `EXIT_ALL`

#### 五、决策信心

输出中必须给出：

- `confidence_score`：0-100
- `confidence_level`：`LOW / MEDIUM / HIGH`

#### 六、失效条件

每次建议都要明确写出失效条件或重新评估条件。

---

## A.2 用户输入提示词模板

### AI 决策用户输入模板

请严格依据《LCSC Handbook V2.1》分析以下数据，并输出：

1. **结构化 JSON**
2. **面向人类的详细分析说明**

请不要泛泛评论，必须给出明确建议动作，并说明触发规则。

#### 市场信息

- 标的：
- 当前价格：
- 200-DMA：
- 50-DMA：
- 20-DMA：
- RSI：
- 当前 IV：
- IVR：
- 是否存在重大事件风险（如战争 / 谈判 / FOMC / CPI / 财报 / 政策事件）：
- 当前趋势描述（如有）：

#### 候选期权列表

请逐项分析，字段如下：

    [
      {
        "symbol": "",
        "expiration": "",
        "strike": 0,
        "bid": 0,
        "ask": 0,
        "delta": 0,
        "gamma": 0,
        "theta": 0,
        "vega": 0,
        "iv": 0
      }
    ]

#### 当前持仓（如有）

    {
      "has_position": true,
      "position_type": "LONG_CALL / BULL_CALL_SPREAD / PMCC / NONE",
      "long_call": {
        "expiration": "",
        "strike": 0,
        "cost_basis": 0,
        "current_mid": 0,
        "delta": 0,
        "theta": 0,
        "vega": 0,
        "dte": 0
      },
      "short_call": {
        "expiration": "",
        "strike": 0,
        "entry_credit": 0,
        "current_mid": 0,
        "delta": 0,
        "theta": 0,
        "dte": 0
      },
      "unrealized_pnl_pct": 0,
      "account_size": 0,
      "notes": ""
    }

#### 输出要求

- 必须先判断趋势是否允许做多
- 必须判断 IV 环境是否适合单腿 Long Call
- 必须判断账户 / 仓位是否适合单腿 LEAPS
- 若已有 PMCC，必须给出短腿处理建议
- 输出需要包含：
  - `recommended_action`
  - `reasons`
  - `candidate_ranking`
  - `risk_flags`
  - `invalidation_conditions`
  - `next_monitoring_points`

---

# 附录 B：结构化输出规范

为了方便程序化，建议统一输出 JSON Schema 风格。

## B.1 标准 JSON 输出结构

    {
      "meta": {
        "strategy_version": "LCSC Handbook V2.1",
        "timestamp": "",
        "underlying": "",
        "analysis_mode": "OPEN_POSITION / MANAGE_POSITION / PMCC_MANAGEMENT / FULL_REVIEW"
      },
      "market_assessment": {
        "trend_allowed": true,
        "trend_score": 0,
        "trend_reason": [],
        "price_vs_200dma": "",
        "price_vs_50dma": "",
        "price_vs_20dma": "",
        "rsi_state": "NEUTRAL / HOT / EXTREME",
        "event_risk": {
          "has_event_risk": true,
          "event_type": [],
          "comment": ""
        }
      },
      "volatility_assessment": {
        "iv_level": "",
        "ivr_level": "",
        "target_contract_iv_assessment": "",
        "iv_crush_risk": "LOW / MEDIUM / HIGH",
        "volatility_reason": []
      },
      "account_assessment": {
        "account_size": null,
        "single_leg_affordable": null,
        "concentration_risk": "LOW / MEDIUM / HIGH / UNKNOWN",
        "comment": ""
      },
      "candidate_analysis": [
        {
          "candidate_id": "",
          "structure_type": "LONG_CALL / BULL_CALL_LONG_LEG / BULL_CALL_SHORT_LEG / SHORT_CALL_PMCC",
          "expiration": "",
          "strike": 0,
          "bid": 0,
          "ask": 0,
          "mid": 0,
          "spread_abs": 0,
          "spread_pct_mid": 0,
          "delta": 0,
          "gamma": 0,
          "theta": 0,
          "vega": 0,
          "iv": 0,
          "dte": 0,
          "quality_score": 0,
          "fit_score": 0,
          "pros": [],
          "cons": [],
          "decision_tag": "PREFERRED / ACCEPTABLE / REJECT"
        }
      ],
      "existing_position_assessment": {
        "has_position": false,
        "position_type": "NONE",
        "position_health": "N/A / GOOD / WARNING / EXIT",
        "pnl_pct": null,
        "long_call_status": null,
        "short_call_status": null,
        "roll_up_needed": false,
        "roll_out_needed": false,
        "pmcc_action_needed": false,
        "reason": []
      },
      "decision": {
        "recommended_action": "WAIT",
        "action_priority": 1,
        "confidence_score": 0,
        "confidence_level": "LOW / MEDIUM / HIGH",
        "primary_reasons": [],
        "rejected_alternatives": [
          {
            "action": "",
            "reason": ""
          }
        ]
      },
      "execution_plan": {
        "structure": "",
        "legs": [],
        "entry_guidance": {
          "order_type": "LIMIT",
          "price_guidance": "",
          "time_guidance": ""
        },
        "risk_management": {
          "stop_rule": [],
          "take_profit_rule": [],
          "roll_rule": []
        }
      },
      "risk_flags": [],
      "invalidation_conditions": [],
      "next_monitoring_points": [],
      "human_readable_summary": {
        "headline": "",
        "summary": "",
        "detailed_reasoning": []
      }
    }

---

# 附录 C：《标准输入样例 + 标准输出样例》

下面给 4 组典型案例：

1. **开 Long Call**
2. **开 Bull Call Spread**
3. **已有 Long Call，执行 Roll Up**
4. **已有 PMCC，短腿被快速拉升击穿**

---

## 样例 1：趋势良好、IV 中低、适合开 Long Call

### 输入样例

    {
      "market_info": {
        "underlying": "QQQ",
        "current_price": 512,
        "dma_200": 498,
        "dma_50": 503,
        "dma_20": 509,
        "rsi": 66,
        "current_iv": 19,
        "ivr": 24,
        "event_risk": {
          "has_event_risk": false,
          "event_type": []
        }
      },
      "candidate_options": [
        {
          "symbol": "QQQ",
          "expiration": "2027-01-15",
          "strike": 430,
          "bid": 96.5,
          "ask": 98.0,
          "delta": 0.81,
          "gamma": 0.003,
          "theta": -0.055,
          "vega": 0.42,
          "iv": 20.1
        },
        {
          "symbol": "QQQ",
          "expiration": "2027-01-15",
          "strike": 450,
          "bid": 82.0,
          "ask": 83.6,
          "delta": 0.75,
          "gamma": 0.0035,
          "theta": -0.061,
          "vega": 0.44,
          "iv": 20.0
        }
      ],
      "position_info": {
        "has_position": false,
        "position_type": "NONE",
        "account_size": 120000
      }
    }

### 标准输出样例

    {
      "meta": {
        "strategy_version": "LCSC Handbook V2.1",
        "timestamp": "2026-04-15T10:00:00Z",
        "underlying": "QQQ",
        "analysis_mode": "OPEN_POSITION"
      },
      "market_assessment": {
        "trend_allowed": true,
        "trend_score": 86,
        "trend_reason": [
          "当前价格高于200-DMA",
          "50-DMA高于200-DMA且向上",
          "20-DMA高于50-DMA，短中期趋势一致"
        ],
        "price_vs_200dma": "ABOVE",
        "price_vs_50dma": "ABOVE",
        "price_vs_20dma": "ABOVE",
        "rsi_state": "NEUTRAL",
        "event_risk": {
          "has_event_risk": false,
          "event_type": [],
          "comment": "无明显事件驱动IV抬升"
        }
      },
      "volatility_assessment": {
        "iv_level": "LOW_TO_MID",
        "ivr_level": "LOW",
        "target_contract_iv_assessment": "适合单腿Long Call",
        "iv_crush_risk": "LOW",
        "volatility_reason": [
          "IVR为24，处于相对低位",
          "无明显事件溢价"
        ]
      },
      "account_assessment": {
        "account_size": 120000,
        "single_leg_affordable": true,
        "concentration_risk": "LOW",
        "comment": "单张长期Call成本约占账户8%左右，可承受"
      },
      "candidate_analysis": [
        {
          "candidate_id": "C1",
          "structure_type": "LONG_CALL",
          "expiration": "2027-01-15",
          "strike": 430,
          "bid": 96.5,
          "ask": 98.0,
          "mid": 97.25,
          "spread_abs": 1.5,
          "spread_pct_mid": 1.54,
          "delta": 0.81,
          "gamma": 0.003,
          "theta": -0.055,
          "vega": 0.42,
          "iv": 20.1,
          "dte": 640,
          "quality_score": 88,
          "fit_score": 92,
          "pros": [
            "Delta位于标准长期底仓优选区间",
            "到期时间符合LEAPS要求",
            "IV环境适合单腿"
          ],
          "cons": [
            "绝对权利金较高"
          ],
          "decision_tag": "PREFERRED"
        },
        {
          "candidate_id": "C2",
          "structure_type": "LONG_CALL",
          "expiration": "2027-01-15",
          "strike": 450,
          "bid": 82.0,
          "ask": 83.6,
          "mid": 82.8,
          "spread_abs": 1.6,
          "spread_pct_mid": 1.93,
          "delta": 0.75,
          "gamma": 0.0035,
          "theta": -0.061,
          "vega": 0.44,
          "iv": 20.0,
          "dte": 640,
          "quality_score": 84,
          "fit_score": 86,
          "pros": [
            "成本略低",
            "Delta仍在可接受区间"
          ],
          "cons": [
            "时间价值占比相对更高",
            "Theta略高于430行权价"
          ],
          "decision_tag": "ACCEPTABLE"
        }
      ],
      "existing_position_assessment": {
        "has_position": false,
        "position_type": "NONE",
        "position_health": "N/A",
        "pnl_pct": null,
        "long_call_status": null,
        "short_call_status": null,
        "roll_up_needed": false,
        "roll_out_needed": false,
        "pmcc_action_needed": false,
        "reason": []
      },
      "decision": {
        "recommended_action": "OPEN_LONG_CALL",
        "action_priority": 1,
        "confidence_score": 88,
        "confidence_level": "HIGH",
        "primary_reasons": [
          "趋势允许做多",
          "IV环境适合单腿Long Call",
          "账户可承受标准LEAPS成本",
          "430行权价合约Delta和时间结构更优"
        ],
        "rejected_alternatives": [
          {
            "action": "OPEN_BULL_CALL_SPREAD",
            "reason": "当前IV不高，且账户承受能力良好，没有必要过早封顶上行空间"
          },
          {
            "action": "WAIT",
            "reason": "当前趋势和估值条件都支持建仓，不必继续等待"
          }
        ]
      },
      "execution_plan": {
        "structure": "LONG_CALL",
        "legs": [
          {
            "action": "BUY",
            "expiration": "2027-01-15",
            "strike": 430,
            "target_delta": 0.81
          }
        ],
        "entry_guidance": {
          "order_type": "LIMIT",
          "price_guidance": "优先以mid附近挂单，约97.25，若无法成交可小步上调",
          "time_guidance": "优先在常规交易时段中段执行，避免开盘和重大事件窗口"
        },
        "risk_management": {
          "stop_rule": [
            "若QQQ跌破200-DMA且连续3日无法收回，退出",
            "若趋势结构破坏且期权亏损达到35%-40%，退出"
          ],
          "take_profit_rule": [
            "若前6个月盈利达到50%，考虑减仓1/3",
            "若收益80%-100%且RSI>80，进一步止盈"
          ],
          "roll_rule": [
            "若Delta>0.95，考虑Roll Up",
            "若DTE<180，考虑Roll Out"
          ]
        }
      },
      "risk_flags": [
        "长期Call绝对价格较高，仍需注意单笔集中度"
      ],
      "invalidation_conditions": [
        "QQQ重新跌回200-DMA下方并连续3日站不回",
        "市场进入事件驱动高IV环境且合约IV显著上升"
      ],
      "next_monitoring_points": [
        "200-DMA得失",
        "Delta是否接近0.95",
        "IV是否显著抬升",
        "持仓收益是否快速达到50%"
      ],
      "human_readable_summary": {
        "headline": "建议开标准Long Call，优先选择2027-01-15 430 Call",
        "summary": "当前QQQ趋势结构健康，IV处于低至中低区间，账户规模也足以承受单腿LEAPS，适合用标准长期Call建立多头底仓。",
        "detailed_reasoning": [
          "QQQ当前价格明显高于200-DMA，且20-DMA、50-DMA、200-DMA呈多头排列，趋势过滤通过。",
          "IVR只有24，说明当前并不是典型的高IV环境，不需要为了规避IV Crush而强行用价差封顶上行空间。",
          "430行权价Call的Delta约0.81，处于本手册定义的标准长期底仓区间，更接近模拟股票，同时Theta也更温和。",
          "相比450行权价，430行权价虽然更贵，但结构更稳健，更符合长期趋势持有逻辑。"
        ]
      }
    }

---

## 样例 2：事件驱动高 IV，适合 Bull Call Spread

### 输入样例

    {
      "market_info": {
        "underlying": "QQQ",
        "current_price": 500,
        "dma_200": 494,
        "dma_50": 497,
        "dma_20": 499,
        "rsi": 61,
        "current_iv": 27,
        "ivr": 63,
        "event_risk": {
          "has_event_risk": true,
          "event_type": ["WAR_NEGOTIATION", "MACRO_EVENT"]
        }
      },
      "candidate_options": [
        {
          "symbol": "QQQ",
          "expiration": "2027-01-15",
          "strike": 420,
          "bid": 96,
          "ask": 99,
          "delta": 0.80,
          "gamma": 0.003,
          "theta": -0.065,
          "vega": 0.51,
          "iv": 28
        },
        {
          "symbol": "QQQ",
          "expiration": "2027-01-15",
          "strike": 580,
          "bid": 21,
          "ask": 22,
          "delta": 0.27,
          "gamma": 0.002,
          "theta": -0.031,
          "vega": 0.36,
          "iv": 31
        }
      ],
      "position_info": {
        "has_position": false,
        "position_type": "NONE",
        "account_size": 30000
      }
    }

### 标准输出样例

    {
      "meta": {
        "strategy_version": "LCSC Handbook V2.1",
        "timestamp": "2026-04-15T10:05:00Z",
        "underlying": "QQQ",
        "analysis_mode": "OPEN_POSITION"
      },
      "market_assessment": {
        "trend_allowed": true,
        "trend_score": 78,
        "trend_reason": [
          "价格位于200-DMA上方",
          "50-DMA和20-DMA也支持短中期多头结构"
        ],
        "price_vs_200dma": "ABOVE",
        "price_vs_50dma": "ABOVE",
        "price_vs_20dma": "ABOVE",
        "rsi_state": "NEUTRAL",
        "event_risk": {
          "has_event_risk": true,
          "event_type": ["WAR_NEGOTIATION", "MACRO_EVENT"],
          "comment": "存在明显事件驱动因素，IV可能包含不确定性溢价"
        }
      },
      "volatility_assessment": {
        "iv_level": "HIGH",
        "ivr_level": "HIGH",
        "target_contract_iv_assessment": "不适合单腿Long Call，优先Bull Call Spread",
        "iv_crush_risk": "HIGH",
        "volatility_reason": [
          "IVR为63，处于高位",
          "事件风险可能在未来数周落地，IV有回落风险",
          "Long Call Vega暴露明显"
        ]
      },
      "account_assessment": {
        "account_size": 30000,
        "single_leg_affordable": false,
        "concentration_risk": "HIGH",
        "comment": "420 Call单张成本接近账户三分之一，不适合直接做标准LEAPS单腿"
      },
      "candidate_analysis": [
        {
          "candidate_id": "L1",
          "structure_type": "BULL_CALL_LONG_LEG",
          "expiration": "2027-01-15",
          "strike": 420,
          "bid": 96,
          "ask": 99,
          "mid": 97.5,
          "spread_abs": 3,
          "spread_pct_mid": 3.08,
          "delta": 0.80,
          "gamma": 0.003,
          "theta": -0.065,
          "vega": 0.51,
          "iv": 28,
          "dte": 640,
          "quality_score": 78,
          "fit_score": 84,
          "pros": [
            "Delta合理",
            "可作为Bull Call Spread的买入腿"
          ],
          "cons": [
            "单腿成本高",
            "Vega暴露较大"
          ],
          "decision_tag": "PREFERRED"
        },
        {
          "candidate_id": "S1",
          "structure_type": "BULL_CALL_SHORT_LEG",
          "expiration": "2027-01-15",
          "strike": 580,
          "bid": 21,
          "ask": 22,
          "mid": 21.5,
          "spread_abs": 1,
          "spread_pct_mid": 4.65,
          "delta": 0.27,
          "gamma": 0.002,
          "theta": -0.031,
          "vega": 0.36,
          "iv": 31,
          "dte": 640,
          "quality_score": 74,
          "fit_score": 88,
          "pros": [
            "Delta适合作为卖出腿",
            "IV更高，卖出有助于对冲高IV"
          ],
          "cons": [
            "点差略宽"
          ],
          "decision_tag": "PREFERRED"
        }
      ],
      "existing_position_assessment": {
        "has_position": false,
        "position_type": "NONE",
        "position_health": "N/A",
        "pnl_pct": null,
        "long_call_status": null,
        "short_call_status": null,
        "roll_up_needed": false,
        "roll_out_needed": false,
        "pmcc_action_needed": false,
        "reason": []
      },
      "decision": {
        "recommended_action": "OPEN_BULL_CALL_SPREAD",
        "action_priority": 1,
        "confidence_score": 93,
        "confidence_level": "HIGH",
        "primary_reasons": [
          "趋势允许做多",
          "当前IV和IVR处于高位，存在明显IV Crush风险",
          "账户规模不适合单腿LEAPS",
          "Bull Call Spread可降低成本并缓解Vega风险"
        ],
        "rejected_alternatives": [
          {
            "action": "OPEN_LONG_CALL",
            "reason": "高IV + 事件驱动 + 账户规模较小，单腿Long Call风险过高"
          },
          {
            "action": "WAIT",
            "reason": "虽然等待也合理，但趋势已经通过，使用价差能在控制风险的同时参与上涨"
          }
        ]
      },
      "execution_plan": {
        "structure": "BULL_CALL_SPREAD",
        "legs": [
          {
            "action": "BUY",
            "expiration": "2027-01-15",
            "strike": 420,
            "target_delta": 0.80
          },
          {
            "action": "SELL",
            "expiration": "2027-01-15",
            "strike": 580,
            "target_delta": 0.27
          }
        ],
        "entry_guidance": {
          "order_type": "LIMIT",
          "price_guidance": "按净借记价格挂单，优先参考买入腿mid 97.5、卖出腿mid 21.5，组合净借记约76附近，小步调整",
          "time_guidance": "避免重大事件公布前后立即成交"
        },
        "risk_management": {
          "stop_rule": [
            "若QQQ跌破200-DMA且连续3日无法收回，退出价差",
            "若事件落地后趋势未延续且价差价值明显衰减，可提前止损"
          ],
          "take_profit_rule": [
            "若标的快速接近580行权价，且价差价值达到最大收益的70%-85%，可提前止盈"
          ],
          "roll_rule": [
            "Bull Call Spread一般不需要频繁滚动，除非接近短腿上限且仍强烈看多"
          ]
        }
      },
      "risk_flags": [
        "当前存在事件驱动高IV",
        "若事件反转，价格与IV都可能剧烈波动"
      ],
      "invalidation_conditions": [
        "QQQ跌回200-DMA下方并连续3日无法收复",
        "事件导致趋势破坏而非只是IV回落"
      ],
      "next_monitoring_points": [
        "事件结果是否落地",
        "IV是否快速回落",
        "QQQ是否继续维持在200-DMA上方",
        "价差价值是否达到最大收益的70%-85%"
      ],
      "human_readable_summary": {
        "headline": "建议开Bull Call Spread，而非单腿Long Call",
        "summary": "当前趋势允许做多，但IV受事件影响明显偏高，单腿Long Call会暴露较大的IV Crush风险，同时账户规模也不足以舒适承受单腿LEAPS，因此更适合用Bull Call Spread参与行情。",
        "detailed_reasoning": [
          "QQQ仍在200-DMA之上，趋势过滤通过，因此不是纯粹观望环境。",
          "但IVR已达到63，并且市场受战争谈判与宏观事件影响，期权中很可能包含较高的不确定性溢价。",
          "如果事件两周后落地，IV从27%回到18%-20%，单腿Long Call即使方向看对，也可能因Vega损失而表现不佳。",
          "同时账户规模只有3万美元，单张420 Call成本过高，会造成明显集中度风险。",
          "因此本策略优先推荐Bull Call Spread：既能保留看涨敞口，又能降低初始成本和IV回落伤害。"
        ]
      }
    }

---

## 样例 3：已有 Long Call，Delta 过高，建议 Roll Up

### 输入样例

    {
      "market_info": {
        "underlying": "QQQ",
        "current_price": 560,
        "dma_200": 510,
        "dma_50": 538,
        "dma_20": 552,
        "rsi": 82,
        "current_iv": 18,
        "ivr": 22,
        "event_risk": {
          "has_event_risk": false,
          "event_type": []
        }
      },
      "candidate_options": [
        {
          "symbol": "QQQ",
          "expiration": "2027-01-15",
          "strike": 500,
          "bid": 104,
          "ask": 106,
          "delta": 0.79,
          "gamma": 0.003,
          "theta": -0.066,
          "vega": 0.46,
          "iv": 19
        }
      ],
      "position_info": {
        "has_position": true,
        "position_type": "LONG_CALL",
        "long_call": {
          "expiration": "2027-01-15",
          "strike": 420,
          "cost_basis": 78,
          "current_mid": 144,
          "delta": 0.96,
          "theta": -0.048,
          "vega": 0.31,
          "dte": 620
        },
        "unrealized_pnl_pct": 84.6,
        "account_size": 150000
      }
    }

### 标准输出样例

    {
      "meta": {
        "strategy_version": "LCSC Handbook V2.1",
        "timestamp": "2026-04-15T10:10:00Z",
        "underlying": "QQQ",
        "analysis_mode": "MANAGE_POSITION"
      },
      "market_assessment": {
        "trend_allowed": true,
        "trend_score": 91,
        "trend_reason": [
          "价格远高于200-DMA",
          "中短期均线强势上行",
          "趋势仍然健康"
        ],
        "price_vs_200dma": "ABOVE",
        "price_vs_50dma": "ABOVE",
        "price_vs_20dma": "ABOVE",
        "rsi_state": "EXTREME",
        "event_risk": {
          "has_event_risk": false,
          "event_type": [],
          "comment": "无明显事件风险，但市场处于高热状态"
        }
      },
      "volatility_assessment": {
        "iv_level": "LOW",
        "ivr_level": "LOW",
        "target_contract_iv_assessment": "适合继续持有长期多头结构",
        "iv_crush_risk": "LOW",
        "volatility_reason": [
          "IVR偏低，当前不是IV风险主导环境"
        ]
      },
      "account_assessment": {
        "account_size": 150000,
        "single_leg_affordable": true,
        "concentration_risk": "LOW",
        "comment": "账户足以支持Roll Up"
      },
      "candidate_analysis": [
        {
          "candidate_id": "R1",
          "structure_type": "LONG_CALL",
          "expiration": "2027-01-15",
          "strike": 500,
          "bid": 104,
          "ask": 106,
          "mid": 105,
          "spread_abs": 2,
          "spread_pct_mid": 1.9,
          "delta": 0.79,
          "gamma": 0.003,
          "theta": -0.066,
          "vega": 0.46,
          "iv": 19,
          "dte": 620,
          "quality_score": 86,
          "fit_score": 90,
          "pros": [
            "Delta可将底仓重置回有效杠杆区间",
            "到期日与原仓一致，管理简单"
          ],
          "cons": [
            "新仓Theta略高"
          ],
          "decision_tag": "PREFERRED"
        }
      ],
      "existing_position_assessment": {
        "has_position": true,
        "position_type": "LONG_CALL",
        "position_health": "GOOD",
        "pnl_pct": 84.6,
        "long_call_status": "DELTA_TOO_HIGH",
        "short_call_status": null,
        "roll_up_needed": true,
        "roll_out_needed": false,
        "pmcc_action_needed": false,
        "reason": [
          "当前Long Call Delta已达0.96",
          "收益已较大，杠杆效率下降",
          "RSI极热，适合锁定部分利润并重置结构"
        ]
      },
      "decision": {
        "recommended_action": "ROLL_UP_LONG_CALL",
        "action_priority": 1,
        "confidence_score": 90,
        "confidence_level": "HIGH",
        "primary_reasons": [
          "现有Long Call Delta超过0.95，触发Roll Up规则",
          "持仓盈利超过80%，适合锁定收益",
          "趋势仍强，不必整体退出多头方向"
        ],
        "rejected_alternatives": [
          {
            "action": "HOLD_LONG_CALL",
            "reason": "继续持有并非错误，但杠杆效率已明显下降"
          },
          {
            "action": "EXIT_LONG_CALL",
            "reason": "趋势尚未破坏，不必完全退出多头敞口"
          }
        ]
      },
      "execution_plan": {
        "structure": "ROLL_UP_LONG_CALL",
        "legs": [
          {
            "action": "SELL_TO_CLOSE",
            "expiration": "2027-01-15",
            "strike": 420
          },
          {
            "action": "BUY_TO_OPEN",
            "expiration": "2027-01-15",
            "strike": 500,
            "target_delta": 0.79
          }
        ],
        "entry_guidance": {
          "order_type": "LIMIT",
          "price_guidance": "先平旧仓，再以mid附近开新仓；也可用组合单降低滑点",
          "time_guidance": "避免情绪极热时段追价"
        },
        "risk_management": {
          "stop_rule": [
            "若QQQ跌破200-DMA且连续3日无法收复，退出新仓"
          ],
          "take_profit_rule": [
            "新仓重新计算收益目标，若再次快速盈利50%-80%，分批止盈"
          ],
          "roll_rule": [
            "若未来DTE<180，执行Roll Out"
          ]
        }
      },
      "risk_flags": [
        "RSI高达82，市场短线偏热，Roll Up后需防短期震荡"
      ],
      "invalidation_conditions": [
        "若QQQ出现极端回落并跌破关键趋势结构"
      ],
      "next_monitoring_points": [
        "QQQ对20-DMA和50-DMA的回踩表现",
        "新仓Delta是否仍维持在0.75-0.85区间",
        "RSI是否从极热状态回落"
      ],
      "human_readable_summary": {
        "headline": "建议对现有Long Call执行Roll Up，而不是继续死拿",
        "summary": "当前持仓已经大幅盈利，且Delta达到0.96，说明这张期权越来越像股票，杠杆效率下降。趋势还很强，因此更合理的做法不是全部离场，而是Roll Up到更高行权价，锁定利润并重置杠杆。",
        "detailed_reasoning": [
          "QQQ仍处于明显多头趋势中，因此没有触发整体止损或完全退出的条件。",
          "但现有420 Call的Delta已达到0.96，说明它已经失去了作为高效杠杆工具的部分优势。",
          "持仓收益也已接近85%，同时RSI处于82的极热区，适合将一部分浮盈转化为已实现利润。",
          "滚到500行权价后，可把Delta重新放回约0.79，更符合长期底仓的效率区间。"
        ]
      }
    }

---

## 样例 4：已有 PMCC，短腿被快速拉升击穿

### 输入样例

    {
      "market_info": {
        "underlying": "QQQ",
        "current_price": 535,
        "dma_200": 500,
        "dma_50": 520,
        "dma_20": 529,
        "rsi": 76,
        "current_iv": 20,
        "ivr": 32,
        "event_risk": {
          "has_event_risk": false,
          "event_type": []
        }
      },
      "candidate_options": [
        {
          "symbol": "QQQ",
          "expiration": "2026-06-19",
          "strike": 560,
          "bid": 6.8,
          "ask": 7.3,
          "delta": 0.18,
          "gamma": 0.010,
          "theta": -0.085,
          "vega": 0.14,
          "iv": 21
        }
      ],
      "position_info": {
        "has_position": true,
        "position_type": "PMCC",
        "long_call": {
          "expiration": "2027-01-15",
          "strike": 450,
          "cost_basis": 82,
          "current_mid": 110,
          "delta": 0.84,
          "theta": -0.058,
          "vega": 0.41,
          "dte": 640
        },
        "short_call": {
          "expiration": "2026-05-17",
          "strike": 525,
          "entry_credit": 5.5,
          "current_mid": 11.8,
          "delta": 0.58,
          "theta": -0.12,
          "dte": 18
        },
        "unrealized_pnl_pct": 20.5,
        "account_size": 80000
      }
    }

### 标准输出样例

    {
      "meta": {
        "strategy_version": "LCSC Handbook V2.1",
        "timestamp": "2026-04-15T10:15:00Z",
        "underlying": "QQQ",
        "analysis_mode": "PMCC_MANAGEMENT"
      },
      "market_assessment": {
        "trend_allowed": true,
        "trend_score": 87,
        "trend_reason": [
          "价格位于200-DMA和50-DMA之上",
          "短中期趋势仍偏强"
        ],
        "price_vs_200dma": "ABOVE",
        "price_vs_50dma": "ABOVE",
        "price_vs_20dma": "ABOVE",
        "rsi_state": "HOT",
        "event_risk": {
          "has_event_risk": false,
          "event_type": [],
          "comment": "无事件驱动，但价格上行速度较快"
        }
      },
      "volatility_assessment": {
        "iv_level": "MID",
        "ivr_level": "MID",
        "target_contract_iv_assessment": "波动率不是当前主要矛盾",
        "iv_crush_risk": "LOW",
        "volatility_reason": [
          "当前主要问题是短腿Delta过高，而非IV"
        ]
      },
      "account_assessment": {
        "account_size": 80000,
        "single_leg_affordable": true,
        "concentration_risk": "MEDIUM",
        "comment": "账户可承受回补短腿或重新卖出更高执行价短腿"
      },
      "candidate_analysis": [
        {
          "candidate_id": "NS1",
          "structure_type": "SHORT_CALL_PMCC",
          "expiration": "2026-06-19",
          "strike": 560,
          "bid": 6.8,
          "ask": 7.3,
          "mid": 7.05,
          "spread_abs": 0.5,
          "spread_pct_mid": 7.09,
          "delta": 0.18,
          "gamma": 0.01,
          "theta": -0.085,
          "vega": 0.14,
          "iv": 21,
          "dte": 51,
          "quality_score": 73,
          "fit_score": 84,
          "pros": [
            "Delta适中，可作为新短腿",
            "行权价高于现有被击穿短腿"
          ],
          "cons": [
            "点差略宽"
          ],
          "decision_tag": "PREFERRED"
        }
      ],
      "existing_position_assessment": {
        "has_position": true,
        "position_type": "PMCC",
        "position_health": "WARNING",
        "pnl_pct": 20.5,
        "long_call_status": "GOOD",
        "short_call_status": "THREATENING_UPSIDE",
        "roll_up_needed": false,
        "roll_out_needed": false,
        "pmcc_action_needed": true,
        "reason": [
          "短腿Delta已达0.58，超过强制处理阈值0.50",
          "短腿DTE仅18天，Gamma风险开始上升",
          "当前短腿正在显著压制Long端利润"
        ]
      },
      "decision": {
        "recommended_action": "ROLL_SHORT_CALL_UP_AND_OUT",
        "action_priority": 1,
        "confidence_score": 89,
        "confidence_level": "HIGH",
        "primary_reasons": [
          "短腿Delta已超过0.50，触发主动处理规则",
          "距离到期仅18天，Gamma风险上升",
          "趋势仍强，不宜继续让525短腿压住Long端利润"
        ],
        "rejected_alternatives": [
          {
            "action": "HOLD_PMCC",
            "reason": "短腿已深度威胁上行空间，不适合继续拖延"
          },
          {
            "action": "CLOSE_SHORT_CALL",
            "reason": "直接回补也合理，但如果仍想维持PMCC，向上并向后滚动更平衡"
          }
        ]
      },
      "execution_plan": {
        "structure": "PMCC_SHORT_CALL_ADJUSTMENT",
        "legs": [
          {
            "action": "BUY_TO_CLOSE",
            "expiration": "2026-05-17",
            "strike": 525
          },
          {
            "action": "SELL_TO_OPEN",
            "expiration": "2026-06-19",
            "strike": 560,
            "target_delta": 0.18
          }
        ],
        "entry_guidance": {
          "order_type": "LIMIT",
          "price_guidance": "优先使用组合滚动单，避免单独回补后再卖出导致滑点扩大",
          "time_guidance": "尽量在波动稳定时执行，不在瞬时拉升尖峰追单"
        },
        "risk_management": {
          "stop_rule": [
            "若后续QQQ继续爆发式上涨，新的560短腿Delta再次升至0.35以上，重新评估"
          ],
          "take_profit_rule": [
            "若新短腿盈利达到50%-70%，考虑提前回补"
          ],
          "roll_rule": [
            "若新短腿Delta>=0.50或DTE<=14-21，再次主动处理"
          ]
        }
      },
      "risk_flags": [
        "短腿已严重压制底仓利润",
        "当前RSI偏热，波动可能继续放大"
      ],
      "invalidation_conditions": [
        "若QQQ快速跌回50-DMA以下，PMCC结构需重新评估是否继续卖短腿"
      ],
      "next_monitoring_points": [
        "新短腿Delta是否再次快速升高",
        "QQQ是否进入更强加速段",
        "Long Call Delta是否接近0.95"
      ],
      "human_readable_summary": {
        "headline": "建议将PMCC短腿向上并向后滚动，避免继续压住利润",
        "summary": "当前525短Call的Delta已经达到0.58，且只剩18天到期，短腿已经从收租工具变成明显的利润封顶器。由于QQQ趋势依然偏强，继续死扛不合理，建议回补旧短腿，并滚动到更高执行价、更远到期的新短腿。",
        "detailed_reasoning": [
          "Long端450 Call状态仍然健康，没有必要因为短腿管理失误而牺牲整体多头方向。",
          "但当前短腿525 Call已明显实值，且DTE仅18天，Gamma风险逐渐上升，继续拖延容易让短腿管理更被动。",
          "如果完全不想继续卖短腿，直接回补也是合理选项；但若希望维持PMCC，则滚到560、约0.18 Delta的新短腿更合适。",
          "处理重点不是保住短腿账面，而是恢复Long端利润空间。"
        ]
      }
    }

---

# 附录 D：推荐的双格式输出方案

实际程序中，建议 AI 每次都输出两段。

## D.1 第一段：机器可读 JSON

给程序直接读取。

## D.2 第二段：人类可读 Markdown

例如：

    # 操作建议摘要
    - 建议动作：OPEN_BULL_CALL_SPREAD
    - 信心等级：HIGH
    - 核心原因：
      1. 趋势允许做多
      2. 当前IV偏高且存在事件风险
      3. 单腿LEAPS成本对账户过高
      4. Bull Call Spread更适合当前环境

    # 具体操作
    - 买入：QQQ 2027-01-15 420 Call
    - 卖出：QQQ 2027-01-15 580 Call
    - 下单方式：限价净借记单，优先从mid附近开始挂单

    # 风险控制
    - 若QQQ跌破200-DMA且连续3日无法收回，退出
    - 若价差价值达到最大收益的70%-85%，考虑提前止盈

    # 重点监控
    - 事件落地后的IV变化
    - QQQ是否继续维持在200-DMA上方
    - 是否快速接近短腿行权价

---

# 附录 E：给程序的建议

如果你后续要程序化，建议：

## E.1 把推荐动作标准化成枚举

- `WAIT`
- `OPEN_LONG_CALL`
- `OPEN_BULL_CALL_SPREAD`
- `HOLD_LONG_CALL`
- `EXIT_LONG_CALL`
- `PARTIAL_TAKE_PROFIT`
- `ROLL_UP_LONG_CALL`
- `ROLL_OUT_LONG_CALL`
- `START_PMCC`
- `HOLD_PMCC`
- `CLOSE_SHORT_CALL`
- `ROLL_SHORT_CALL_UP_AND_OUT`
- `EXIT_ALL`

## E.2 把风险标签也标准化

- `HIGH_IV`
- `EVENT_RISK`
- `IV_CRUSH_RISK`
- `TREND_NOT_CONFIRMED`
- `SHORT_CALL_DELTA_WARNING`
- `SHORT_CALL_DELTA_DANGER`
- `DTE_TOO_LOW`
- `OVERBOUGHT`
- `POSITION_CONCENTRATION_HIGH`

## E.3 把失效条件也拆成结构化字段

便于程序轮询触发。

---
