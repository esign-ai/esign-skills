# 场景通用规范

> 仅服务本包的场景 recipe（[best_practices](../)）与各能力行动指南。安全门控、危险操作确认、`--format json` 等已在根 [SKILL.md](../../../SKILL.md) 中定义，此处不重复。

## 批量与并行采集

| # | 规范 |
|---|------|
| 1 | **并行取独立对象**：多个互不依赖的对象（多份合同、多个流程、多个审批任务）用 `&` 合并到同一条 Shell 命令并行执行 + `wait`，严禁逐条串行 |
| 2 | **翻页拉全**：列表接口须按原排序与筛选翻页直至无更多，不把当前页当全部；「全部」须核对筛选范围与分页 |
| 3 | **优先批量接口**：有批量接口（`process batch`／`approval batch-agree`／`approval batch-refuse`）则用批量；无则按 #1 并行 |
| 4 | **列表少轮次**：带条件搜索／列表一次采全详情；禁止无新参数时重复同一 `list` 调用 |

## 文件正文读取

- 本地可访问文件：`esign document extract-contract-body --file '<文件的真实可访问路径>'`。
- 已选平台文件：`esign document extract-contract-body --file-key '<真实fileKey>'`，取值来自 `esign sign qa file` 或文件列表，不依据名称或路径拼接。
- 两种方式择一，不重复调用；核对 `code == "0"` 且 `data.content` 非空后才作为正文，仅有文件列表、标识、下载链接或任务状态不代表已取得正文。
- 抽取产物按本轮独立工作目录保存（如 `source.txt`／`base.txt`／`compare.txt`），每份文件独立目录，不覆盖原文件、不混用旧文件。

## 字段术语与 ID 传递

> 列表返回 JSON 后，必须提取下表字段传给后续命令；禁止用其他字段替代。

| 标识 | 来源 | 传递给 |
|------|------|--------|
| `fileKey` | `esign sign qa file`／文件列表 | `esign file download --file-key`、`esign document extract-contract-body --file-key` |
| `processId` | `esign sign process list` | `process url/revoke --process-id`、`process download --process-id`（多个逗号拼接）、`process batch --process-ids`（每个 ID 一个参数）、`approval list --process-id` |
| `approvalCode` + `taskId` | `esign sign approval list` | `approval url/agree/refuse --approval-code --task-id`、`approval batch-* --operate-data` |
| `draftId` | `esign sign draft start` | `esign sign draft set/submit --draft-id` |
| `flowTemplateId` | `esign sign template select` | `esign sign draft set --flow-template-id` |
| `dedicatedCloudId` | `esign sign process list` | 批量下载前校验云归属（全空或同一非空才允许） |

**ID 边界硬约束**：

- `processId` 与 `approvalCode + taskId` 是两类标识，不得互换；从合同进入审批须用 `approval list --process-id` 取得真实任务，不把 `processId` 当审批 ID，也不按同名猜任务。
- `--process-id`（下载／撤回，单个或逗号拼接）与 `--process-ids`（批量签署，每个 ID 一个参数）不可混用。
- 内部 ID、编码、命令与日志只保存在执行上下文，不展示给普通用户；名称只用于核对已有对象，不能据名称编造路径、流程 ID 或文件标识。
- 切换账号、企业或环境后，旧身份下的 `fileKey`／流程／草稿／审批对象不能直接跨身份复用。

## 提问与合批

场景 recipe 中的必要提问统一遵循[提问与答案处理规则](../../_common/question-rules.md)：先扫描所有必要缺口，独立问题合批，依赖项待前提确定后再问；不新增「是否开始」「信息是否正确」关卡。各能力特有的可问范围见对应 recipe 的「关键区分」与产品文件。recipe 不吻合时可自行组合，但提问合批规则同样适用。
