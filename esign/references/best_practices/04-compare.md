# 比对场景

> 能力入口见 [比对能力](../products/compare.md)。本文件是场景 recipe 表 + 深挖小节；通用规范（并行、翻页、ID 传递）见 [_common/conventions.md](_common/conventions.md)。recipe 为推荐组合（不吻合时可自行组合）；具体判断、边界与输出仍以能力主体及其引用规则为准。

| Recipe | 行动指南（推荐路线） |
|--------|-------------------|
| compare-two-files | 1. 确认恰好两份合同：基准件与比对件。用户指定方向优先；未指定但上传／选择顺序可靠时第一份作基准并在结果说明；顺序不可靠才只问基准<br>2. 缺一份只补一份并等待，不从电脑或平台自行查找配对；超过两份且未指定组合时只澄清组合（支持多选时核对恰好两份，否则文字收集两份文件名）<br>3. 两份分别按 [文件来源规则](../products/compare/file-sources.md) 取得，允许本地与流程文件混合；每轮独立目录，不覆盖原文件<br>4. 分别抽取：`esign document extract-contract-body --file '<真实路径>'` 或 `--file-key '<真实fileKey>'`，将原始 `data.content` 分别保存为本次 `base.txt`、`compare.txt`<br>5. 一份完全不可读时停止依赖两份正文的比对，不生成「无差异」；部分可读时只比较可覆盖范围并说明限制 |
| compare-analyze | 1. 按 [差异判断规则](../products/compare/comparison-rules.md) 对齐标题、条款编号、表格对象与含义，检查移动内容，避免重复算新增与删除<br>2. 逐项记录两份原文与真实位置，操作选新增／删除／修改／替换（边界不明按修改，同一变化只记一次）；影响分类选实质／非实质／不确定<br>3. 不能因改动短或仅编号标点就自动判非实质；涉及交叉引用时核对对应关系<br>4. 未知立场时客观描述必要影响，不为每项凑双方长篇解读；分类依据内部保留，不与变化说明重复展示 |
| compare-validate | 1. 按 [结果协议](../products/compare/result_schema.md) 生成本轮 `compare_payload.json`，明细按合同条款顺序列出，保留全部真实变化及必要上下文<br>2. `python3 '<本技能目录>/scripts/validate_compare_result.py' --input-file '<本轮compare_payload.json>' --base-text-file '<本轮base.txt>' --compare-text-file '<本轮compare.txt>' --output-json '<本轮compare_result.json>'`<br>3. 脚本检查结构、连续原文引用、重复项、操作证据侧及计数；不做语义比对，不证明位置正确或差异无遗漏<br>4. 按真实正文修正，最多修正重跑两次；不得修改源文、删除真实变化或谎报覆盖求通过 |
| compare-report | 1. 按 [比对展示规则](../products/compare/compare-display.md) 在原 payload 之外生成本轮 `compare_view.json`，只补短标题、变化概括、事实确认状态及表格字段，不改写证据引用<br>2. `python3 '<本技能目录>/scripts/render_contract_report.py' --kind compare --input-file '<本轮compare_payload.json>' --presentation-file '<本轮compare_view.json>' --source-text '<本轮base.txt>' --compare-text '<本轮compare.txt>' --output-dir '<本轮报告目录>'`（入参见下方「报告渲染入参」）<br>3. 使用真实文件显示名称（`files.base` 为用户可识别的基准文件名）；自动命名为 `合同比对报告-基准文件名-比对日期.html`<br>4. 核对退出成功、文件真实存在且可通过实际入口访问后再展示；会话只给简短主要变化结论和真实报告入口 |
| compare-summary-only | 1. 用户明确只要总结时，完成必要比对与校验后直接给总结，不运行渲染器、不生成报告、不展示 diff 明细<br>2. 结论须换成本次事实，说明两份文件与方向及主要变化<br>3. 无差异仅说明正文未发现差异，不推断印章、字体、版式或流程状态相同 |
| compare-followup | 1. 同一任务连续追问沿用本次比对证据；文件替换后按来源规则重新读取、比对、计数，不沿用旧结论<br>2. 描述变化时明确实际文件名称；基准是新版、比对件是旧版时用「旧版比新版少……」等方向清楚的表述，不暗示实际修订历史<br>3. 用户要明细时直接在会话展开，保留两份文件与差异证据关联<br>4. 用户取消即停止，不附加身份设置或其他任务 |

## 四分类：操作类型 vs 影响分类

两个**独立**维度，同一变化各选一个，不重复计数：

- **操作类型**（怎样改了）：`add` 新增、`delete` 删除、`modify` 修改、`replace` 替换。替换 = 可对应的原条款被另一套内容整体取代；修改 = 局部字段或措辞变化；**边界模糊按修改**。条款换位置先对齐再判断，纯位置变化不虚构内容差异。不输出 `move`。
- **影响分类**（改变了什么意义）：`material` 实质、`non_material` 非实质、`uncertain` 不确定。不可把「结构／关键要素／语义」当影响分类。

## 影响判定要点

- **实质**检查项：主体与角色、金额税费、付款条件、数量规格、交付验收、期限续约、权利义务、责任上限、违约救济、解除条件、保密与知识产权、争议处理、定义与适用范围。一个否定词、「应当→可以」、收款账号或引用编号变化都可能实质改变义务；删除违约金只说明原约定被删除，不推断「不再承担任何责任」。
- **非实质**仅在核实含义不变后用：文字纠正、编号与交叉引用同步更新、含义等价表达。**不按字符数量分类**，标点／排版也可能影响范围与解释。
- **不确定**适用：已定位疑似变化但 OCR 不可靠、含义有多种合理解释、关键定义或附件缺失。写清不确定来源与应核实什么，不只说「建议人工确认」。确认实质但不知进一步影响时不必整条降为不确定，把未知写入该条 `questions`。
- 立场未知时客观描述必要影响；用户明确站在某方可按其角色解释，但分类仍基于原文。

## 校验入参

```bash
python3 '<本技能目录>/scripts/validate_compare_result.py' --input-file '<本轮compare_payload.json>' --base-text-file '<本轮base.txt>' --compare-text-file '<本轮compare.txt>' --output-json '<本轮compare_result.json>'
```

- 顶层仅 `schema_version`（固定 `2.0`）／`files`（`base`+`compare` 显示名称，不填绝对路径或下载 URL）／`coverage`／`differences`，全部必填，不加额外字段。
- 每条 `differences`：`id` 唯一；`operation` 仅不确定项可为 `null`；`classification` 三选一；新增侧无原文为空串、有原文侧须有可复核位置；非空引用须在对应 `base.txt`／`compare.txt` 命中（仅统一 CRLF/CR/LF）；`analysis` 为完整自然句；`basis` 内部保留；`questions` 不确定项至少一条。
- 完整比对且明细为空时，要求两份正文仅统一换行后相同；不相同必须补齐，不得谎报部分比对。
- 脚本只验结构、连续原文、重复项与证据侧，**不做语义比对**，不证明位置正确或差异无遗漏；退出码 0 且指定 JSON 存在才算完成，失败最多修正重跑两次。

## 报告渲染入参

```bash
python3 '<本技能目录>/scripts/render_contract_report.py' --kind compare --input-file '<本轮compare_payload.json>' --presentation-file '<本轮compare_view.json>' --source-text '<本轮base.txt>' --compare-text '<本轮compare.txt>' --output-dir '<本轮报告目录>'
```

- `compare_view.json` 只补短标题、变化概括、事实确认状态与表格字段，**不改写证据引用**；证据仍用原校验器、原 payload。
- `files.base` 用用户可识别的真实基准文件名，不用内部重命名的 `base.pdf`。
- 自动命名 `合同比对报告-基准文件名（去扩展名）-比对日期.html`；`--date` 只能填真实比对日期 `YYYY-MM-DD`，不用合同日期。
- 仅退出成功且返回的 `file` 真实可访问才交付；会话只给一句主要变化结论 + 真实报告入口，不铺开全部明细。渲染失败保留分析、说明未生成报告并给可用结论，不伪造入口。

## 关键区分

- **比对 vs 审查**：比对识别两版差异并分类影响；审查针对单份评风险、给等级与修改建议，不产出改后正文。
- **比对 vs 总结**：比对面向两份文件的版本差异；总结面向单份合同的概述＋关键字段，不评差异。
- **比对 vs 问答**：只对特定问题核对两份文件中的事实由问答承接；完整版本差异识别由比对承接。
- **覆盖限制 vs 条目不确定**：第 3 页未读取是覆盖限制（`coverage.status=partial`），第 3 页金额疑似差异且无法辨认是条目不确定；前者不能伪装成已完成全文比较。
- **基准方向 vs 真实时间**：基准方向不等于真实修订先后；实际新旧未知时只描述差异、不编造修订过程，逆时间比较时明确文件名。
