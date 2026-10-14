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

### 3.7 个性化营养推荐的调用链

健康类诉求（减脂/增肌/控糖/控钠/代谢异常）**必须先完成个体信息采集，再进入数据查询**：

```
用户表达健康类诉求
  ↓
【强制询问】健康状况与忌口 / 目标类型 / 进食场景 / 身体数据(可选) / 口味接受度
  ↓  用户拒绝询问 → 标注为「通用方案」，不得称「为你定制」
query-my-account（可选，确认是否有可用积分兑换更优方案）
  ↓
list-nutrition-foods（拉取全量营养表）
  ↓
按个体情况**先排除**不适宜品项（高尿酸避高嘌呤、控糖避含糖、控钠避腌渍与高钠酱料…）
  ↓
按热量 / 蛋白质 / 钠三维度排序候选
  ↓
query-meals（storeCode）→ **在现售商品表中核验每个候选品项可点且有currentPrice**
  ↓  套餐子项编码不可当作独立单品，须剔除或说明「仅存在于套餐内」
校验饮品数量：单人份上限 1 种（奶饮计入）
  ↓
calculate-price（可用时实算；不可用时按 currentPrice 累加并标注「非接口实算」）
  ↓
输出推荐：热量 + 蛋白质 + 钠 + 价格 + 适用性说明
  ↓
提示「实际以门店出品为准」；涉及代谢疾病须提示以医生或营养师意见为准
```

**关键约束**：
- 营养表**不含嘌呤字段**，高尿酸等代谢指标只能按食材常识判断，**不得声称具体嘌呤含量**。
- 套餐热量须按 `query-meal-detail` 返回的 `rounds[].choices[]` 逐项相加，不可凭套餐名臆测。
- 正餐场景不得给出 400kcal 以下方案，避免掉肌肉与报复性进食。

## 四、业务价值

### 4.1 解决的真问题

| 痛点 | 本项目的处理方式 |
|---|---|
| MCP 有 35 个工具，但参数组合复杂，裸调用极易出错 | 将 `beType`/`orderType`/`beCode` 矩阵固化为 Skill 规则，模型无需每次重新推导 |
| 价格单位为「分」，直接展示会差 100 倍 | 明确要求除以 100 并复述确认 |
| 写操作误触风险 | 一键领券、下单、兑换、抽奖、取消订单、创建地址，全部强制复述+确认 |
| 营养数据脏 | 明确标注异常项，要求模型只给参考区间并提示以门店实际为准，不编造合理值 |
| 抽奖规则易误判 | 锁定唯一判定依据，禁止推断与试抽 |
| 宽泛推荐对个体无效，甚至有害 | 健康类诉求强制先采集健康状况/忌口/进食场景，按个体情况先排除后推荐 |
| 只看热量会漏掉钠| 强制三维度（热量/蛋白质/钠）并列，拒绝单报热量 |
| 套餐子项与独立单品混淆 | 规定可点性校验：套餐 `choices[].code` 不可单点，查不到即说明，禁止编造价格 |
| 营养密度与减脂目标冲突 | 单人份饮品上限 1 种（含奶饮），避免为「凑营养」叠加饮品 |

### 4.2 真实使用情况

- 开发过程中实际调用麦当劳 MCP 完成：门店查询、菜单拉取、算价、营养数据全量拉取、工具元数据核对。
- 对 `list-nutrition-foods` 返回的 160 条数据做了实测校验，发现并记录：2 组重复条目、大杯鲜萃咖啡（54kJ）低于小杯（63kJ）的倒挂数值、「纯悦」全字段为 0、无糖可乐热量 0 但碳水为 1 等异常。这些校验结论已固化为 Skill 中的数据可信度章节。
- 逐个核对了 35 个 Tool 的入参约束，发现官方 README 中 `query-partystore-date` / `query-partystore-session` 的写法与实际暴露名不一致（实际为 `query-party-store-date` / `query-party-store-session`），已在 Skill 中标注。
- **真实下单链路实测受阻并定位根因**：在完整业务流程演练中，`create-order` 的 `items` 参数连续三次被服务端校验拒绝（`/items: must be array`），包括退化为最简数组的极端情况。对比同结构下 `calculate-price` 可正常调用，定位为 **MCP Server 侧 schema 缺陷**（内层对象缺少 `"type": "object"` 声明）。该缺陷后续在 `calculate-price` 上也出现过，判定为**间歇性问题**。
  - 已据此在 Skill 中固化降级规则：**一次失败即降级，不反复重试**，改由 `query-meals` 的 `currentPrice` 手工核算并明确标注「非接口实算」，同时引导用户走官方 App / 小程序。
- **实测暴露并修正了自身的两处错误**（均由用户追问发现）：
  1. 首轮推荐只报热量、漏掉钠维度——吉士汉堡包套餐 347kcal 但钠 709mg，而麦乐鸡组合约 223kcal、钠仅 373mg，后者双维更优。已固化为「三维度并列」的硬性要求。
  2. 曾将套餐子项 `3071`（无糖可口可乐中杯）误当作可单点商品并**编造了价格**。已新增可点性校验章节，明确套餐子项不可单点、查不到须如实说明、禁止编价。

### 4.3 差异化定位

赛事已有 200+ 参赛项目（截至 2026-10-14 官方 RANKING.md 前 100 名分布：第 1 名 303 Star、第 100 名 8 Star，入榜门槛 8 Star）。其中**营养/卡路里类不少于 10 个**：`mcd-calorie-butler`(149★)、`mcd-order-calories`(35★)、`mcd-calorie-budget-skill`(22★)、`mcd-nutrition-coach`(8★)、`mcd-100kcal`(16★)、`mcd-solo-meal`(10★) 等。

同质化现状：多数项目把 MCP 当**数据源**，核心价值在「取数 → 计算 → 输出」的速度与准确度，典型做法是用 Python 做确定性求解（券组合优化、卡路里约束求解）。

**本项目的差异点**：把 MCP 当**能力源**，核心价值在**决策前的信息完备性**——

| 差异维度 | 常见做法 | 本项目 |
|---|---|---|
| 营养推荐的信息源 | 直接取营养表出推荐 | **强制先问 5 组个人情况**，问完才推荐 |
| 指标维度 | 热量为主 | **热量/蛋白质/钠三维度强制并列**（钠是被普遍漏掉的一维） |
| 代谢异常人群 | 一般不区分 | 高血压/高尿酸/控糖/乳糖不耐等**先排除品项再推荐** |
| 进食场景 | 不区分 | 正餐 / 加餐 / 唯一进食，热量上限不同 |
| 无法完成时 | 模糊带过或假装成功 | **如实告知接口不可用 + 降级路径** |
| 规则来源 | 设计推导 | 记录三处被用户实测抓出的错误并固化修正 |

**一句话**：别人算「这份套餐多少卡」，本项目先问「这份套餐是给谁吃的」。

### 4.4 可复用性

Skill 采用「通用铁律 + 参数矩阵 + 分场景流程 + 展示规范」的分层结构。参数矩阵与特制 key 规则是与具体餐厅无关的通用约束，可直接迁移到任何接入同一 MCP Server 的 Agent；场景流程按业务域独立组织，增删某一业务域不影响其它章节。

## 五、错误处理

| code | 原因 | 处理 |
|---|---|---|
| 401 | Token 无效、过期或未提供 | 提示重新配置 MCP 连接，不重试 |
| 429 | 超过 600 次/分钟 | 降低调用频率，稍后重试 |

任何工具失败时，如实告知原因并建议改用麦当劳官方 App 或小程序，**不得用记忆中的数据替代真实查询结果**。

### 5.1 实测发现的接口缺陷：`items` 参数校验失败

**现象**：调用时返回参数校验错误 `/items: must be array`，与实际传入的数组结构无关。

**实测记录**：

| 接口 | 尝试结构 | 结果 |
|---|---|---|
| `create-order` | 完整套餐结构（`roundList` 嵌套 `comboItemList`） | 失败 |
| `create-order` | `roundList` 传为双层数组 | 失败 |
| `create-order` | 退化为最简元素数组 `[{productCode, quantity}]` | 失败 |
| `create-order` | 修正外层括号后重试 | 失败 |
| `calculate-price` | 完整套餐结构（多次） | **成功** |
| `calculate-price` | 单品数组 | 失败（同一时段） |

**根因判定**：MCP Server 侧 schema 缺陷。`create-order` 的 `items` 内层对象**缺少 `"type": "object"` 声明**（仅有 `properties`），而结构相近的 `calculate-price` 有该声明。同结构下 `calculate-price` 可正常调用，可排除业务策略限制——**该 MCP 的下单设计本就不接管资金流**（仅创建订单并返回支付链接，支付由用户在 APP 完成），不存在被限制的支付接口。

**该缺陷为间歇性**：同日`create-order` 三次全部失败，而 `calculate-price` 先后出现成功与失败，疑与不同时段的服务实例有关。

**降级策略**（已写入 Skill）：
1. **一次失败即降级，禁止反复重试或变换参数结构**——schema 缺陷非调用方式可解。
2. 改由 `query-meals` 返回的 `currentPrice` 逐项累加估算，并**明确告知用户「此为按菜单现价估算，非接口实算」**。
3. 引导用户改用麦当劳官方 App / 小程序完成下单。
4. **未成功创建订单时不得宣称下单成功。**

## 六、免责声明

本项目为麦当劳程序员节创意开发大赛参赛作品，由参赛者独立开发，非麦当劳官方产品。项目输出仅供参考，不构成医疗、营养或其他专业建议；餐品信息、价格及供应状态以麦当劳官方渠道的实时结果为准。使用麦当劳 MCP 服务须遵守麦当劳中国的《使用条款》及《麦当劳 MCP 服务规则》。
