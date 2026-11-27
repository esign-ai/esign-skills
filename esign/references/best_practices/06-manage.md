# 发起与管理场景

> 能力入口见 [发起与管理能力](../products/manage.md)。本文件是场景 recipe 表 + 深挖小节；通用规范（并行、翻页、ID 传递）见 [_common/conventions.md](_common/conventions.md)。recipe 为推荐组合（不吻合时可自行组合）；具体判断、边界与输出仍以能力主体及其引用规则为准。

| Recipe | 行动指南（推荐路线） |
|--------|-------------------|
| manage-classify | 1. 「签合同」明确指向新发起还是已有流程时直接选分支；无法判断只澄清这一点，不默认新建流程<br>2. 新发起 → [发起流程](#发起流程create-flow)；查合同／待我签署／列表进度／查看填写／签署入口／下载／撤回 → [处理流程](#处理流程process)；待我审批／审批历史／同意拒绝 → [审批](#审批approval)<br>3. 「驳回」按真实审批任务处理，不扩写为拒签或作废；「作废」不能解释为撤回<br>4. 仅咨询方法、要求方案或明确暂不执行时，不调用业务命令 |
| manage-create-flow | 1. 明确材料：文件发送按 [文件选择规则](../products/manage/files.md)；模板发送先 `esign sign template select --flow-template-name '<关键词>' --page-num 1 --page-size 50 --format json` 展示真实候选并等待选择<br>2. 无可用文件／目标／模板意图时只引导上传并等待，不先建空草稿、不搜索电脑或平台同名合同<br>3. 核对必要字段：企业签署方（企业名称、企业类型、签署联系人、手机号或邮箱至少一种）、个人签署方（姓名、个人类型、手机号或邮箱至少一种）；其他必填仅以模板或真实接口返回为准，不造必填项<br>4. `esign sign draft start` → `esign sign draft set --draft-id '<draftId>' --files '<完整files数组>'`（或 `--flow-template-id`）→ `--signers '<当前完整signers数组>'`；修改文件／名称／签署配置必须 `draft set`<br>5. 不带 `--yes` 的 `esign sign draft submit --draft-id '<draftId>'` 检查草稿（不是发送）；核对实际文件、模板、签署配置及必要设置<br>6. 信息完整、检查通过且用户明确确认覆盖当前完整配置后，`esign sign draft submit --draft-id '<draftId>' --yes` 真正发送；按 [发起恢复](../products/manage/create-recovery.md) 核对结果，提供实际 `longResultUrl`，不把已发起说成已签署 |
| manage-process | 1. 查询：`esign sign process list`（常用 `--doc-query-type`／`--process-status-list`／`--title`／`--contract-no`／`--page-num`／`--page-size`）；待我签署 `--doc-query-type 5 --process-status-list 2`，我发起的 `--doc-query-type 1`，明确查全部 `--doc-query-type 0`<br>2. 保留 `processId`／`processName`／`processStatus`／`processStatusDesc`／`operations`／`dedicatedCloudId`；内部 ID 只存上下文，不展示给用户；成功空结果用文字说明，当前页空但 `total>0` 提示调整页码<br>3. 单个流程打开／查看／填写／签署：`esign sign process url --process-id '<processId>'`；未取得 URL 时返回流程名或业务编号并提示到系统打开，不拼链接<br>4. 撤回：仅当状态为 `填写中`／`签署中`／`合同审批中` 且 `operations` 含 `revoke` 才执行 `esign sign process revoke --process-id '<processId>' --revoke-reason '<撤回原因>'`；原因必填且绑定当前 processId<br>5. 下载：每个流程 `operations` 须含 `download`；批量下载校验 `dedicatedCloudId`（全空或同一非空才允许），多个 ID 用英文逗号拼进同一个 `--process-id`（不是 `--process-ids`），返回 `下载地址：<URL>`／任务中心地址／`failReason`<br>6. 批量签署：完整选中范围放同一次 `esign sign process batch --process-ids '<id1>' --process-ids '<id2>'`，每个 ID 一个参数；禁止逐份调 `process url`、禁止按单个 ID 循环调 `process batch`；成功只交付一个真实 `redirectUrl` |
| manage-approval | 1. 查询：`esign sign approval list`（常用 `--query-type`／`--approval-type`／`--approval-name`／`--contract-no`／`--process-id`／`--page-num`／`--page-size`）；待我审批 `--query-type 3`，审批历史 `--query-type 4`，我发起 `--query-type 2`，我收到 `--query-type 1`<br>2. 空结果按 [审批流程](../products/manage/approval.md) 回显实际筛选条件并给放宽建议；`total>0` 但当前页空不得说没有待审批；保留 `approvalCode`／`taskId`／名称／类型／状态／流程ID／合同编号／操作项供「第 N 条」指代<br>3. 访问入口：`esign sign approval url --approval-code '<approvalCode>'`；未取得 URL 时给可识别审批名称或业务编号<br>4. 单条同意：用户明确「无意见」时 `esign sign approval agree --approval-code '<code>' --task-id '<taskId>' --reason '无'`（不是省略或空值）；单条拒绝 `esign sign approval refuse --approval-code '<code>' --task-id '<taskId>' --reason '<驳回原因>'`（原因必填）<br>5. 批量：`esign sign approval batch-agree --operate-data '[{"approvalCode":"<code1>","taskId":"<id1>"}]' --remark '<意见或"无">'`；`batch-refuse` 同理，`--remark` 传驳回原因<br>6. 不统一前置 `approval will`；目标、范围及必填信息齐备后直接提交，由服务端决定认证；已明确需认证的任务改给真实审批入口，不再无认证提交 |
| manage-followup | 1. 切换账号／企业／环境后清除旧列表指代、待执行动作、原因意见绑定、认证与批量结果上下文，再查询<br>2. 写操作结果不明或连接中断时通过只读能力核查，不盲目重放；部分成功分别报告实际成功、失败、未知及待处理对象<br>3. 业务已成功但展示失败时只重试展示或输出文本，不再次提交业务<br>4. 用户取消后清除待执行动作，不继续认证后的旧操作<br>5. 选定合同后可将真实流程／文件引用、原问题、来源及已读范围交给包内摘要／比对／审查／问答能力；不传凭据、不声称已读取正文 |

## 发起流程（create-flow）

- **发送确认的三个独立条件**：信息完整、用户授权、草稿检查通过，**不能互相代替**。用户一句「帮我发合同」不覆盖尚未选定的文件或未知签署人；命令返回的「请确认」业务文字也不是用户同意。
- 有缺项时用**同一份发送信息**引导补充，而不是先取得一句「确认发送」再发送发现缺项；已知缺项在调用前补齐，服务端检查只负责剩余真实业务校验。
- 修改文件／名称／签署配置必须 `draft set` 后再**不带 `--yes`** 的 `draft submit` 检查；检查通过且用户确认覆盖当前完整配置才 `--yes` 发送。
- 文件／模板／签署人集合变化或后台产生用户未见的配置变化后，**重新展示当前完整发送信息**，不沿用旧确认；用户精确补齐并说「其他不变，可以发」且实际结果无差异时，可按该授权直接发送，不再重复确认。

## 处理流程（process）

- **撤回双校验**：状态 ∈ {`填写中`／`签署中`／`合同审批中`} **且** `operations` 含 `revoke` 才执行；`--revoke-reason` 必填且绑定当前 `processId`。「作废」不当撤回试探。
- **下载**：每个流程 `operations` 须含 `download`；批量下载 `dedicatedCloudId` 须全空或同一非空；多 ID 用英文逗号拼进同一个 `--process-id`（**不是** `--process-ids`）。
- **批量签署**：整批放同一次 `process batch`，每个 ID 一个 `--process-ids`；禁止逐份 `process url`、禁止按单个 ID 循环；成功只交付一个真实 `redirectUrl`。
- 空结果区分「无数据」与「当前页空但 `total>0`」；内部 ID 只存上下文不展示；未取得 URL 不拼链接。

## 审批（approval）

- **认证不前置**：不统一前置 `approval will`，也不要求先取 `NeedAuth=false`；目标、范围、必填信息齐备后直接提交，由服务端决定认证。仅当服务端实际返回或先有有效查询明确需认证时才进认证分支；账号登录或口头确认不能替代意愿认证。
- 单条同意：用户明确「无意见」时 `--reason '无'`（**不是省略或空值**）；拒绝 `--reason '<驳回原因>'` 必填。
- 批量用 `--operate-data` 传 `[{approvalCode, taskId}]`、`--remark` 传意见或驳回原因；`total>0` 但当前页空不得说没有待审批。

## 关键区分

- **发起 vs 管理 vs 审批**：发起创建新签署流程（`draft` 命令族）；管理查询和处理已有合同流程（`process` 命令族）；审批处理合同与用印审批任务（`approval` 命令族）。草稿、合同流程、审批任务和模板的真实标识与对象类型分别保存，不同列表的序号不能混用（见 [_common/conventions.md](_common/conventions.md)）。
- **文件发送 vs 模板发送**：文件发送用真实 `file_key`；模板发送先选模板再用真实模板 ID，模板查询只作为发送的一步，不提供独立查模板入口。
- **单份签署 vs 批量签署**：明确只选一份用 `process url`；多份必须同一次 `process batch`、每个 ID 一个 `--process-ids`；不把当前页当全部，不自行剔除对象后用单份入口冒充批次成功。
- **发起确认 vs 查询/选择**：发送确认只能由用户对当前完整配置明确授权；查询结果或对象选择不替代发送确认。
- **撤回 vs 作废 vs 驳回**：撤回走 `process revoke`（状态与 `operations` 双校验）；「作废」不能当撤回试探；「驳回」按真实审批任务处理，不扩写为拒签。
- **认证前置 vs 服务端决定**：不统一前置 `approval will`，也不要求先取 `NeedAuth=false`；账号登录或口头确认不能替代意愿认证。
