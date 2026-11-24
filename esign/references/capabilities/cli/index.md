# esign-cli 入口技能

本 skill 是 `esign` 命令的基础入口，主要覆盖四类高频操作：

1. 登录/登出认证
2. 租户切换与管理
3. CLI 更新与版本检查
4. API 相关命令

如果用户的问题属于这四类，优先使用本 skill，再决定是否进入更细的业务工作流。

## 使用原则

- 统一使用本地 `esign` 命令执行，不要直接调用后端接口替代 CLI。
- 真正有副作用的命令执行前，参数必须来自用户输入、CLI 输出或本地可验证结果，不要编造。
- 只想检查版本或更新状态时，优先用只读命令，不要默认执行升级。
- 如果 CLI 明确提示未安装，再走安装脚本；安装完成后先确认版本和登录状态。

## 环境选择规则

- 首次登录且用户未指定环境时，默认登录正式环境（`prod`），使用 `esign auth login`，不加 `--sandbox`。
- 已有环境选择持续有效。先通过 `esign auth status` 确认当前环境，后续操作、登录过期后的重新登录及分步登录都保持该环境；不要因为用户未再次说明环境就切回正式环境。仅检查登录态时不改变环境。
- 只有用户明确要求“切到沙箱环境”“使用沙箱登录”等，才切换或登录沙箱；只有用户明确要求“切到正式环境”“使用正式环境登录”等，才从沙箱切回正式环境。用户先前明确选择的环境在后续任务中继续有效，直到用户要求更改。
- 用户明确要求切沙箱时执行 `esign auth switch --sandbox`，明确要求切正式时执行 `esign auth switch`；目标环境未登录时，CLI 会自动触发登录。
- 正式环境遇到登录失败、未实名、权限不足、接口报错、网络异常或其他阻塞时，必须留在正式环境排查或说明阻塞原因，禁止自动切到沙箱、用沙箱重试或以重新登录沙箱绕过问题。“继续”“重试”“修复”等不代表用户要求更改环境。沙箱遇阻时同样不得自动切回正式环境。

## 1. 登录/登出认证

`esign-cli` 对外版常用认证命令：

```bash
esign auth login
esign auth login --sandbox
esign auth login url --no-browser
esign auth login code --code <auth-code>
esign auth status
esign auth switch
esign auth switch --sandbox
esign auth logout
esign auth logout --all
esign auth tenant list
esign auth tenant set <tenant-id>
```

使用建议：

- 未实名后的恢复流程优先级最高：如果上一轮 CLI 输出过“当前登录人未完成实名认证，请打开以下链接完成认证:”或“当前主体未完成实名认证，请打开以下链接完成认证:”，且用户随后回复“已实名”“已完成实名”“认证完成”“继续”等，不要执行 `esign auth login`、`esign auth login url` 或 `esign auth login code`，也不要要求用户重新授权。此时 token 可能已经保存但当前企业未设置，应直接执行 `esign auth tenant list`，再根据列表引导或执行 `esign auth tenant set <tenant-id>`；只有 `tenant list` 明确返回未登录或登录过期时，才重新走登录。
- 用户只说“登录”“检查登录态”时，先执行 `esign auth status`。
- 未登录或登录过期时，按上面的环境选择规则登录：默认正式环境使用 `esign auth login`，已选择沙箱则使用 `esign auth login --sandbox`（不要加 `--no-browser`）。CLI 会自动打开浏览器，用户完成授权后会自动完成登录，无需复制或手动输入 code。
- 只有不能拉起浏览器时，才走分步登录；两步必须使用同一环境。正式环境示例：

```bash
esign auth login url --no-browser
esign auth login code --code <auth-code>
```

- 已明确选择沙箱时，一键登录或分步登录的 `url`、`code` 两步均加 `--sandbox`；不要在失败后改变环境参数。
- 用户要切换当前企业时，先用 `esign auth tenant list` 查看可用企业，再用 `esign auth tenant set <tenant-id>` 切换。
- 如果 CLI 输出“当前登录人未完成实名认证，请打开以下链接完成认证:”或“当前主体未完成实名认证，请打开以下链接完成认证:”，直接把该 URL 提供给用户；说明当前登录/租户切换未完成，token 可能已保存，但当前企业尚未设置，需要先打开链接完成实名认证。
- 如果用户在未实名流程后回复“已实名”“已完成实名”“认证完成”“继续”等，不要直接继续原业务命令；按上面的未实名恢复流程先完成当前企业设置，成功后再继续原本的业务命令。
- 用户明确要求“退出登录”“清除登录态”“退出所有环境”“清除全部登录态”时，执行 `esign auth logout --all`；把 `--all` 作为默认登出方式。
- 用户明确要求“登出当前环境”时，执行 `esign auth logout`。
- 用户只是想确认是否已登录时，只执行 `esign auth status`，不要顺手登出。

## 2. CLI 更新与版本

版本和更新相关命令：

```bash
esign --version
esign -v
esign update check
esign update
```

使用建议：

- 只看当前版本：`esign --version`
- 只检查是否有更新：`esign update check`
- 用户明确要求升级，或 CLI 输出要求升级时，再执行：`esign update`

不要把 `esign update` 当成默认前置步骤。只有用户明确要求更新，或者排障时确实需要升级，才执行升级命令。

## CLI 不可用时

如果本机没有 `esign` 命令，可先执行安装命令：

```bash
`npm install -g @esign-ai/esign-cli`
```


安装后建议先执行：

```bash
esign --version
esign auth status
```

## 决策顺序

遇到 `esign-cli` 相关问题时，优先按下面顺序处理：

1. 先判断是“登录登出 / 更新 / API”哪一类问题
2. 先用只读命令确认状态
3. 需要变更状态时再执行有副作用命令
4. 如果仍不够，再进入更细分的业务 skill 或工作流
