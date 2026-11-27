# Lite Recipe 速查（高频单步）

> 命中「核心流程已判定为 lite、直接照抄执行」的高频单步操作。参数以 `esign <command-path> --help` 为准；通用规范见 [_common/conventions.md](_common/conventions.md)。

| 意图 | 一条命令 |
|------|---------|
| 抽取正文（本地文件） | `esign document extract-contract-body --file '<文件的真实可访问路径>' --format json` |
| 抽取正文（平台已选文件） | `esign document extract-contract-body --file-key '<真实fileKey>' --format json` |
| 待我签署 | `esign sign process list --doc-query-type 5 --process-status-list 2 --format json` |
| 我发起的合同 | `esign sign process list --doc-query-type 1 --format json` |
| 待我审批 | `esign sign approval list --query-type 3 --format json` |
| 查流程状态 | `esign sign qa context --process-id '<真实processId>' --format json` |
| 流程内文件 | `esign sign qa file --process-id '<真实processId>' --format json` |
| 合同查看／签署入口 | `esign sign process url --process-id '<真实processId>'` |
| 审批入口 | `esign sign approval url --approval-code '<真实approvalCode>'` |
| 下载文件 | `esign file download --file-key '<真实fileKey>' --output-dir '<本次独立目录>'` |
| 模板候选 | `esign sign template select --flow-template-name '<关键词>' --page-num 1 --page-size 50 --format json` |

## 使用边界

- 仅用于**单步只读**速查；多步任务优先参考对应 `0N-*.md` 场景 recipe，recipe 不吻合时可直接用本表单步命令 + 产品参考自行组合，不硬套。
- 写操作（`draft submit --yes`／`process revoke`／`approval agree|refuse`／`batch-*`／`process batch`／`auth logout --all`）不在速查内直接执行，须走 [SKILL.md](../../SKILL.md) 的「危险操作确认」门禁。
- 抽取正文须核对 `code == "0"` 且 `data.content` 非空；仅有文件列表、`fileKey` 或下载链接不代表已取得正文。
