# ☕ 瑞幸咖啡全天候助手（lk-coffee-all-weather-assistant）

> 让智能体帮你**查瑞幸、盯价格、到点下单**——纯对话驱动，不写一行脚本。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Protocol: MCP](https://img.shields.io/badge/Protocol-MCP-blue.svg)](https://open.lkcoffee.com/mcp)

---

## 这是什么

`lk-coffee-all-weather-assistant` 是一个 **Agent Skill（智能体技能）**，由「[SKILL.md](./SKILL.md) + 宿主平台能力」组成：

- **纯对话 + MCP 工具驱动**：不依赖任何本地脚本，也不绑定特定平台；
- **直连瑞幸官方 MCP Server**（`gwmcp.lkcoffee.com`）：智能体在对话中直接调用 8 个官方工具取数；
- **面向小白用户**：从申请 Token 到配置连接器、再到落定时任务，全程手把手引导；
- **可被组合调用**：任何技能识别到「查瑞幸 / 订瑞幸 / 点瑞幸」的意图，都可路由到本 skill。

运行本 skill 只需宿主平台具备两项能力：

1. 能连接瑞幸官方 MCP Server（Streamable HTTP 连接器）；
2. 具备定时任务（Scheduler / Cron / 自动化任务）能力。

例如：QoderWork、WorkBuddy、Hermes 等。

## 核心能力

| 场景 | 你可以这样说 | 智能体会做 |
| --- | --- | --- |
| 引导式配置 | 「帮我开通瑞幸助手」 | 手把手带你完成 Token 申请 → MCP 配置 → 验证 |
| 常用门店 | 「记一下我的常用门店：望京 SOHO T1 店」 | 智能体侧记住门店与坐标；之后说「老地方」直接复用（瑞幸 MCP 无门店配置接口） |
| 即时查询 | 「公司附近有哪些瑞幸？生椰拿铁多少钱？」 | 查门店、搜商品、报官方实时到手价 |
| 即时下单 | 「帮我下杯小杯冰美式」 | 先问下单位置 → 列出最近 3 家门店（没满意的可报店名 / 换位置）→ 核价 → 创建订单 → 推送付款二维码链接 |
| 价格订阅 | 「生椰拿铁涨过 20 块就告诉我」 | 建定时任务，按你设定的频率比价，触发阈值才通知 |
| 定时点餐 | 「工作日早上 8 点来杯美式，超 11 块就别下」 | 到点核价下单、推送付款入口；**价格超你设的上限则禁止下单**（绝不代付） |

具体场景细节与执行规范见 [SKILL.md](./SKILL.md)。

## 快速开始

### 第 1 步：申请 Token

1. 打开瑞幸开放平台：<https://open.lkcoffee.com/mcp>
2. 点击「立即领取」或「控制台」；
3. 用手机号 + 验证码登录（可能有滑块人机验证）；
4. 复制页面上的那一串 Token（**整串复制全**），妥善保存。

> 提示 1：瑞幸 Token 不是常见的 32 位串——**完整 Token 以 `mcpLUCKIN_MCP_AI` 结尾**，
> 复制 / 粘贴时请勿截断（截断会直接 401）。
> 提示 2：Token 有一定时效。若后续提示 401，先核对是否复制完整，再重新领取并替换配置即可。

### 第 2 步：配置 MCP 连接器

在宿主平台的连接器 / MCP 设置中，新增一个名为 `my-coffee` 的 MCP Server（名称可任意；把 Token 换成你自己的），可参考 [templates/mcp-config.example.json](./templates/mcp-config.example.json)：

```json
{
  "mcpServers": {
    "my-coffee": {
      "type": "streamablehttp",
      "url": "https://gwmcp.lkcoffee.com/order/user/mcp",
      "headers": {
        "Authorization": "Bearer <你的 Token>"
      }
    }
  }
}
```

> 说明：瑞幸官网按域名派生环境——生产 `gwmcp.lkcoffee.com`、预发 `mcpgatewaypre.lkcoffee.com`、测试 `mcpgatewaytest03.lkcoffee.com`，路径均为 `/order/user/mcp`。本项目默认对接生产环境。

### 第 3 步：安装本 Skill

把本仓库加入宿主平台的技能目录，或通过平台的「导入 Skill / 技能市场」能力导入：

- 仓库根目录即技能目录，入口为 [SKILL.md](./SKILL.md)；
- 具体安装路径因平台而异，以平台文档为准。

### 第 4 步：开始对话

直接对智能体说一句即可，例如：

```text
根据 https://github.com/Little-cao/lk-coffee-all-weather-assistant 这个地址，帮我安装这个 skill，我在望京 SOHO，帮我配置每天 9:30 点一杯生椰拿铁
```

更多可直接复制的指令见 [examples/prompts.md](./examples/prompts.md)。

## 支持的 MCP 工具

| 工具 | 用途 | 关键参数 |
| --- | --- | --- |
| `queryShopList` | 按经纬度 / 门店名找门店（点单时优先给最近 3 家） | `longitude`、`latitude`、`deptName?` |
| `searchProductForMcp` | 按口语搜索商品 | `deptId`、`query` |
| `queryProductDetailInfo` | 商品详情与规格组 | `deptId`、`productId` |
| `switchProduct` | 切换大小杯 / 冰热 / 加料 | `deptId`、`productId`、`skuCode`… |
| `previewOrder` | 下单前核价 | `deptId`、`productList` |
| `createOrder` | 创建订单（返回付款二维码 URL） | `deptId`、`productList`、`longitude`、`latitude` |
| `queryOrderDetailInfo` | 查订单状态 | `orderId` |
| `cancelOrder` | 取消未支付订单 | `orderId` |

完整参数说明见 [SKILL.md](./SKILL.md) 的「能力与对应 MCP 工具」章节。

## 明确不支持的场景（如实告知，绝不假装）

| 场景 | 原因 |
| --- | --- |
| 自动领券 / 券包 / 活动日历 | 瑞幸官方 MCP **未开放优惠券相关接口**；瑞幸的折扣已自动包含在「到手价」中 |
| 外送到家 / 送公司 | 瑞幸 MCP 当前未暴露外送接口，本 skill 默认**到店自提** |
| 商品评价、门店专属促销 | 瑞幸 MCP 未开放 |
| 代付 / 自动扣款 | 红线：只生成订单与付款入口，付款必须由本人完成 |

## 安全与合规

- **凭证不落库**：Token 仅用于当次配置，不写入任何文件、不回显完整内容，长期由宿主平台连接器统一管理；
- **绝不自动付款（红线）**：只到「生成订单 + 给出付款入口（`payOrderQrCodeUrl`）」为止；
- **超上限禁止下单**：定时点餐的「价格上限」由你输入具体数值（如「超 10.9 就别下」）；触发时当前价超上限即禁止下单、只发通知；
- **商品下架不擅自替换**：只发通知，绝不自作主张换其他商品；
- **通知渠道必须经用户提供并确认**，不臆造收件地址；
- **一切以瑞幸官方实时结果为准**。

## 项目结构

```text
lk-coffee-all-weather-assistant/
├── SKILL.md                        # 技能核心定义（智能体读取入口）
├── README.md                       # 项目说明
├── CHANGELOG.md                    # 更新记录
├── CONTRIBUTING.md                 # 贡献指南
├── LICENSE                         # MIT 许可证
├── examples/
│   └── prompts.md                  # 可直接复制的对话指令示例
└── templates/
    └── mcp-config.example.json     # MCP 连接器配置模板
```

## 常见问题

### 「钥匙」（Token）是什么？安全吗？

它是瑞幸官方开放平台发放的访问凭证，相当于一张会员卡：只用于查询门店、商品、下单，不会动到你的钱、也不会自动下任何单。本 skill 只把它用于当次配置，不存储、不外发。

### 提示 401 / 钥匙过期怎么办？

Token 有一定时效；**也请先检查是否复制完整**——瑞幸 Token 以 `mcpLUCKIN_MCP_AI` 结尾，容易漏掉尾巴（截断同样会 401）。确认无误后回到 <https://open.lkcoffee.com/mcp> 重新领取，替换连接器配置中的 Token 即可。

### 为什么不能「自动领券」？

瑞幸官方 MCP 目前只开放门店 / 商品 / 下单 / 订单四类能力，未提供优惠券接口。好消息是瑞幸的折扣本来就自动算进「到手价」，无需手动领券。想盯价格可以改用价格监控订阅（见 SKILL.md 场景 A）。

### 支持外送到公司吗？

瑞幸 MCP 当前未明确暴露外送接口，本 skill 默认到店自提，不做外送承诺。

## 免责声明

- 本项目为**社区作品**，非瑞幸咖啡官方产品，与瑞幸官方无隶属关系；
- 所有门店、商品、价格、订单信息均以**瑞幸官方 MCP 实时返回结果为准**；
- 使用本 skill 产生的任何订单与支付行为，均由用户本人在官方渠道完成并自行承担。

## License

[MIT](./LICENSE) © 2026 lk-coffee-all-weather-assistant contributors
