# MCP_INTEGRATION — 麦当劳 MCP 集成说明

## 使用的 MCP Server

| 项目 | 值 |
|------|-----|
| Server | 麦当劳中国官方 MCP Server |
| 接入地址 | `https://mcp.mcd.cn` |
| 协议 | Streamable HTTP |
| 鉴权 | `Authorization: Bearer <MCP_TOKEN>`（Token 由 https://open.mcd.cn/mcp 申请，仅存本地配置，绝不入库） |

## 工具使用清单与调用流程

### 1. 抽签阶段（本地，不走 MCP）

- 从 `skills/mclucky/fortune/luzu-signs.json`（吕祖灵签 100 签）随机抽签
- 按 `grade`（上上/上吉/中吉/中平/下下）查 `meal-mapping.json` 得到点餐策略

### 2. 策略执行阶段（MCP 调用链）

```
now-time-info                      # 获取当前日期，供活动日历与早餐/正餐判断
        │
query-nearby-stores                # 用户给地址后查附近门店（或 delivery-query-stores 外送场景）
        │
query-meals                        # 拉当前门店可售菜单
        │
┌─ 策略分支（由签级决定）──────────────────────────────┐
│ 上上：campaign-calendar 查当日活动，找可叠加的营销加成      │
│ 上吉：query-my-coupons 挑最优券                        │
│ 中吉：available-coupons 查可领券，提示领取                │
│ 中平：直接按轻食关键词选品                              │
│ 下下：auto-bind-coupons 一键领全部券 + query-my-coupons  │
└──────────────────────────────────────────────────┘
        │
query-meal-detail                  # 确认套餐组成与可替换项
        │
calculate-price                    # 含券计价：餐品金额 + 配送费 - 优惠 = 应付
        │
【人工确认】用户明确同意后才继续
        │
create-order                       # 创建订单，返回订单详情与支付链接
        │
（演示场景）cancel-order             # 演示完毕取消订单，避免真实扣款
```

### 3. 辅助工具

| 工具 | 触发场景 |
|------|---------|
| query-order | 用户问「我的单到哪了」 |
| query-my-account | 下下签附加环节：查积分，提示「积分别浪费，转运要靠麦麦商城」 |
| query-my-coupons | 任意签级查用户已有券 |

## 业务价值

1. **情绪化点餐的决策引擎**：把「今天吃什么」这个千年难题交给玄学，趣味化解选择困难
2. **优惠券激活器**：下下签强制触达领券链路（auto-bind-coupons），提升用户券使用率与复购
3. **活动日历分发器**：上上签自动关联 campaign-calendar 营销活动，为官方活动带来曝光
4. **完整交易闭环**：选品→计价→下单→支付链接全链路可用，非仅查询类玩具

## 安全声明

- MCP Token 仅保存在本地 `~/.workbuddy/mcp.json`，仓库与日志中不出现任何真实凭证
- 所有写操作（create-order）前强制人工确认；演示订单一律 cancel-order 回收
