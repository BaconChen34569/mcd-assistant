# MCP 集成说明

本文档说明 mcd-assistant Skill 实际使用的麦当劳 MCP Server、Tool、调用流程与业务价值。

## 一、MCP Server

| 项目 | 内容 |
|---|---|
| 服务提供方 | 麦当劳中国（金拱门（中国）有限公司） |
| Server 名称 | `mcd-mcp` |
| 接入地址 | `https://mcp.mcd.cn` |
| 传输协议 | Streamable HTTP |
| 认证方式 | 请求头 `Authorization: Bearer <MCP_TOKEN>` |
| Token 申请 | https://open.mcd.cn/mcp → 手机号登录 → 控制台 → 激活 |
| 服务区域 | 中国大陆地区（不含港澳台） |
| 限流 | 每个 Token 每分钟最多 600 次请求，超限返回 429 |

本项目为**纯 Skill 项目**，不含任何后端服务或中间层。全部业务能力均由麦当劳 MCP Server 直接提供，Skill 仅负责编排调用顺序、约束参数组合、并向用户复述确认。

## 二、实际使用的 Tool

以下为 mcd-assistant Skill 真实调用或调度的工具，按业务域分组。

### 2.1 门店与地址（6 个）

| Tool | 用途 | 关键入参 |
|---|---|---|
| `query-nearby-stores` | 到店场景门店查询 | `beType`(1/5), `searchType`(1 收藏/2 位置), `city`, `keyword` |
| `delivery-query-addresses` | 用户配送地址列表 | 无 |
| `delivery-create-address` | 新增配送地址 | `city`,`contactName`,`phone`,`address`,`addressDetail`,`gender` |
| `delivery-query-stores` | 外送场景可配送门店 | `addressId`, `beType`(2/6) |
| `now-time-info` | 获取服务器当前时间 | 无 |
| `campaign-calendar` | 当月营销活动日历 | `specifiedDate`(选填, yyyy-MM-dd) |

### 2.2 菜单与价格（3 个）

| Tool | 用途 | 关键入参 |
|---|---|---|
| `query-meals` | 当前门店在售餐品列表 | `storeCode`,`orderType`,`beType`,`beCode`,`reservationDate` |
| `query-meal-detail` | 餐品详情、套餐组成、特制选项 | 上一步 + `code` |
| `calculate-price` | 计算含优惠后的价格 | `storeCode`,`orderType`,`beType`,`items`,`withOrder`,`needTableware` |

### 2.3 下单与订单（6 个）

| Tool | 用途 | 关键入参 |
|---|---|---|
| `create-order` | 创建点餐订单，返回支付链接 | `storeCode`,`orderType`,`beType`,`items` + 条件必填 |
| `query-order` | 订单详情与配送进度 | `orderId` |
| `cancel-order` | 取消点餐订单 | `orderId`, `cancelReasonCode` |
| `order-list` | 点餐历史订单（非商城） | 无 |
| `query-survey-coupon` | 订单 CSAT 满意度答卷与关联奖券 | `orderId` |
| `query-meal-assistance` | 团餐助餐服务（仅 beType=6） | `storeCode`,`beCode` |

### 2.4 优惠券（4 个）

| Tool | 用途 | 关键入参 |
|---|---|---|
| `available-coupons` | 麦麦省可领券列表 | 无 |
| `auto-bind-coupons` | 一键领取全部可用券 | 无 |
| `query-my-coupons` | 卡包全量券（不校验可用性） | `page`,`pageSize`(≤200, 最多5页) |
| `query-store-coupons` | 指定门店实际可用券 | `storeCode`,`orderType`,`beType`,`beCode` |

### 2.5 营养（1 个）

| Tool | 用途 | 关键入参 |
|---|---|---|
| `list-nutrition-foods` | 常见餐品营养成分（能量/蛋白质/脂肪/碳水/钠/钙） | 无 |

### 2.6 积分与商城（7 个）

| Tool | 用途 | 关键入参 |
|---|---|---|
| `query-my-account` | 积分账户（可用/累计/冻结/过期） | 无 |
| `mall-points-products` | 积分可兑换商品列表 | `catRuleIds`(选填) |
| `mall-product-detail` | 商品详情与 SKU 规格 | `spuId` |
| `mall-create-order` | 积分兑换下单（shopId=2） | `skuId`,`spuCategory`,`addressId?`,`count?` |
| `mall-order-list` | 商城订单列表 | `lastId`,`size` |
| `mall-order-detail` | 商城订单详情 | `orderId` |
| `query-promotions` | 团餐满减/满折规则（仅 beType=6） | `storeCode`,`beCode`,`orderType`,`beType` |

### 2.7 抽奖与奖品（3 个）

| Tool | 用途 | 关键入参 |
|---|---|---|
| `query-lottery-info` | 抽奖活动信息与本次消耗决策 | 无 |
| `draw-lottery` | 执行一次抽奖 | 无 |
| `query-my-prizes` | 中奖记录与状态 | `pageNum`,`pageSize`(≤50) |

### 2.8 主题活动（5 个）

| Tool | 用途 | 关键入参 |
|---|---|---|
| `query-party-city` | 活动可参与城市 | `spuId` |
| `query-party-store` | 城市下可参与门店 | `spuId`,`code`,`latitude`,`longitude` |
| `query-party-store-date` | 门店可预约日期 | `spuId`,`storeCode` |
| `query-party-store-session` | 指定日期场次与余位 | `spuId`,`storeCode`,`dateStr` |
| `party-order-create` | 创建活动订单（shopId=5） | `spuId`,`skuId`,`partyType`,`code`,`storeCode`,`dateStr`,`id`,`timeStart`,`timeEnd`,`leftNum`,`count` |

> 合计 35 个 Tool。所有 Tool 均为麦当劳 MCP Server 原生提供，Skill 未对任何接口做二次封装或模拟。

## 三、调用流程

### 3.1 核心链路：下单

```
用户意图识别
  ↓
确定取餐方式 beType（必须先问清，不可推断）
  ↓
┌─────────────┬──────────────────────────┐
│ beType=1/5  │ beType=2/6               │
│ 到店/得来速  │ 外送/团餐                 │
├─────────────┼──────────────────────────┤
│ query-      │ delivery-query-          │
│ nearby-     │ addresses                 │
│ stores      │   ↓ 无地址则 create-address│
│             │ delivery-query-stores     │
└─────────────┴──────────────────────────┘
  ↓  得到 storeCode + beCode
query-store-coupons（该店可用券）
  ↓
query-meals（在售餐品）
  ↓  用户选定
query-meal-detail（如需改套餐组成或特制）
  ↓
query-meal-assistance（仅 beType=6，取 gmServiceCode）
  ↓
calculate-price（试算，含券）
  ↓  取 takeWayCode
【复述门店/餐品/券/金额/时间 → 用户确认】
  ↓
create-order
  ↓
返回订单号与支付链接 → 用户支付 → query-order 查状态
```

### 3.2 参数矩阵（最高频出错点）

每次调用菜单、算价、券、下单工具前，必须按下表确定三个参数：

| beType | 含义 | orderType | beCode | 门店来源 |
|---|---|---|---|---|
| 1 | 到店自取 | `1` | **不传**（传入报错） | `query-nearby-stores` |
| 5 | 得来速车道取餐 | `1` | **必传** | `query-nearby-stores` |
| 2 | 麦乐送外送 | `2` | **必传** | `delivery-query-stores` |
| 6 | 企业团餐 | `2` | **必传** | `delivery-query-stores` |

条件必填项：
- `orderType=1` 下单时必传 `takeWayCode`，取自 `calculate-price` 返回的 `data.takeWayList[].code`
- `beType=6` 下单时必传 `gmServiceCode`，取自 `query-meal-assistance`
- 预约场景所有工具加传 `reservationDate`（`yyyy-MM-dd HH:mm`），非预约一律不传

### 3.3 券的查询顺序

```
用户问「有什么券可用」
  ↓
已确定门店？──否→ 先定位门店
  ↓ 是
query-store-coupons  ← 下单场景首选，真实校验门店与配送方式
query-my-coupons     ← 仅当用户想看卡包全量时使用
```

`query-my-coupons` 不做门店、渠道、配送方式校验，**不能承诺该券可用于当前订单**。

### 3.4 积分兑换的路由

```
mall-points-products（可选 catRuleIds 筛选）
  ↓
mall-product-detail(spuId) → 用户选 skuId
  ↓
商品是否需要选门店与场次？
  ├─ 否（商品券 spuCategory=1 / 实物 spuCategory=2）
  │    → mall-create-order（仅 shopId=2）
  │      实物需传 addressId；多商品必须串行排队，逐个下单
  └─ 是（派对/体验营/品鉴会/读书会/兑换活动）
       → query-party-city → -store → -store-date → -store-session
         → party-order-create（仅 shopId=5）
```

### 3.5 抽奖的决策链

```
query-lottery-info
  ↓
展示 drawDecision.nextConsumption（本次实际消耗）
  ↓
获得用户明确确认（"试试看""如果可以就抽"不算确认）
  ↓
resourceEligible == true ？──否→ 不得调用
  ↓ 是
draw-lottery
  ↓
如实展示 status.message 与中奖结果
```

一律以 `drawDecision.resourceEligible` / `nextConsumption` 为唯一依据，禁止根据 `availableTimes` 或 `drawTypeText` 自行推断。严禁试抽、自动抽、连续抽。

### 3.6 特制（modification）的 key 传参

用户改配料或换套餐组成时，先 `query-meal-detail` 取 `modification`：

- 用户**选中**的特调项 → `key` = `selectedKey`
- 用户**未选中**的特调项 → 若 `unselectedKey` 非空，`key` = `unselectedKey`，**且必须传入**

即：含 `unselectedKey` 的特调组必须整组传入。

## 四、业务价值

### 4.1 解决的真问题

| 痛点 | 本项目的处理方式 |
|---|---|
| MCP 有 35 个工具，但参数组合复杂，裸调用极易出错 | 将 `beType`/`orderType`/`beCode` 矩阵固化为 Skill 规则，模型无需每次重新推导 |
| 价格单位为「分」，直接展示会差 100 倍 | 明确要求除以 100 并复述确认 |
| 写操作误触风险 | 一键领券、下单、兑换、抽奖、取消订单、创建地址，全部强制复述+确认 |
| 营养数据脏 | 明确标注异常项，要求模型只给参考区间并提示以门店实际为准，不编造合理值 |
| 抽奖规则易误判 | 锁定唯一判定依据，禁止推断与试抽 |

### 4.2 真实使用情况

- 开发过程中实际调用麦当劳 MCP 完成：门店查询、菜单拉取、算价、营养数据全量拉取、工具元数据核对。
- 对 `list-nutrition-foods` 返回的 160 条数据做了实测校验，发现并记录：2 组重复条目、大杯鲜萃咖啡（54kJ）低于小杯（63kJ）的倒挂数值、「纯悦」全字段为 0、无糖可乐热量 0 但碳水为 1 等异常。这些校验结论已固化为 Skill 中的数据可信度章节。
- 逐个核对了 35 个 Tool 的入参约束，发现官方 README 中 `query-partystore-date` / `query-partystore-session` 的写法与实际暴露名不一致（实际为 `query-party-store-date` / `query-party-store-session`），已在 Skill 中标注。

### 4.3 可复用性

Skill 采用「通用铁律 + 参数矩阵 + 分场景流程 + 展示规范」的分层结构。参数矩阵与特制 key 规则是与具体餐厅无关的通用约束，可直接迁移到任何接入同一 MCP Server 的 Agent；场景流程按业务域独立组织，增删某一业务域不影响其它章节。

## 五、错误处理

| code | 原因 | 处理 |
|---|---|---|
| 401 | Token 无效、过期或未提供 | 提示重新配置 MCP 连接，不重试 |
| 429 | 超过 600 次/分钟 | 降低调用频率，稍后重试 |

任何工具失败时，如实告知原因并建议改用麦当劳官方 App 或小程序，**不得用记忆中的数据替代真实查询结果**。

## 六、免责声明

本项目为麦当劳程序员节创意开发大赛参赛作品，由参赛者独立开发，非麦当劳官方产品。项目输出仅供参考，不构成医疗、营养或其他专业建议；餐品信息、价格及供应状态以麦当劳官方渠道的实时结果为准。使用麦当劳 MCP 服务须遵守麦当劳中国的《使用条款》及《麦当劳 MCP 服务规则》。
