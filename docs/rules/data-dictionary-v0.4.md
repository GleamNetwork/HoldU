# 七模块与双端 Profile 数据字典

版本：V0.4 · 2026-09-09。与双端 Profile Excel 同一规则清单。全部为产品设计与虚构情境，不是诊断、真实患者数据或已部署服务。

共 45 项。Sheet1 每行与以下同编号记录对应。下游规则由规则的 inputs 反向关联，避免简称混淆。

**权限勘误：** Excel 的志愿者 Profile 使用了通用“本人查看修改/非必要项可跳过”模板，不能据此修改核验、排班计数或权限。以下保留原文以便对表，同时逐项列出“实施权限澄清”；应优先按澄清及原始 entry 实施。此处不声称旧 Excel 模板已修复。

## 情绪体验

<a id="dat-emo-01"></a>

### DAT-EMO-01｜心情、焦虑、易怒、绝望等主观体验

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 情绪体验 |
| 来源/入口 | 早晚主动关怀；文字/主动语音；可选点选 |
| 字段与单位 | 情绪主题enum[]；强度0–10分（可选）；原句；对象；发生时间 |
| 时间窗 | 日常问候由用户调整；自评记录当前，不能代表两周 |
| 质量、授权与缺失（Excel 原文） | 语音无应答可切换点选，也可跳过；普通未回复不判定异常。强度须本人选择，不从语气估分。 |
| AI 整理与本人确认（Excel 原文） | 转写、区分对象/否定/引用，提取本人明确表达；歧义才澄清 |
| 观察/量表边界 | 日常观察，0–10无已批准转诊切点 |
| 下游规则 | [TRG-EMO-K01](trigger-rules-v0.4.md#trg-emo-k01)；[TRG-X04](trigger-rules-v0.4.md#trg-x04) |
| 用途 | 观察记录；TRG-EMO-K01/K02；若当前安全表达则走SAF规则 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [DEP：NICE NG222. Depression in adults: treatment and management. 2022](https://www.nice.org.uk/guidance/ng222/chapter/Recommendations)；[AI：WHO. Ethics and governance guidance for large multi-modal models. 2024](https://www.who.int/news/item/18-01-2024-who-releases-ai-ethics-and-governance-guidance-for-large-multi-modal-models) |
| 支持边界 | 指南支持询问症状，不支持每天固定频次或强度分自动转诊。 |

<a id="dat-emo-02"></a>

### DAT-EMO-02｜低落与兴趣减少的频率、持续时间

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 情绪体验 |
| 来源/入口 | 自然对话；必要时补问时间/频率 |
| 字段与单位 | 主题；回顾起止日期；发生天数或本人频率档；功能例子 |
| 时间窗 | 正式PHQ适用回顾窗为过去2周；自然话语保持原时间窗 |
| 质量、授权与缺失（Excel 原文） | 取得适用授权。保留来源、时间、质量与修订记录。缺失不补0。 |
| AI 整理与本人确认（Excel 原文） | 明确答案直接复用；“没劲”先区分疲乏/兴趣/心情；不同事实不重复计分 |
| 观察/量表边界 | 可生成PHQ条目主题候选；未完成规范施测不计正式分 |
| 下游规则 | [TRG-EMO-K02](trigger-rules-v0.4.md#trg-emo-k02)；[TRG-X01](trigger-rules-v0.4.md#trg-x01) |
| 用途 | TRG-EMO-Q01-A/Q02-A；TRG-X01；候选不得直接触发分数规则 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [PHQ9：Kroenke等. The PHQ-9. J Gen Intern Med. 2001;16:606–613](https://pubmed.ncbi.nlm.nih.gov/11556941/)；[HOPE：Guo等. HopeBot对话式PHQ-9施测研究. 预印本，2026更新](https://arxiv.org/abs/2507.05984) |
| 支持边界 | 对话式施测需要版本与等效性核验；研究不验证同频。 |

<a id="dat-emo-03"></a>

### DAT-EMO-03｜PHQ-2/PHQ-9正式答案

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 情绪体验 |
| 来源/入口 | 自愿规范施测；可用核验版口头回答 |
| 字段与单位 | 工具版本；语言；item_id；每题0–3分；完整性；施测日期 |
| 时间窗 | 过去2周；复测按服务计划，不每天重复长表 |
| 质量、授权与缺失（Excel 原文） | 完整、有效、适龄。缺题不生成完整总分。安全题不等总表。 |
| AI 整理与本人确认（Excel 原文） | AI仅整理，固定程序计分；不能自由补答案 |
| 观察/量表边界 | PHQ-2总分0–6；PHQ-9总分0–27 |
| 下游规则 | [TRG-EMO-Q01-A](trigger-rules-v0.4.md#trg-emo-q01-a)；[TRG-EMO-Q02-A](trigger-rules-v0.4.md#trg-emo-q02-a)；[TRG-X07](trigger-rules-v0.4.md#trg-x07) |
| 用途 | TRG-EMO-Q01-A/Q02-A；第9题独立关联TRG-SAF-Q01 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [PHQ9：Kroenke等. The PHQ-9. J Gen Intern Med. 2001;16:606–613](https://pubmed.ncbi.nlm.nih.gov/11556941/)；[PHQ2：Kroenke等. PHQ-2验证研究. Medical Care. 2003;41:1284–1292](https://pubmed.ncbi.nlm.nih.gov/14583691/) |
| 支持边界 | 切点来自研究，不等于诊断；需要专业评估。 |

<a id="dat-emo-04"></a>

### DAT-EMO-04｜GAD-2/GAD-7正式答案

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 情绪体验 |
| 来源/入口 | 自愿规范施测；核验版对话流程 |
| 字段与单位 | 版本；语言；item_id；每题0–3分；完整性 |
| 时间窗 | 过去2周 |
| 质量、授权与缺失（Excel 原文） | 只使用适龄版本；缺题不计完整总分；允许拒答 |
| AI 整理与本人确认（Excel 原文） | 仅记录直接答案与来源；固定求和，不与PHQ相加 |
| 观察/量表边界 | GAD-2总分0–6；GAD-7总分0–21 |
| 下游规则 | [TRG-EMO-Q01-B](trigger-rules-v0.4.md#trg-emo-q01-b)；[TRG-EMO-Q02-B](trigger-rules-v0.4.md#trg-emo-q02-b) |
| 用途 | TRG-EMO-Q01-B/Q02-B |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [GAD2：华盛顿大学 National HIV Curriculum. GAD-2](https://www.hiv.uw.edu/page/mental-health-screening/gad-2)；[GAD7：Spitzer等. A brief measure for assessing generalized anxiety disorder: the GAD-7. 2006](https://pubmed.ncbi.nlm.nih.gov/16717171/) |
| 支持边界 | GAD-7有8与10等研究/教学切点差异，需按目标人群预先选定。 |

## 睡眠与节律

<a id="dat-slp-01"></a>

### DAT-SLP-01｜入睡、起床、夜醒与恢复感

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 睡眠与节律 |
| 来源/入口 | 晨间自然问候；点选/文字/语音 |
| 字段与单位 | 睡眠起止时间；夜醒次数；主观恢复感；午睡/轮班情境 |
| 时间窗 | 昨夜或本人明确时间窗 |
| 质量、授权与缺失（Excel 原文） | 取得适用授权。保留来源、时间、质量与修订记录。缺失不补0。 |
| AI 整理与本人确认（Excel 原文） | 提取时间和症状，跨午夜正确记录；不把卧床时长等同实际睡眠 |
| 观察/量表边界 | 日常睡眠观察；可选量表另行核验，不强配 |
| 下游规则 | [TRG-SLP-K01](trigger-rules-v0.4.md#trg-slp-k01)；[TRG-X01](trigger-rules-v0.4.md#trg-x01) |
| 用途 | TRG-SLP-K01；TRG-X01 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [DEP：NICE NG222. Depression in adults: treatment and management. 2022](https://www.nice.org.uk/guidance/ng222/chapter/Recommendations)；[SLEEP：AASM. Consumer sleep technology: position statement. J Clin Sleep Med. 2018;14:877–880](https://aasm.org/advocacy/position-statements/consumer-sleep-technology/) |
| 支持边界 | 一次少睡不代表两周失眠或疾病。 |

<a id="dat-slp-02"></a>

### DAT-SLP-02｜设备睡眠时长与趋势

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 睡眠与节律 |
| 来源/入口 | 授权手表/健康平台 |
| 字段与单位 | 时长number（小时/分钟）；起止时间；设备/算法版本；佩戴与同步质量 |
| 时间窗 | 每晚；趋势参考窗必须同类日期与足够有效日 |
| 质量、授权与缺失（Excel 原文） | 先质控。更换设备、轮班、缺失、未佩戴不能直接比较。 |
| AI 整理与本人确认（Excel 原文） | 对照自述但不互相覆盖；同一晚只一条观察，保留多个来源 |
| 观察/量表边界 | 设备估计，不是量表分数 |
| 下游规则 | [TRG-SLP-D01](trigger-rules-v0.4.md#trg-slp-d01) |
| 用途 | TRG-SLP-D01（数字草案未启用） |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [SLEEP：AASM. Consumer sleep technology: position statement. J Clin Sleep Med. 2018;14:877–880](https://aasm.org/advocacy/position-statements/consumer-sleep-technology/)；PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09 |
| 支持边界 | 少2小时×2晚以及14天/7有效日均为产品草案，不是AASM阈值。 |

<a id="dat-slp-03"></a>

### DAT-SLP-03｜睡眠需要减少、睡少却不困

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 睡眠与节律 |
| 来源/入口 | 本人/授权照护者现实描述 |
| 字段与单位 | 相较平常是否少睡；是否不困；精力、兴奋/冲动；发生时间 |
| 时间窗 | 当前与近期变化 |
| 质量、授权与缺失（Excel 原文） | 区分熬夜后困倦、失眠和主观不需要睡。观察不能靠手表单独确认。 |
| AI 整理与本人确认（Excel 原文） | 提取直接事实，并核实异常行为与即时危险 |
| 观察/量表边界 | 不由问卷或LLM诊断双相障碍 |
| 下游规则 | [TRG-SLP-K02](trigger-rules-v0.4.md#trg-slp-k02)；[TRG-X03](trigger-rules-v0.4.md#trg-x03)；[TRG-EMO-K03](trigger-rules-v0.4.md#trg-emo-k03) |
| 用途 | TRG-SLP-K02；TRG-X03 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [BIP：NICE CG185. Bipolar disorder: assessment and management. 更新2025-09-02](https://www.nice.org.uk/guidance/cg185/chapter/recommendations) |
| 支持边界 | 指南支持疑似躁狂紧急专科评估，不规定本项目三项AND算法。 |

## 精力与活动

<a id="dat-act-01"></a>

### DAT-ACT-01｜步数、活动分钟与佩戴质量

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 精力与活动 |
| 来源/入口 | 授权手表/手机活动数据 |
| 字段与单位 | steps整数（步）；活动分钟；设备来源；有效佩戴；同步时间 |
| 时间窗 | 按有效日；只与可靠个人同类日参考比较 |
| 质量、授权与缺失（Excel 原文） | 同来源去重。未佩戴/缺失不填0；轮椅、行动障碍、住院等改变解释。 |
| AI 整理与本人确认（Excel 原文） | 汇总有效数据并标明限制；不能从低步数推出疾病发作 |
| 观察/量表边界 | 不进入PHQ/GAD评分 |
| 下游规则 | [TRG-ACT-D01](trigger-rules-v0.4.md#trg-act-d01)；[TRG-ACT-D02](trigger-rules-v0.4.md#trg-act-d02)；[TRG-X08](trigger-rules-v0.4.md#trg-x08) |
| 用途 | TRG-ACT-D01/D02；TRG-X08 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[AI：WHO. Ethics and governance guidance for large multi-modal models. 2024](https://www.who.int/news/item/18-01-2024-who-releases-ai-ethics-and-governance-guidance-for-large-multi-modal-models) |
| 支持边界 | 步数<50%×3有效日及基线算法无直接临床标准支持，仅提醒假设。 |

<a id="dat-act-02"></a>

### DAT-ACT-02｜主观精力、疲倦与活动情境

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 精力与活动 |
| 来源/入口 | 日常聊天/点选 |
| 字段与单位 | 精力自述；可选0–10自评分；活动类型；疼痛/运动后状态 |
| 时间窗 | 当前及本人说明的持续时间 |
| 质量、授权与缺失（Excel 原文） | 取得适用授权。保留来源、时间、质量与修订记录。缺失不补0。 |
| AI 整理与本人确认（Excel 原文） | 区分“累”“无聊”“没兴趣”，不一词多次计分 |
| 观察/量表边界 | PHQ精力主题只能在符合施测要求时成为正式答案 |
| 下游规则 | [TRG-SLP-K02](trigger-rules-v0.4.md#trg-slp-k02)；[TRG-ACT-K01](trigger-rules-v0.4.md#trg-act-k01)；[TRG-X02](trigger-rules-v0.4.md#trg-x02)；[TRG-X03](trigger-rules-v0.4.md#trg-x03)；[TRG-EMO-K03](trigger-rules-v0.4.md#trg-emo-k03) |
| 用途 | TRG-ACT-K01；TRG-X02 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [DEP：NICE NG222. Depression in adults: treatment and management. 2022](https://www.nice.org.uk/guidance/ng222/chapter/Recommendations)；[PHQ9：Kroenke等. The PHQ-9. J Gen Intern Med. 2001;16:606–613](https://pubmed.ncbi.nlm.nih.gov/11556941/) |
| 支持边界 | 无已验证的“精力低于几分自动转诊”。 |

<a id="dat-act-03"></a>

### DAT-ACT-03｜久坐、外出与任务完成

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 精力与活动 |
| 来源/入口 | 优先本人自述；可选授权活动记录 |
| 字段与单位 | 久坐估计分钟；是否外出；任务例子；身体限制 |
| 时间窗 | 当前/近期；日常提醒节奏由用户设置 |
| 质量、授权与缺失（Excel 原文） | 久坐设备判定依设备验证；无连续定位需求。 |
| AI 整理与本人确认（Excel 原文） | 只做观察或自愿活动邀请；身体警示时停止运动建议 |
| 观察/量表边界 | 无默认量表 |
| 下游规则 | 无临床规则 |
| 用途 | TRG-X02；身体警示优先BOD；久坐提醒另审 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[DEP：NICE NG222. Depression in adults: treatment and management. 2022](https://www.nice.org.uk/guidance/ng222/chapter/Recommendations) |
| 支持边界 | 没有统一久坐分钟数能判定精神疾病发作。 |

## 对话与认知

<a id="dat-cog-01"></a>

### DAT-COG-01｜语音转写、语句语境

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 对话与认知 |
| 来源/入口 | 用户主动开启的语音会话/文字 |
| 字段与单位 | 转写文本；不清片段；对象；当前/历史；肯定/否定；引用标记 |
| 时间窗 | 本次片段 |
| 质量、授权与缺失（Excel 原文） | 不后台监听；不把口音/低音量当疾病；低置信度不补全关键伤害词。 |
| AI 整理与本人确认（Excel 原文） | 语境识别和候选提取；重要歧义澄清，原句留证 |
| 观察/量表边界 | 转写文本不直接等于正式答案 |
| 下游规则 | [TRG-COG-K03](trigger-rules-v0.4.md#trg-cog-k03)；[TRG-SAF-K03](trigger-rules-v0.4.md#trg-saf-k03)；[TRG-G01](trigger-rules-v0.4.md#trg-g01) |
| 用途 | TRG-COG-K03；TRG-SAF-K03；TRG-G01 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [AI：WHO. Ethics and governance guidance for large multi-modal models. 2024](https://www.who.int/news/item/18-01-2024-who-releases-ai-ethics-and-governance-guidance-for-large-multi-modal-models)；PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09 |
| 支持边界 | WHO支持治理，不提供文本分类器阈值或临床效度。 |

<a id="dat-cog-02"></a>

### DAT-COG-02｜突发混乱、定向障碍、现实言语改变

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 对话与认知 |
| 来源/入口 | 本人或现实在场者报告；专业核实 |
| 字段与单位 | 是否突发；是否仍持续；不能辨认的人/地等真实例子 |
| 时间窗 | 当前；必须区别已恢复/未确认恢复 |
| 质量、授权与缺失（Excel 原文） | 不能仅凭AI判断文风“混乱”；要核对真实症状。 |
| AI 整理与本人确认（Excel 原文） | 突出原始证据并走急症入口，不等量表 |
| 观察/量表边界 | 不默认用认知筛查替代急症评估 |
| 下游规则 | [TRG-COG-K01](trigger-rules-v0.4.md#trg-cog-k01)；[TRG-X06](trigger-rules-v0.4.md#trg-x06) |
| 用途 | TRG-COG-K01；TRG-X06 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [HALL：NHS. Hallucinations and hearing voices. 更新2025-07-28](https://www.nhs.uk/mental-health/feelings-symptoms-behaviours/feelings-and-symptoms/hallucinations-hearing-voices/) |
| 支持边界 | NHS是官方就医提示，不是LLM诊断规则。 |

<a id="dat-cog-03"></a>

### DAT-COG-03｜异常感知及是否有伤害指令

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 对话与认知 |
| 来源/入口 | 本人现实描述；授权照护者报告 |
| 字段与单位 | 感知类型；新发/加重；是否伤害指令；当前危险 |
| 时间窗 | 当前/近期 |
| 质量、授权与缺失（Excel 原文） | 区分梦境、引用、比喻和真实经历；不能据此直接诊断精神病。 |
| AI 整理与本人确认（Excel 原文） | 必要澄清，不争辩体验；命令伤害线索优先安全路径 |
| 观察/量表边界 | 无默认量表 |
| 下游规则 | [TRG-COG-K02](trigger-rules-v0.4.md#trg-cog-k02)；[TRG-SAF-K01](trigger-rules-v0.4.md#trg-saf-k01)；[TRG-X06](trigger-rules-v0.4.md#trg-x06)；[TRG-COG-K04](trigger-rules-v0.4.md#trg-cog-k04) |
| 用途 | TRG-COG-K02；TRG-SAF-K01；TRG-X06 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [HALL：NHS. Hallucinations and hearing voices. 更新2025-07-28](https://www.nhs.uk/mental-health/feelings-symptoms-behaviours/feelings-and-symptoms/hallucinations-hearing-voices/) |
| 支持边界 | 新发异常感知需要医疗评估；不同急迫程度取决于伴随事实。 |

<a id="dat-cog-04"></a>

### DAT-COG-04｜回复延迟、语速、消息长度

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 对话与认知 |
| 来源/入口 | 仅本App内会话元数据，另行授权 |
| 字段与单位 | 回复间隔秒；消息字符数；语速估计及质量 |
| 时间窗 | 会话内；不得跨平台抓取 |
| 质量、授权与缺失（Excel 原文） | 取得适用授权。保留来源、时间、质量与修订记录。缺失不补0。 |
| AI 整理与本人确认（Excel 原文） | 仅作交互适配/缺失提示，不靠风格直接填症状 |
| 观察/量表边界 | 不评分 |
| 下游规则 | 无临床规则 |
| 用途 | TRG-G01；无独立疾病触发 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[AI：WHO. Ethics and governance guidance for large multi-modal models. 2024](https://www.who.int/news/item/18-01-2024-who-releases-ai-ethics-and-governance-guidance-for-large-multi-modal-models) |
| 支持边界 | 未验证为同频临床指标；不新增情绪/认知风险分。 |

## 社交与功能

<a id="dat-soc-01"></a>

### DAT-SOC-01｜工作学习、自理、社交影响

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 社交与功能 |
| 来源/入口 | 本人自然描述 |
| 字段与单位 | 功能领域；具体受影响事情；本人可选0–3分项；时间 |
| 时间窗 | 本人报告的当前/近期 |
| 质量、授权与缺失（Excel 原文） | 不擅自访问学校考勤、联系人或消息。分项定义待核验。 |
| AI 整理与本人确认（Excel 原文） | 保留具体生活例子，困难≠疾病；不相加成风险总分 |
| 观察/量表边界 | 产品观察0–3，非已验证量表 |
| 下游规则 | [TRG-EMO-K02](trigger-rules-v0.4.md#trg-emo-k02)；[TRG-SOC-K01](trigger-rules-v0.4.md#trg-soc-k01)；[TRG-X01](trigger-rules-v0.4.md#trg-x01) |
| 用途 | TRG-EMO-K02；TRG-SOC-K01；TRG-X01 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [DEP：NICE NG222. Depression in adults: treatment and management. 2022](https://www.nice.org.uk/guidance/ng222/chapter/Recommendations)；PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09 |
| 支持边界 | 指南支持评估功能，不给出本产品0–3或总分转诊阈值。 |

<a id="dat-soc-02"></a>

### DAT-SOC-02｜回避联系、支持网络与求助意愿

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 社交与功能 |
| 来源/入口 | 本人自述；授权支持者补充 |
| 字段与单位 | 相较平常变化；可信支持者；想要普通陪伴/专业帮助 |
| 时间窗 | 当前需要优先 |
| 质量、授权与缺失（Excel 原文） | 独处可能是偏好；没有消息不等于社交退缩。 |
| AI 整理与本人确认（Excel 原文） | 提取明确意愿与可联系对象，不自动发消息 |
| 观察/量表边界 | 无强制量表 |
| 下游规则 | [TRG-SOC-K01](trigger-rules-v0.4.md#trg-soc-k01)；[TRG-SOC-K02](trigger-rules-v0.4.md#trg-soc-k02)；[TRG-SOC-K03](trigger-rules-v0.4.md#trg-soc-k03)；[TRG-X02](trigger-rules-v0.4.md#trg-x02) |
| 用途 | TRG-SOC-K01/K02/K03；TRG-X02 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [DEP：NICE NG222. Depression in adults: treatment and management. 2022](https://www.nice.org.uk/guidance/ng222/chapter/Recommendations)；[HOTNOTE：国家卫生健康委. 解读《心理援助热线技术指南（试行）》. 2021-01-14](https://www.nhc.gov.cn/jkj/c100062/202101/68a7fb6840074476b359b03dab4199ec.shtml) |
| 支持边界 | 不要求用户先达到分数才能表达求助。 |

<a id="dat-soc-03"></a>

### DAT-SOC-03｜预约与转介进度

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 社交与功能 |
| 来源/入口 | 服务系统事件；人工回执 |
| 字段与单位 | request_id；当前owner；发送/接受时间；接收人；备用资源 |
| 时间窗 | 每次状态变化 |
| 质量、授权与缺失（Excel 原文） | 发送、拨号、已读都不等于具名接收；不得伪造值守。 |
| AI 整理与本人确认（Excel 原文） | 整理事件，不由模型宣布接管/结案 |
| 观察/量表边界 | 流程状态，不是症状分数 |
| 下游规则 | [TRG-G02](trigger-rules-v0.4.md#trg-g02)；[MATCH-06](trigger-rules-v0.4.md#match-06) |
| 用途 | TRG-G02；MATCH-06 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [HOTNOTE：国家卫生健康委. 解读《心理援助热线技术指南（试行）》. 2021-01-14](https://www.nhc.gov.cn/jkj/c100062/202101/68a7fb6840074476b359b03dab4199ec.shtml)；PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09 |
| 支持边界 | 具名回执字段与备用队列是产品流程设计，非统一国际时限。 |

## 身体与生活

<a id="dat-bod-01"></a>

### DAT-BOD-01｜心率及测量情境

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 身体与生活 |
| 来源/入口 | 授权手表/本人测量/身体感受 |
| 字段与单位 | heart_rate（次/分）；静息/运动后；体位；咖啡因；持续时间；质量 |
| 时间窗 | 每次有效测量与相应症状时间对齐 |
| 质量、授权与缺失（Excel 原文） | 来源、佩戴和情境可用；不把一个数当作病因。 |
| AI 整理与本人确认（Excel 原文） | 并列记录数值与自述，不先归因为焦虑 |
| 观察/量表边界 | 不折算焦虑量表 |
| 下游规则 | [TRG-BOD-K01](trigger-rules-v0.4.md#trg-bod-k01)；[TRG-BOD-D01](trigger-rules-v0.4.md#trg-bod-d01)；[TRG-X07](trigger-rules-v0.4.md#trg-x07) |
| 用途 | TRG-BOD-D01/K01/K02；TRG-X04 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [HR：American Heart Association. All About Heart Rate](https://www.heart.org/en/health-topics/high-blood-pressure/the-facts-about-high-blood-pressure/all-about-heart-rate-pulse)；[PALP：NHS. Heart palpitations. 更新2026-03-17](https://www.nhs.uk/symptoms/heart-palpitations/) |
| 支持边界 | 118/126次每分是旧案例数值，不是精神转诊线。 |

<a id="dat-bod-02"></a>

### DAT-BOD-02｜HRV

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 身体与生活 |
| 来源/入口 | 授权设备提供的具体HRV指标 |
| 字段与单位 | HRV（ms）；指标类型SDNN/RMSSD等；设备与算法；时段/质量 |
| 时间窗 | 同设备、同指标、相似测量条件 |
| 质量、授权与缺失（Excel 原文） | 不混比不同指标或厂商算法；接入前查设备文档。 |
| AI 整理与本人确认（Excel 原文） | 仅呈现合格趋势，缺失不估填 |
| 观察/量表边界 | 不作为PHQ/GAD答案 |
| 下游规则 | 无临床规则 |
| 用途 | 无独立精神转诊阈值；TRG-G01质控 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[AI：WHO. Ethics and governance guidance for large multi-modal models. 2024](https://www.who.int/news/item/18-01-2024-who-releases-ai-ethics-and-governance-guidance-for-large-multi-modal-models) |
| 支持边界 | 本次没有可直接用于同频精神分诊的统一HRV阈值依据。 |

<a id="dat-bod-03"></a>

### DAT-BOD-03｜心悸、胸痛、气短、晕厥等当前症状

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 身体与生活 |
| 来源/入口 | 本人/现实在场者直接报告 |
| 字段与单位 | 症状；当前性；持续/缓解；放射/伴随症状；原话 |
| 时间窗 | 即时；不等待设备有数据 |
| 质量、授权与缺失（Excel 原文） | 已出现明确警示不等再次确认或填表。 |
| AI 整理与本人确认（Excel 原文） | 优先突出当前事实与现实求助，不安抚为“只是焦虑” |
| 观察/量表边界 | 不需要量表达到分数 |
| 下游规则 | [TRG-BOD-K01](trigger-rules-v0.4.md#trg-bod-k01)；[TRG-BOD-K02](trigger-rules-v0.4.md#trg-bod-k02)；[TRG-BOD-K03](trigger-rules-v0.4.md#trg-bod-k03)；[TRG-X04](trigger-rules-v0.4.md#trg-x04)；[TRG-X05](trigger-rules-v0.4.md#trg-x05)；[TRG-BOD-K04](trigger-rules-v0.4.md#trg-bod-k04) |
| 用途 | TRG-BOD-K02/K03；TRG-X04 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [PALP：NHS. Heart palpitations. 更新2026-03-17](https://www.nhs.uk/symptoms/heart-palpitations/)；[CHEST：NHS. Chest pain. 页面复核2023-08-08](https://www.nhs.uk/symptoms/chest-pain/) |
| 支持边界 | 依据为NHS官方就医提示。中国急救入口按本地配置，不复制英国电话。 |

<a id="dat-bod-04"></a>

### DAT-BOD-04｜食欲、体重、疼痛、用药及酒精/咖啡因

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 身体与生活 |
| 来源/入口 | 本人自述；可选授权药物清单 |
| 字段与单位 | 食欲变化；体重kg；疼痛部位/强度；药名/变更时间；物质/用量单位 |
| 时间窗 | 本人说明时间；用药与不适需时间关联 |
| 质量、授权与缺失（Excel 原文） | 不要求无关用药细节；与处方来源区分；不推测依从性。 |
| AI 整理与本人确认（Excel 原文） | 提取变化与症状，不诊断药物因果，不给自行加减药建议 |
| 观察/量表边界 | 食欲主题可关联PHQ候选；不得用体重直接代答 |
| 下游规则 | [TRG-SAF-K04](trigger-rules-v0.4.md#trg-saf-k04)；[TRG-X05](trigger-rules-v0.4.md#trg-x05) |
| 用途 | TRG-X05；已过量或急症走R0 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [DEP：NICE NG222. Depression in adults: treatment and management. 2022](https://www.nice.org.uk/guidance/ng222/chapter/Recommendations)；[PALP：NHS. Heart palpitations. 更新2026-03-17](https://www.nhs.uk/symptoms/heart-palpitations/) |
| 支持边界 | “任何药物变化+不适都R1”无统一依据，本表改为先按具体症状核实。 |

<a id="dat-bod-05"></a>

### DAT-BOD-05｜血压、血氧、体温扩展

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 身体与生活 |
| 来源/入口 | 本轮不接入；后续按设备与临床方案 |
| 字段与单位 | 血压mmHg；血氧%；体温°C；来源与测量条件 |
| 时间窗 | 未启用 |
| 质量、授权与缺失（Excel 原文） | 没有已批准设备与阈值时不纳入触发。 |
| AI 整理与本人确认（Excel 原文） | 不生成估计值 |
| 观察/量表边界 | 未启用 |
| 下游规则 | 无临床规则 |
| 用途 | 无本轮触发条件 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09 |
| 支持边界 | 不补造数值或通用转诊线。 |

## 风险信息

<a id="dat-saf-01"></a>

### DAT-SAF-01｜本人当前自杀、自伤/伤人意图或伤害指令

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 风险信息 |
| 来源/入口 | 直接询问与本人自述；必要现实在场者 |
| 字段与单位 | 对象；是否现在；是否已发生；当前意图；原话及时间 |
| 时间窗 | 当前安全优先 |
| 质量、授权与缺失（Excel 原文） | 否定、历史、引用不混为本人当前；明确危险不等待所有字段齐全。 |
| AI 整理与本人确认（Excel 原文） | 捕捉并保留证据；安全响应不等待正式评分 |
| 观察/量表边界 | 不以总分预测未来自杀 |
| 下游规则 | [TRG-SAF-K01](trigger-rules-v0.4.md#trg-saf-k01)；[TRG-SAF-K04](trigger-rules-v0.4.md#trg-saf-k04)；[TRG-X07](trigger-rules-v0.4.md#trg-x07) |
| 用途 | TRG-SAF-K01/K04；TRG-X07 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [BSSA：NIMH. Adult Outpatient Brief Suicide Safety Assessment Guide](https://www.nimh.nih.gov/research/research-conducted-at-nimh/asq-toolkit-materials/adult-outpatient/adult-outpatient-brief-suicide-safety-assessment-guide)；[HALL：NHS. Hallucinations and hearing voices. 更新2025-07-28](https://www.nhs.uk/mental-health/feelings-symptoms-behaviours/feelings-and-symptoms/hallucinations-hearing-voices/) |
| 支持边界 | NIMH原路径用于成人ASQ阳性后临床评估；App即时旁路是本地转译。 |

<a id="dat-saf-02"></a>

### DAT-SAF-02｜正式PHQ-9第9题回答

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 风险信息 |
| 来源/入口 | 核验版适龄施测的直接回答 |
| 字段与单位 | item9分值0–3；回顾窗；有效性；施测来源 |
| 时间窗 | 过去2周；当前紧迫性需另问 |
| 质量、授权与缺失（Excel 原文） | 任一非零有效回答独立核查，不等待总表完成。 |
| AI 整理与本人确认（Excel 原文） | 不把安全话语自动写成第9题分值 |
| 观察/量表边界 | 第9题不是完整自杀风险评估 |
| 下游规则 | [TRG-SAF-Q01](trigger-rules-v0.4.md#trg-saf-q01) |
| 用途 | TRG-SAF-Q01；当前危险优先TRG-SAF-K01 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [UWP：华盛顿大学 National HIV Curriculum. Screening for Mental Health Conditions. 2026](https://www.hiv.uw.edu/go/basic-primary-care/screening-mental-disorders/core-concept/all)；[SELF：NICE NG225. Self-harm: assessment, management and preventing recurrence. 2022](https://www.nice.org.uk/guidance/ng225/chapter/recommendations#risk-assessment-tools-and-scales) |
| 支持边界 | 低总分不抵消安全题；0分也不能排除所有安全问题。 |

<a id="dat-saf-03"></a>

### DAT-SAF-03｜自伤/自杀历史、计划与现实保护支持

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 风险信息 |
| 来源/入口 | 有能力人员必要询问；本人愿意补充 |
| 字段与单位 | 事件时间；当前/历史；是否需要即刻保护；可安全联系的支持者 |
| 时间窗 | 当前核查与经授权历史 |
| 质量、授权与缺失（Excel 原文） | 仅收集安全处置所需，不要求在普通Profile里详填创伤史。 |
| AI 整理与本人确认（Excel 原文） | 结构化事实，不输出未来风险概率或“安全承诺书” |
| 观察/量表边界 | 安全评估不由AI总分替代 |
| 下游规则 | [TRG-SAF-K03](trigger-rules-v0.4.md#trg-saf-k03) |
| 用途 | TRG-SAF-K03；当前事实改变则走即时路径 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [BSSA：NIMH. Adult Outpatient Brief Suicide Safety Assessment Guide](https://www.nimh.nih.gov/research/research-conducted-at-nimh/asq-toolkit-materials/adult-outpatient/adult-outpatient-brief-suicide-safety-assessment-guide)；[SELF：NICE NG225. Self-harm: assessment, management and preventing recurrence. 2022](https://www.nice.org.uk/guidance/ng225/chapter/recommendations#risk-assessment-tools-and-scales) |
| 支持边界 | 来源支持临床核实项目，不授权志愿者独立做临床判断。 |

<a id="dat-saf-04"></a>

### DAT-SAF-04｜现实第三方安全情况

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 风险信息 |
| 来源/入口 | 用户报告现实朋友/家人当前情况 |
| 字段与单位 | 对象关系；当前性；可识别情境；转述与虚构标记 |
| 时间窗 | 当下 |
| 质量、授权与缺失（Excel 原文） | 只保留必要信息；第三方不写入本人量表。 |
| AI 整理与本人确认（Excel 原文） | 澄清对象，呈现现实援助路径 |
| 观察/量表边界 | 非本人量表答案 |
| 下游规则 | [TRG-SAF-K02](trigger-rules-v0.4.md#trg-saf-k02) |
| 用途 | TRG-SAF-K02；引用排除TRG-COG-K03 |
| 可见范围（Excel 原文） | 用户可见/可纠正。志愿者按授权看最小摘要，管理端默认仅流程状态，专业人员按职责取证。 |
| 参考来源 | [HALL：NHS. Hallucinations and hearing voices. 更新2025-07-28](https://www.nhs.uk/mental-health/feelings-symptoms-behaviours/feelings-and-symptoms/hallucinations-hearing-voices/)；PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09 |
| 支持边界 | 转述到App分流的具体路径仍须本地审核。 |

## 用户Profile

<a id="pro-u-01"></a>

### PRO-U-01｜年龄与适龄分组

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 本人提供；必要授权下监护人提供 |
| 字段与单位 | 年龄（岁）；声明/核验方式；核验日期；适用年龄段 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [TRG-G01](trigger-rules-v0.4.md#trg-g01)；[MATCH-01](trigger-rules-v0.4.md#match-01) |
| 用途 | MATCH-01；TRG-G01；选择适龄工具/专业资源 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 优先年龄而非完整生日。14岁是个人信息特别保护边界之一，不是临床成人线。 |

<a id="pro-u-02"></a>

### PRO-U-02｜性别与陪伴者偏好

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 可选档案项；本人主动提出偏好 |
| 字段与单位 | 自愿自述性别；不愿透露；是否有陪伴者性别偏好 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-04](trigger-rules-v0.4.md#match-04) |
| 用途 | MATCH-04；不改变临床阈值 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 不根据姓名/声音推断性别；临床相关生理信息确有必要时另行说明收集。 |

<a id="pro-u-03"></a>

### PRO-U-03｜爱好与感兴趣话题

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 可选点选或本人主动分享 |
| 字段与单位 | 兴趣标签；不喜欢/不想聊的话题；更新时间 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-04](trigger-rules-v0.4.md#match-04) |
| 用途 | MATCH-04；聊天话题和活动建议 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 未收集不降低服务优先级；不自动抓取社交账号。 |

<a id="pro-u-04"></a>

### PRO-U-04｜MBTI自述（可选）

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 完全自愿；不要求付费测试 |
| 字段与单位 | 自述类型/不知道/不填写；来源标为自述 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-05](trigger-rules-v0.4.md#match-05) |
| 用途 | MATCH-05；只供用户选择展示或聊天 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 不作为诊断、风险分层、资质认定或分配排除条件；不从聊天预测类型。 |

<a id="pro-u-05"></a>

### PRO-U-05｜想获得帮助的问题

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 本人描述，关联动态观察记录 |
| 字段与单位 | 当前困扰主题；期待帮助；不想被问的内容；更新时间 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-03](trigger-rules-v0.4.md#match-03) |
| 用途 | TRG-SOC-K02/K03；MATCH-03；七模块观察 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 记录“希望帮助的主题”，不是擅自贴诊断；当前情绪是动态状态，不永久固化。 |

<a id="pro-u-06"></a>

### PRO-U-06｜喜欢的颜色、宠物和展示风格

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 可选个性化设置 |
| 字段与单位 | 颜色/主题ID；宠物ID；文字大小/减少动效偏好 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-05](trigger-rules-v0.4.md#match-05) |
| 用途 | 仅UI个性化；无临床触发 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 颜色和宠物选择不用于推断心理问题或临床风险。 |

<a id="pro-u-07"></a>

### PRO-U-07｜交流方式、语言及关怀时段

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 用户设置或主动说明 |
| 字段与单位 | 语音/文字/点选；语言/方言；可联系时段；免打扰 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-02](trigger-rules-v0.4.md#match-02)；[MATCH-04](trigger-rules-v0.4.md#match-04) |
| 用途 | MATCH-02/04；主动关怀频次和无语音回退 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 避免因听障、语言或沉默降低服务机会；不将没响应直接判为恶化。 |

<a id="pro-u-08"></a>

### PRO-U-08｜所属服务地区与联系授权

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 用户主动提供，按服务必要程度细化 |
| 字段与单位 | 区/街道服务范围；联系方式与可联系对象；授权用途 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-02](trigger-rules-v0.4.md#match-02)；[MATCH-06](trigger-rules-v0.4.md#match-06) |
| 用途 | MATCH-02/06；转介资源定位 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 一般匹配不需要家庭精确坐标或持续定位；紧急共享按经审查的例外流程。 |

<a id="pro-u-09"></a>

### PRO-U-09｜年龄相关授权与数据分享设置

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 用户Profile |
| 来源/入口 | 适龄告知与适用授权流程 |
| 字段与单位 | 授权范围；版本；授权人；时间；撤回状态；监护关系核验状态 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [TRG-G01](trigger-rules-v0.4.md#trg-g01)；[MATCH-01](trigger-rules-v0.4.md#match-01)；[MATCH-07](trigger-rules-v0.4.md#match-07) |
| 用途 | MATCH-01；TRG-G01；控制日记/摘要/原文共享 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | [LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm)；[ASQ：NIMH. Ask Suicide-Screening Questions Toolkit](https://www.nimh.nih.gov/research/research-conducted-at-nimh/asq-toolkit-materials) |
| 支持边界 | 未满14周岁信息处理需核验监护同意等要求。其他未成年人也需适龄保护。 |

## 志愿者Profile

<a id="pro-v-01"></a>

### PRO-V-01｜年龄与岗位适用范围

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 本人申请；管理端按岗位要求核验 |
| 字段与单位 | 年龄（岁）；岗位年龄条件；核验状态 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-01](trigger-rules-v0.4.md#match-01) |
| 用途 | MATCH-01/03 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 实施权限澄清（优先） | 本人提交年龄资料或更正申请；管理端核验岗位适龄状态。岗位条件未核验不能以跳过填写获得接单资格。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 不单靠与用户年龄相近分配；未成年人岗位权限如开放须单独审核。 |

<a id="pro-v-02"></a>

### PRO-V-02｜性别与自愿展示偏好

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 可选档案项 |
| 字段与单位 | 自述性别/不愿透露；公开范围 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-04](trigger-rules-v0.4.md#match-04) |
| 用途 | MATCH-04 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 不根据性别推断同理心、能力或疾病处理水平。 |

<a id="pro-v-03"></a>

### PRO-V-03｜爱好与语言/交流风格

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 可选自报；服务语言按需要核验 |
| 字段与单位 | 兴趣标签；服务语言；语音/文字偏好 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-04](trigger-rules-v0.4.md#match-04) |
| 用途 | MATCH-04 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 共同爱好仅辅助匹配；语言能力与临床资质分开。 |

<a id="pro-v-04"></a>

### PRO-V-04｜MBTI自述（可选）

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 完全自愿填写 |
| 字段与单位 | 自述类型/不知道/不填写；展示范围 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-05](trigger-rules-v0.4.md#match-05) |
| 用途 | MATCH-05 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 不能替代培训、督导或岗位能力核验；不按类型招募、淘汰或排队。 |

<a id="pro-v-05"></a>

### PRO-V-05｜自报擅长话题与服务经验

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 申请人自报 |
| 字段与单位 | 自报话题；服务经历；不愿接/需支持的话题 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-03](trigger-rules-v0.4.md#match-03) |
| 用途 | MATCH-03；管理复核候选 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | “擅长解决的问题”应改为“可支持的话题”；自报不等于已获临床处理资格。 |

<a id="pro-v-06"></a>

### PRO-V-06｜已核验能力、培训和人群范围

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 管理端核验；合格培训/督导记录 |
| 字段与单位 | 核验通过的能力；资质/培训证据；有效期；可服务年龄/人群；边界 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-01](trigger-rules-v0.4.md#match-01)；[MATCH-03](trigger-rules-v0.4.md#match-03) |
| 用途 | MATCH-03；专业核查不得交普通志愿者独立负责 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 实施权限澄清（优先） | 本人可提交培训/资质材料或更正申请；核验结果、有效期与可服务人群只能由有权限管理者核验维护，AI 不授予资格。缺少有效核验不得分配超出已确认能力的任务。 |
| 参考来源 | [HOTNOTE：国家卫生健康委. 解读《心理援助热线技术指南（试行）》. 2021-01-14](https://www.nhc.gov.cn/jkj/c100062/202101/68a7fb6840074476b359b03dab4199ec.shtml)；PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09 |
| 支持边界 | 热线规范可参考，但App志愿者不是自动等同热线咨询员或临床人员。 |

<a id="pro-v-07"></a>

### PRO-V-07｜偏好的志愿时间

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 本人填写并可更新 |
| 字段与单位 | 星期；时段；时区；偏好白/夜班；临时不可用 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-02](trigger-rules-v0.4.md#match-02) |
| 用途 | MATCH-02；只用于排班候选 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm) |
| 支持边界 | 偏好时段≠已确认值班；不得据此向用户承诺有人接待。 |

<a id="pro-v-08"></a>

### PRO-V-08｜确认排班、接待容量与休息

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 管理排班与系统事件；本人确认 |
| 字段与单位 | 班次开始/结束；在岗确认；今日已接人数；并发；配置上限；休息状态 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-02](trigger-rules-v0.4.md#match-02) |
| 用途 | MATCH-02；满额/休息时不新增普通派单 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 实施权限澄清（优先） | 管理者维护排班与配置上限，系统按事件维护去重人数及并发。本人只能确认在岗、休息或提出变更，不可改计数、提高上限或跳过容量检查。 |
| 参考来源 | PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09；[HOTNOTE：国家卫生健康委. 解读《心理援助热线技术指南（试行）》. 2021-01-14](https://www.nhc.gov.cn/jkj/c100062/202101/68a7fb6840074476b359b03dab4199ec.shtml) |
| 支持边界 | 每日上限/夜班长度由机构核定，本表不编造统一国际数值。 |

<a id="pro-v-09"></a>

### PRO-V-09｜所属地区、督导与升级联系人

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 机构维护，不由志愿者自填即生效 |
| 字段与单位 | 服务地区；值班管理者ID；督导ID；专业备用路径；最近核验时间 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-02](trigger-rules-v0.4.md#match-02)；[MATCH-06](trigger-rules-v0.4.md#match-06) |
| 用途 | MATCH-06；TRG-G02；有紧急情况可直接求助高级角色 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 实施权限澄清（优先） | 机构维护地区、督导及专业备用路径；本人可提请更正，不可自行变更接续责任人。缺少可用接续路径时不得假定资源已接通。 |
| 参考来源 | [HOTNOTE：国家卫生健康委. 解读《心理援助热线技术指南（试行）》. 2021-01-14](https://www.nhc.gov.cn/jkj/c100062/202101/68a7fb6840074476b359b03dab4199ec.shtml)；PRODUCT：同频B2产品需求与本次设计假设. 2026-09-09 |
| 支持边界 | 管理协调≠专业接管；实际联系人与接收能力须可核验。 |

<a id="pro-v-10"></a>

### PRO-V-10｜服务授权及关系边界

| 数据环节 | 内容 |
| --- | --- |
| 模块 | 志愿者Profile |
| 来源/入口 | 管理按岗位授予并审计 |
| 字段与单位 | 可见数据范围；授权生效/终止；当前任务角色；保密/边界培训状态 |
| 时间窗 | 入驻后可渐进补充；变更时更新并保留核验时间 |
| 质量、授权与缺失（Excel 原文） | 非必要项可跳过；不填不影响普通求助。AI不从聊天/声音猜人口属性。 |
| AI 整理与本人确认（Excel 原文） | 用户主动选择或明确表达后记录；修改同步匹配，保留最小审计 |
| 观察/量表边界 | Profile字段，不是临床量表 |
| 下游规则 | [MATCH-07](trigger-rules-v0.4.md#match-07) |
| 用途 | MATCH-07；最小权限、任务结束撤销访问 |
| 可见范围（Excel 原文） | 本人查看修改。匹配服务仅取必要标签；对端不默认看到原始档案，管理按职责访问。 |
| 实施权限澄清（优先） | 有权限管理者按岗位授予或撤回权限，系统审计执行；本人可查看适用范围和申请更正，不能自授或扩大访问权限。 |
| 参考来源 | [LAW：中华人民共和国个人信息保护法. 2021](https://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm)；[HOTNOTE：国家卫生健康委. 解读《心理援助热线技术指南（试行）》. 2021-01-14](https://www.nhc.gov.cn/jkj/c100062/202101/68a7fb6840074476b359b03dab4199ec.shtml) |
| 支持边界 | 不默认展示用户完整日记、通讯录或志愿者私人联系方式。 |

