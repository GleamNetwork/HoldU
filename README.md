# HoldU · 同频 B2

用温暖陪伴降低求助门槛，用可追溯的观察和清楚的责任交接支持人工服务。

同频围绕用户端、志愿者端和管理端设计：小频日常陪伴、早晚问候、低负担观察与可选量表，结合授权手表数据，再根据明确证据核实需要并连接合适的真人或专业资源。

**当前是黑客松产品设计与演示原型，不是医疗器械、诊断工具、真实分诊引擎或全天候急救服务。** 无真实患者或志愿者档案。R0–R3 为本产品的响应路径，不是国际统一风险分层。

![小频](assets/xiaopin/wave.png)

## 文档入口

| 想了解什么 | 从这里开始 |
| --- | --- |
| 完整产品需求、三端分工、排班与交接 | [PRD V0.4](docs/product/PRD-v0.4.md) |
| 用户/志愿者年龄、兴趣、MBTI 与服务匹配 | [双端 Profile](docs/product/profiles-and-matching.md) |
| 每项数据怎么来、怎么用、谁能看 | [45 项数据字典](docs/rules/data-dictionary-v0.4.md) |
| 单项或组合达到什么条件才触发 | [47 条规则与逐条依据](docs/rules/trigger-rules-v0.4.md) |
| 信息如何流向核实、响应和人工接续 | [九张 Mermaid 流程图](docs/rules/seven-module-flowcharts-v0.4.md) |
| 筛选对照表 | [Excel：Sheet1 数据流 / Sheet2 条件](docs/rules/data-flow-and-triggers-2026-09-09.xlsx) |
| 临床依据和产品草案的区别 | [参考来源](docs/rules/evidence-sources.md) |
| 小频、暖色、未成年体验与跟练 | [小频设计](docs/design/xiaopin.md) |
| 手表情绪陪伴版与运动健康版概念原图 | [手表设计](assets/watch/README.md) |
| 双击打开原型 / 手表接入现状 | [原型指南](docs/prototypes/README.md) |
| 国家、国际、上海政策和 GitHub 项目 | [研究索引](research/README.md) |
| 旧版 PRD、Word 和历史 HTML | [历史档案](archive/README.md) |
| 本轮差异 | [变更记录](docs/CHANGELOG.md) |

## 原型与真实能力

下载完整仓库后可打开 [离线原型](prototypes/tongpin-demo.html)。既有 [在线 Demo](https://tongpin-b2-companion.ruomengbi.chatgpt.site/)本轮未改动。演示中固定脚本、设备数据、人员与接单状态均不代表真实服务，V0.4 文档也不等于功能已上线。

## 规则使用

日常话语先形成观察；只有规范有效的直接答案才能成为正式量表条目。缺失不补零，设备或模型估计不冒充答案。只有同一量表内按规定计分，跨模块使用逐条 AND/OR，不生成综合风险总分。MBTI、颜色和爱好只作可选偏好。

睡眠、步数等产品草案保持未启用。实际部署需要机构、专业与隐私审查，并建立真实接续渠道。当前安全及身体急症不等待量表、资料填写或匹配名额。未接收不能写“已转诊”。

## 版本与分发

当前文字主稿是 V0.4；Excel 原样保留，V0.3.x HTML/Word 明确归档。源文件未移动或覆盖。目录不包含登录截图、账号信息、模型权重、缓存或未核验再分发许可的第三方全文。素材权利边界见小频设计；未设置覆盖第三方资料或角色的开源许可证。
