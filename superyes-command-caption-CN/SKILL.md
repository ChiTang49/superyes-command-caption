---
name: superyes-command-caption-cn
description: Generate concise Chinese command captions before running shell, PowerShell, bash, cmd, git, npm, pip, file operation, network download, or other terminal commands. Use when an agent is about to execute any command so it explains the exact target, operation, impact, consequence, uncertainty, and advisory risk level in Chinese before running the original command.
---

# SuperYes Command Caption CN

Use this skill whenever you are about to run a terminal command and the user-facing caption should be in Chinese. Its job is to make command execution legible to the user and the host agent system before the command runs.

This skill does not provide an MCP tool, sandbox, audit log, confirmation system, or command interceptor. It only provides command explanation and risk signaling.

## Role Boundary

This skill does not decide whether a command is allowed, blocked, or requires confirmation. The host agent/system owns permission and confirmation decisions.

Treat the risk level as advisory metadata, not an enforcement policy. Do not claim that SuperYes approved, blocked, sandboxed, audited, confirmed, or executed the command.

## Required Workflow

Before writing the caption, internally identify these fields:

- `risk`: `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`, or `UNKNOWN`
- `target`: exact files, directories, URLs, services, remotes, environment, or external resources affected
- `operation`: read, write, delete, execute code, transfer data, change permissions, start/stop process, or change external state
- `impact`: what will or may change
- `consequence`: why the operation may be risky or what could go wrong
- `uncertainty`: what is unclear, if anything

Then show only a concise Chinese caption by default:

```text
[RISK] 对 <目标> 执行 <操作>；会/可能 <影响>；风险/后果：<原因或后果>。
```

Always keep command names, paths, flags, URLs, and technical terms exact. Never omit the target when the command has one.

## Caption Detail Rules

- File reads: say which file or directory is being read or searched.
- File writes: say which file or directory may be created, overwritten, appended, copied, or moved.
- File deletes: say which file or directory is deleted, whether deletion is recursive, and that deleted data may be hard to recover.
- Git commands: say whether the command reads status/history, stages files, changes branches, rewrites history, or discards work.
- Package commands: say which package manager runs, whether dependencies may be downloaded, and whether lockfiles, virtual environments, or install scripts may change state.
- Network commands: say the URL/remote host and whether data is downloaded, uploaded, or executed.
- Process/service commands: say what process, service, port, container, or background task may be started or stopped.
- Unknown commands: say what part is unclear and what consequence cannot be ruled out.

## Risk Classification Decision Tree

Use this order before choosing the advisory risk level:

1. If the command targets home, system, root, disk formatting, production-like databases, private keys/tokens, or broad external destruction, use `CRITICAL`.
2. If it deletes data, discards work, rewrites history, runs unreviewed or remote code, uploads sensitive/project data, changes permissions, mutates external resources, or affects paths outside the workspace, use at least `HIGH`.
3. If it writes inside the project, installs dependencies, builds artifacts, stages Git changes, starts local services, or downloads without executing, use at least `MEDIUM`.
4. If it is read-only, local, clear-target, and has no expected side effects, use `LOW`.
5. If parsing, target, sandbox boundary, or side effects are unclear, use `UNKNOWN` unless a clearer higher level applies.

## Risk Classification Axes

Classify by these axes:

1. Operation type: read, write, delete, execute code, transfer data, change permissions, start/stop processes, or change external state.
2. Target scope: current workspace, temporary workspace path, additional writable root, outside workspace, home directory, system/root path, remote host, or external service.
3. Target clarity: exact path/URL/command target, multiple known targets, glob expansion, variable/subshell expansion, chained command, encoded command, or unfamiliar tool.
4. Data sensitivity: ordinary source/build files, dependency metadata, environment files, credentials/tokens/private keys, user documents, database contents, or production-like data.
5. Network and remotes: no network, dependency download, arbitrary download, upload, remote shell, remote code execution, or external API/database mutation.
6. Reversibility: read-only, easy to regenerate, overwrites local files, discards uncommitted work, recursively deletes data, changes permissions, or mutates external resources.
7. Execution boundary: sandboxed to the workspace, explicitly allowed writable root, unknown boundary, or full/unrestricted host access.

Escalate risk when any axis moves outward: read to write, workspace to outside workspace, clear target to dynamic target, local to networked, reversible to destructive, or sandboxed to unrestricted.

## Mandatory Minimum Risk Rules

- Download and execute patterns such as `curl ... | bash`, `wget ... | sh`, `irm ... | iex`, or `iwr ... | iex`: at least `HIGH`.
- Deleting or recursively modifying `$HOME`, `%USERPROFILE%`, `/`, `C:\Users`, `C:\Windows`, system/root paths, or disk roots: `CRITICAL`.
- Secret exfiltration patterns such as uploading `.env`, private keys, tokens, or credential files: `CRITICAL`.
- Git commands that discard work, hard reset, clean untracked files, rewrite history, or force-push: at least `HIGH`.
- Recursive delete with a variable, glob, subshell, or unclear target: at least `UNKNOWN`; use `HIGH` if deletion is likely.
- Encoded, obfuscated, or unfamiliar commands with unclear side effects: `UNKNOWN`, or higher if a destructive/network/execute pattern is visible.

## Dangerous Pattern Catalog / 危险模式目录

详细目录在 `references/dangerous-patterns.md`。遇到破坏性、联网、提权、编码、链式、陌生或难分类命令时读取它。危险模式目录是本 skill 的首要维护面。

核心摘要：

- `CRITICAL`：home/system/root/disk 级破坏、密钥外传、生产级数据库/资源破坏、主机关机/重启/拒绝服务、密码/提权滥用。
- `HIGH`：递归删除、远程脚本执行、危险权限变更、敏感配置写入、服务/进程中断、find/xargs 删除、隐藏代码执行、Git 破坏性操作、容器宿主风险、高风险包管理器变更。
- `UNKNOWN`：动态目标、shell 展开、编码命令、陌生工具、沙箱边界不清、redirection/heredoc 行为不清、复杂链式副作用。

## Pattern Match Priority / 模式命中优先级

多个规则同时命中时，选择最高适用风险：

1. `CRITICAL` 危险模式
2. 可见的 `HIGH` 危险模式
3. 可能隐藏写入/删除/执行/传输行为的 `UNKNOWN`
4. 风险决策树
5. 通用分类轴

如果 `UNKNOWN` 与 `HIGH`/`CRITICAL` 同时适用，使用更高的可见风险，同时说明未知点。

## Limited-Scope Risk Wording / 受限范围表达

不要仅因为目标是临时目录就降低破坏性命令的风险，但要在字幕里说清楚范围受限。例如：递归删除测试目录仍是 `HIGH`，字幕应说明目标是 workspace 内临时目录。

## Sensitive Data Redaction / 敏感信息脱敏

不要在字幕里复述 secret 原值。token、password、private key、Authorization header、URL query secret、connection string、`.env` 值和凭据文件内容都应脱敏。只说明存在敏感数据，并指出受影响的文件或参数形态。

## Post-Execution Result Wording / 执行后结果措辞

字幕描述执行前的计划影响。执行后要区分计划影响与实际结果：

- 成功时，可以总结观察到的结果。
- 失败时，说命令尝试执行该操作但失败；不要暗示计划中的写入/删除/上传已经发生。
- 输出不完整或超时时，说明结果不完整。

## Shell-Specific High-Value Patterns

PowerShell:

- `Remove-Item -Recurse` or aliases such as `rm`, `del`, `erase`: usually `HIGH`; `CRITICAL` for home/system/root targets.
- `Invoke-RestMethod`/`irm` or `Invoke-WebRequest`/`iwr` piped to `Invoke-Expression`/`iex`: at least `HIGH`.
- `powershell -EncodedCommand` or `pwsh -EncodedCommand`: `UNKNOWN` or higher; explain that encoded behavior is not visible.
- `Set-Content`, `Out-File`, or `>`: `MEDIUM` for project files, higher for sensitive/system targets.

Bash/sh/zsh:

- `rm -rf`: usually `HIGH`; `CRITICAL` for home/system/root targets.
- `curl`/`wget` piped to `bash`/`sh`/`sudo bash`: at least `HIGH`.
- `sudo`, `chmod -R`, `chown -R`: at least `HIGH`, often `CRITICAL` for broad targets.
- command substitution such as `$(...)`, backticks, or unexpanded globs in destructive commands: usually `UNKNOWN` or `HIGH`.

cmd:

- `del`, `erase`, `rmdir /s`, `rd /s`: usually `HIGH`; `CRITICAL` for broad/system targets.
- `format`, `diskpart`, or boot/system modification commands: `CRITICAL`.
- `powershell -Command` or `cmd /c` chains: classify by the inner command and highest-risk step.

## Chained Commands

For chained commands using `&&`, `||`, `;`, `|`, PowerShell pipelines, or nested shells:

- classify by the highest-risk step
- mention the risky step explicitly
- do not let a low-risk step hide a later destructive, network, execution, upload, or permission-changing step
- if the chain is too complex to understand, use `UNKNOWN` and explain which part is unclear

## Domain-Specific Rules / 领域专项规则

Git:

- `git status`、`git diff`、`git log`、`git show`：通常是 `LOW`。
- `git add`、`git commit`、`git stash`：通常是 `MEDIUM`；说明暂存、提交或工作区状态变化。
- `git reset --hard`、`git clean -fd`、`git checkout -- <path>`、`git restore --source`、历史重写、force push、删除远程分支：至少 `HIGH`。
- 远程破坏性 Git 操作要说明会改变外部仓库状态。

Docker/containers:

- `docker build`：通常是 `MEDIUM`；说明镜像构建上下文和 Dockerfile。
- 不带宿主敏感参数的 `docker run`：通常是 `MEDIUM`；说明容器启动、端口和 volume。
- `--privileged`、Docker socket 挂载、host namespace、宽泛宿主挂载：至少 `HIGH`，有时 `CRITICAL`。
- 删除 image/volume/container 在可能造成数据丢失时可为 `HIGH`。

Package managers:

- 当前项目内安装依赖：通常是 `MEDIUM`；说明下载依赖、lockfile 变化和 lifecycle scripts。
- 全局安装、强制升级、`npm audit fix --force`、发布、登录/token 命令、workspace 外安装：影响状态或凭据时至少 `HIGH`。
- 不要在字幕里复述包 registry token 或 auth config 值。

## Risk Levels

`LOW`: Read-only local inspection with a clear target and no expected write, delete, upload, remote execution, permission change, or secret exposure.

`MEDIUM`: Writes or changes state inside the current project, a clearly temporary workspace path, or the local development environment, without obvious destructive behavior.

`HIGH`: May cause local data loss, discard work, run unreviewed code, transfer project or secret-like data, change permissions, alter external resources, or affect paths outside the current project.

`CRITICAL`: Could cause broad or hard-to-recover damage: home/system/root/disk paths, production-like database destruction, broad recursive permission changes, private key/token exfiltration, or external resource destruction.

`UNKNOWN`: Cannot be confidently classified because parsing, targets, sandbox boundary, variables, globs, subshells, encoded commands, unfamiliar tools, or chained side effects are unclear.

## Caption Examples

Good examples:

```text
[LOW] 在 src 目录搜索 "TODO"；只读取匹配文本；不会修改文件。
[MEDIUM] 用 npm 安装当前项目依赖；可能创建 node_modules 并修改 lockfile；安装脚本可能改变本地开发环境。
[HIGH] 递归删除 .\dist 目录；会移除其中所有文件和子目录；删除后可能难以从命令行恢复。
[CRITICAL] 删除 $HOME 目录；可能移除大量个人文件和配置；后果可能不可恢复。
[UNKNOWN] 通过变量 $target 删除路径；实际删除目标取决于变量展开；可能删除非预期文件或目录。
```

Bad examples:

```text
Bad: [HIGH] 删除文件。
Good: [HIGH] 递归删除 .\dist 目录；会移除其中所有文件和子目录；删除后可能难以从命令行恢复。

Bad: [MEDIUM] 安装依赖。
Good: [MEDIUM] 用 npm 安装当前项目依赖；可能创建 node_modules 并修改 lockfile；安装脚本可能改变本地开发环境。

Bad: [UNKNOWN] 执行复杂命令。
Good: [UNKNOWN] 执行包含变量和管道的命令；无法确定最终目标；可能产生未预期的写入、删除或代码执行。
```

## Risk Signaling

For `HIGH`, `CRITICAL`, or `UNKNOWN` commands:

- Make the caption specific about the affected target and possible consequence.
- Include the exact risky operation in plain language.
- For `HIGH`, state the plausible consequence, such as data loss, discarded work, remote code execution, credential exposure, permission changes, or external state changes.
- For `UNKNOWN`, state what is unknown, why it is unknown, and what consequence cannot be ruled out.
- Let the host agent/system decide whether to ask for confirmation, allow the command, or block it.

## Smoke Tests

After editing this skill, manually check these safe cases:

- `Get-Content .\README.md` => `LOW`, mentions exact file and read-only behavior.
- `Set-Content .\tmp.txt "x"` => `MEDIUM`, mentions create/overwrite.
- `$target = ".\tmp.txt"; Get-Content $target` => `UNKNOWN`, mentions variable target.
- `Remove-Item -Recurse .\tmp-dir` => `HIGH`, mentions recursive delete and data loss.
- `curl https://example.com/install.sh | bash` => `HIGH`, mentions download plus execution.
- `Remove-Item -Recurse $HOME` => `CRITICAL`, mentions broad personal file loss.
- `find . -name "*.tmp" -delete` => `HIGH`, mentions find-based deletion and affected search scope.
- `docker run --privileged -v /:/host image` => `HIGH` or `CRITICAL`, mentions privileged container and host filesystem access.
- `psql -c "DROP DATABASE prod"` => `CRITICAL`, mentions production-like database destruction.

## Maintenance Notes

- Keep this skill focused on explanation and risk signaling.
- Treat `Dangerous Pattern Catalog / 危险模式目录` as the highest-priority maintenance surface.
- When learning from mainstream agent safety systems, update the catalog first, then update risk rules, examples, and smoke tests.
- When changing risk rules, update examples and smoke tests.
- Run `quick_validate.py` after edits.
- Do not claim MCP, sandbox, audit logging, confirmation, approval, blocking, or execution unless an actual tool provides it.
- Keep captions concise and Chinese by default; preserve command names, paths, flags, URLs, and technical terms exactly.

## Changelog / 变更记录

- `v0.1`: minimal command caption workflow.
- `v0.2`: advisory risk axes and language selection.
- `v0.3`: production-oriented dangerous pattern catalog.
- `v0.4`: pattern priority, redaction, post-execution wording, and domain-specific rules.

## Guardrails

- Explain every command, including low-risk commands.
- The captioned command must be the command that actually runs.
- Do not analyze one command and execute a different command.
- Do not expose secrets verbatim in captions. If a command contains a token, key, or credential value, say that sensitive data is present without repeating the secret.
- If unsure, choose `UNKNOWN` and tell the user what cannot be determined.
