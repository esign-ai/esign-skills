---
name: esign-compare-contract
displayName: e签宝合同比对
name_en: esign-compare-contract
description_en: "Use when the user asks to compare two contracts, identify edits, explain differences between new and old versions, check changes in a counterparty's returned draft, find added or deleted clauses, or examine contract version differences. Compare two provided or selected contract files or explicitly provided contract texts. Identify additions, deletions, modifications and replacements, distinguish material, non-material and uncertain changes, and explain their effects with source evidence. Handle follow-up questions about specific differences in an existing comparison. If source material is missing, ask only for the missing material. Does not perform standalone risk review of a single contract. Route fact checks across multiple files concerning one specific question to contract Q&A."
argument-hint: "提供两份合同，可注明基准件与比对件"
description: "当用户要求“比较这两份合同”“看看修改了哪些内容”“新版和旧版有什么区别”“对方回传稿改了什么”“有没有新增或删除条款”，或核对两份合同的版本差异时使用。比较用户提供或已选定的两份合同文件或明确正文，识别新增、删除、修改、替换，区分实质变更、非实质变更和不确定项，并结合原文解释影响。支持在已有比对结果上追问具体差异；缺少材料时只引导补齐所缺材料。不用于单份合同风险审查；仅针对某个具体问题核对多个文件中的事实，由合同问答承接。"
---

# e签宝合同比对

本能力由 mono 统一入口按意图加载，沿用 `esign` CLI 执行。跨能力续办时复用已确认的文件、流程、身份及已完成步骤，并继续读取 mono 内对应能力入口；不得虚构完成。

比较明确的两份合同，说明变化及其影响。开始时读取[文件来源规则](references/file-sources.md)与[通用依据规则](references/evidence-rules.md)，所有分支均适用。

## 1. 确定文件与方向

- 优先遵从用户指定的基准件与比对件。未指定且上传或选择顺序可靠时，第一份作为基准，在结果明确方向；顺序不可靠才询问。不要仅凭文件名年份推断真实修订先后。
- 缺两份时先请提供两份，缺一份只补一份并等待，不从电脑或平台自行查找配对。用户上传时主动标注原稿/修改稿可以直接保留；未取得实际文件时不弹出虚构文件的方向选择，也不先问哪方利益。超过两份且未指定组合时只澄清组合。已明确两份不重复确认，也不自动对所有组合进行比对。
- 两份分别按来源规则取得文件，允许本地与流程文件混合；不搜索同名合同。来源、文件与范围分别保留。

## 2. 读取两份正文

分别读取两份选定文件内部的正文、表格和交叉引用，原样保存本次 `base.txt`、`compare.txt`。用户直接提供明确正文时可以使用，未提供材料不视为已读。

- 每轮独立目录，不覆盖原文件，不混用另一文件或已被替换文件的正文。
- 一份完全无法读取时停止依赖两份正文的比对，不生成“无差异”。部分可读时仅比较可覆盖范围并说明限制，不能静默只看主要条款。
- 本期只比正文及表格，不识别或比较印章的新增、缺失、位置、颜色等，不将印章放入差异明细，不声称印章相同。扫描件正文仍正常读取；用户明确要求印章比对时简短说明暂不包含此项。文本抽取不证明字体、版式、笔迹或图片相同，不宣称完成视觉比对。

## 3. 对齐、识别与解读

读取[差异判断规则](references/comparison-rules.md)与[结果协议](references/result_schema.md)：

1. 按标题、条款编号、表格对象和含义对齐，检查移动到其他位置的内容，避免重复算新增与删除。
2. 逐项记录基准与比对件原文、真实位置；选择新增、删除、修改、替换，边界不明按修改，同一变化只记一次。
3. 分类为实质变更、非实质变更或不确定。不能因改动短或仅编号、标点而自动判非实质；涉及交叉引用时核对对应关系。
4. 简洁解释有依据的变化和影响，未知立场时客观描述必要影响，不为每项凑双方长篇解读。各类费用分开计算，不将保证金加到费用后称为不可退费用；距离变化不推为面积或收益变化；正文一致不推为流程状态、角色或风险相同。分类依据在内部保留，不和变化说明重复展示。
5. 长合同分段比较后合并、去重，核对表格、交叉引用和未处理范围。完整覆盖与条目不确定是两回事，不为凑类别编造不确定项。

## 4. 校验结果

按协议生成本轮 `compare_payload.json`，明细按合同条款顺序列出，保留全部真实变化及必要上下文。使用Python 3.9+和标准库脚本完成内部校验：

```bash
python3 '<当前能力目录>/scripts/validate_compare_result.py' --input-file '<本轮compare_payload.json>' --base-text-file '<本轮base.txt>' --compare-text-file '<本轮compare.txt>' --output-json '<本轮compare_result.json>'
```

脚本检查结构、连续原文引用、重复项、操作证据侧及计数，保留空格、制表符与换行边界；不做语义比对，不证明位置正确或差异无遗漏。计数从明细产生，不手填。抽取空白变化不证明真实版式变化，不为解释处理方式向用户提供坐标、命令或Skill改进反馈。

根据真实正文修正，最多修正重跑两次；不能修改源文、删除真实变化或谎报覆盖求通过。仍失败时只说明当前结果暂不能可靠交付；执行环境不可用不谎称完成，也不自动改环境。

## 5. 交付与后续

必须读取并遵循本包[结果交付规则](references/result-delivery.md)。它决定直接在会话展示，还是生成报告，不自行扩展当前包的交付能力。

完整结果先简短标明两份文件及方向，再按合同顺序展示全部实际差异的原文和位置，随后总结实质、非实质及有内容的待确认变化；空模块连标题一起省略。必要原文与位置保留，不默认展示内部过程、固定范围声明或“未比较印章、字体、版面”等通用说明。

用户明确只要总结时，完成必要比对与校验后直接给总结，不强制展示diff明细或生成报告。追问某项时直接解释。JSON仅作内部数据，真实读取缺口影响结论时只说具体问题；无差异仅说明正文未发现差异，不推断印章或流程相同。文件入口以实际名称指引，不使用上方／下方卡片等位置措辞。

描述变化时明确实际文件名称；基准是新版、比对件是旧版时，使用“旧版比新版少……”等方向清楚的表述，不暗示实际修订历史。保留两份文件与差异证据关联用于追问；文件替换后按来源规则重新读取、比对、计数，不沿用旧结论。

用户已要求比对后继续问答或审查时，保留选定文件、原任务与已完成步骤，继续实际可用的对应 Skill，不要求重选。只对特定问题核对两份文件的事实由问答处理，完整版本差异识别由本 Skill 处理。

本 Skill 不做单份合同的独立审查、起草、签署或审批；只咨询方法不执行。用户取消后停止后续处理，不附加身份设置或其他任务。

## CLI 与平台接入

获取平台文件或执行远端转换、OCR 时，统一读取[CLI 基础能力](../cli/index.md)，完成后恢复当前比对步骤。粘贴正文和纯本地处理不前置登录。

## 结构化提问节点

依赖关系、同批提问、文字降级及答案吸收统一遵循[提问与答案处理规则](references/question-rules.md)；以下节点决定可问范围，不为合批增加问题。

只在比对对象或方向存在实际歧义时按[结构化提问规则](references/question-rules.md)：多于两份已提供文件且无法确定比对对象时选择两份；已确定两份但无法从用户指令或可靠上传顺序确定基准时，只选择基准文件。

用户已指定方向直接采用；未指定但上传顺序可靠时仍按原规则处理，在结果中说明，不增加强制基准确认。缺文件时直接引导上传，不自行检索候选。不询问比对立场、报告格式或明细／总结模式；用户明确要总结时直接总结。报告交付和简短结论沿用原有规则。

## 正文读取入口

处理真实文件时统一使用 `esign document extract-contract-body`，按[文档处理规则](references/document-processing.md)选择真实本地路径或已选文件的真实 fileKey，并检查正文及覆盖情况。明确粘贴的正文直接使用；没有材料先按原流程引导提供，不运行抽取。

两份文件分别抽取，将原始 data.content 分别保存为本次 base.txt、compare.txt，再执行差异分析与现有校验。表格关系不清才辅助核对，保持两个来源及证据版本独立；HTML报告和只要总结的分支不变。

