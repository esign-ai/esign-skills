# e签宝 Open Skill · 合同全流程

> 基于 `SKILL.md` 约定的开放 Agent Skill，用自然语言完成 e签宝合同全流程：起草 → 审查 → 比对 → 总结 → 问答 → 发起签署 → 流程与审批管理。

底层所有业务操作统一走本地 `esign` CLI 完成，不绕过 CLI 直接拼接口。Skill 本体不绑定任何特定 Agent，只依赖通用的命令执行与文件读写能力。

---

## 能力总览

| 能力 | 能做什么 | 不含什么 |
|------|----------|----------|
| **起草** | 从零起草 / 基于底稿或粘贴正文起草、修改合同与商务法律文书，产出完整正文并生成正式 PDF | 风险审查、总结、直接发起签署 |
| **审查** | 逐条款评估风险，给出依据、影响与修改建议，交付 HTML 审查报告 | 改写正文、总结、审批签署 |
| **总结** | 输出 200–300 字事实概述 + 结构化核心信息（主体 / 标的 / 金额 / 付款 / 期限），支持多份逐份总结 | 风险判断、单条款问答 |
| **比对** | 比较两份合同，识别新增 / 删除 / 修改 / 替换，区分实质 / 非实质 / 不确定项 | 单份风险审查、改后正文 |
| **问答** | 依据合同正文回答条款问题，或依据流程信息回答「谁还没签 / 到哪一步 / 谁在审批」 | 合同检索、完整总结、完整版本比对、任何写操作 |
| **发起与管理** | 用文件或模板发起签署；查询合同与待办、获取查看 / 填写 / 签署 / 下载入口、撤回；单条及批量同意 / 拒绝审批 | 起草正文、风险判断 |
| **基础 CLI** | 登录认证、租户切换、CLI 更新 | — |

> 还内置 13 类常见合同 / 文书的起草规则（买卖、租赁、借款、劳动、服务、技术开发、委托、咨询、合作、股权转让、NDA、证明信、SaaS 服务等），命中类型时按需加载。

---

## 前置依赖

| 依赖 | 说明 |
|------|------|
| **Node.js / npm** | 用于安装 `esign` CLI |
| **esign CLI** | e签宝命令行工具，业务操作的实际执行者。安装：`npm install -g @esign-ai/esign-cli` |
| **Python 3.9+** | 仅部分校验 / 报告渲染脚本需要（`scripts/*.py`） |

验证 CLI 可用：

```bash
esign --version
esign auth status
```

---

## 安装

推荐使用 [skills CLI](https://github.com/vercel-labs/skills) 直接从 GitHub 安装，它会自动发现仓库中的 `esign/` skill：

```bash
# 安装到当前项目
npx skills add esign-ai/esign-skills

# 安装到全局（所有项目可用）
npx skills add esign-ai/esign-skills -g
```

也可以手动安装：把 `esign/` 目录放进所在 Agent 的 Skills 路径（下列以 Claude Code 的约定路径为例，其它遵循 `SKILL.md` 约定的 Agent 按其自身 Skills 目录放置）：

```bash
# 全局安装（所有项目可用）
mkdir -p ~/.claude/skills
cp -r esign ~/.claude/skills/esign

# 项目级安装（仅当前项目可用）
mkdir -p .claude/skills
cp -r esign .claude/skills/esign
```

安装后确认目录结构为 `…/skills/esign/SKILL.md`，即可通过自然语言触发。

---

## 快速开始

```bash
# 1. 安装并登录 CLI
npm install -g @esign-ai/esign-cli
esign auth login          # 默认登录正式环境；沙箱用 esign auth login --sandbox

# 2. 查看登录态 / 切换企业
esign auth status
esign auth tenant list
esign auth tenant set <tenant-id>
```

装好 Skill 后，直接用自然语言提问即可，例如：

- 「帮我审查一下这份合同有没有对我不利的条款」（→ 审查）
- 「把这份合同的主要内容总结成几个要点」（→ 总结）
- 「对方回传的版本改了哪些地方？」（→ 比对）
- 「什么时候付款？违约金怎么算？」（→ 问答）
- 「用这份合同发起签署」（→ 发起与管理）

---

## 工作原理

```
用户提问
   │
   ▼
SKILL.md 解析意图 ──► 命中 7 个能力之一
   │
   ├─ 固定多步链路? ──► 读 references/best_practices/ 对应流程
   ├─ 起草且类型已定? ──► 再读 references/types/<类型>.md
   └─ 单步能力 ──► 直接读 references/products/<能力>.md
   │
   ▼
引用 references/_common/ 公共规则 ──► 执行 esign 命令 ──► 按输出规则交付
```

**渐进加载分层（L1–L6）**：入口文件常驻，其余参考只在命中时逐层读取，不一次性预加载，避免上下文膨胀。

---

## 目录结构

```
esign/
├── SKILL.md                    # 入口：能力总览 + 意图决策树 + 危险操作确认 + 错误处理
├── references/
│   ├── products/               # 7 个能力主体与细分参考
│   ├── best_practices/         # 6 条场景 recipe（01–06）+ lite 速查 + 通用规范
│   ├── types/                  # 13 类合同 / 文书起草规则
│   └── _common/                # 登录、CLI 引导、文档处理、输出、证据、提问等公共规则
├── scripts/                    # 摘要 / 比对校验、合同报告渲染（Python）
└── assets/                     # 示例资产
```

---

## 安全与确认

- **写操作需明确确认**：发起签署（`draft submit --yes`）、撤回流程、同意 / 拒绝审批、批量操作、`auth logout --all` 等不可逆或高影响操作，执行前必须先展示操作摘要并获得用户明确同意。
- **不编造标识**：`fileKey`、`processId`、`approvalCode`、模板与企业 ID、金额、主体、日期等，一律来自用户输入或命令真实返回。
- **不记录凭据**：不在 Skill 或项目中记录、回显、迁移 token。
- **不静默重试**：写操作结果不明时先只读核查，不盲目重放。
- **不自动切环境**：登录失败、无权限时停留在当前环境排查，不自动切换正式 / 沙箱环境绕过问题。

---

## 说明

- 命令与参数以 `esign <command-path> --help` 为准；文档中的命令为便于理解的参考，冲突时以 `--help` 为准。
- 部分命令可用性取决于企业开通情况与 CLI 版本，报 `unknown command` / `unknown flag` 时以实际 `--help` 为准。
