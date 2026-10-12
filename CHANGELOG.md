# 更新记录

本项目所有重要变更都记录在此文件。
格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [0.1.0] - 2026-10-12

### 新增

- 初始版本发布，包含完整的 `SKILL.md` 技能定义；
- 支持四大场景：引导式配置、即时查询 / 下单、价格监控订阅（场景 A）、定时点餐（场景 E）；
- 支持瑞幸官方 MCP 8 个工具：`queryShopList`、`searchProductForMcp`、`switchProduct`、`queryProductDetailInfo`、`previewOrder`、`createOrder`、`queryOrderDetailInfo`、`cancelOrder`；
- 明确合规红线：不支持自动领券、绝不代付、凭证不落库、默认到店自提、门店 / 商品异常不硬凑结果；
- 补充项目配套：README、示例指令（`examples/prompts.md`）、MCP 配置模板（`templates/mcp-config.example.json`）、贡献指南。
