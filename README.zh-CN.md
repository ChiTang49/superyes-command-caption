# SuperYes Command Caption

[English](README.md) | [简体中文](README.zh-CN.md)

![SuperYes Command Caption hero](assets/superyes-command-caption-hero.png)

**你的项目，我们的任务。按 Yes 之前，先看懂命令。**

Agent 准备执行命令时，终端通常只问一句：

```text
Do you want to proceed?
> 1. Yes
  2. Yes, and allow all access to project
  3. No
```

这个问题问的是“允不允许”，但它没有告诉你命令到底会做什么。

SuperYes Command Caption 补上这一步：命令执行前，先给一行字幕。

```text
[LOW] 在 src 目录搜索 "TODO"；只读取匹配文本；不会修改文件。
[MEDIUM] 用 npm 安装项目依赖；可能创建 node_modules 并修改 lockfile；安装脚本可能运行本地代码。
[HIGH] 递归删除 .\dist；会移除其中所有文件和子目录；删除后可能难以恢复。
```

这个仓库提供的是一组可复用的 agent 指令包。它们教 agent 在运行终端命令前，把目标、动作、影响、不确定性和风险等级讲清楚。

## 它解决的问题

很多 agent 审批流不是缺少确认按钮，而是缺少上下文。用户看到命令，却不一定知道自己正在批准什么。

SuperYes 处理的是这段很小但很关键的空白：

- 低风险命令也解释，让用户能跟上 agent 的操作；
- 删除、联网、提权、重置等高风险命令要说清后果；
- 复杂或判断不清的命令标为 `UNKNOWN`；
- 命令名、路径、flag、URL 等技术字面量保持原样。

它不是沙箱，也不是权限系统。它只负责让命令在执行前变得可读。

## 三个指令包

- `superyes-command-caption`：默认通用版，推荐大多数场景使用。它会自动跟随用户语言：中文对话输出中文命令字幕，英文对话输出英文命令字幕，混合语言按当前指令语言走。语言不明确时默认英文。
- `superyes-command-caption-CN`：固定输出中文。
- `superyes-command-caption-EN`：固定输出英文。

每个指令包都是一个完整目录：

- `SKILL.md`：核心指令。
- `agents/openai.yaml`：可选的 agent 界面元数据。
- `references/dangerous-patterns.md`：针对破坏性、提权、联网、编码、链式或陌生命令的补充规则。

## 快速安装

安装单个指令包目录，不要安装仓库根目录。仓库根目录里有 README 和图片；真正要放进 agent 的 skill 或 instruction 目录的是下面三个目录之一：

```text
superyes-command-caption
superyes-command-caption-CN
superyes-command-caption-EN
```

推荐先用默认通用版：

```text
https://github.com/ChiTang49/superyes-command-caption/tree/main/superyes-command-caption
```

固定语言版本：

```text
https://github.com/ChiTang49/superyes-command-caption/tree/main/superyes-command-caption-CN
https://github.com/ChiTang49/superyes-command-caption/tree/main/superyes-command-caption-EN
```

如果你的安装器支持 GitHub folder URL，填上面的某一个 URL 即可。手动安装时，先 clone 仓库，再只复制需要的目录：

```sh
git clone https://github.com/ChiTang49/superyes-command-caption.git
cd superyes-command-caption
cp -R superyes-command-caption <your-agent-skills-dir>/
```

Windows PowerShell：

```powershell
git clone https://github.com/ChiTang49/superyes-command-caption.git
Set-Location superyes-command-caption
$skillsDir = "<your-agent-skills-dir>"
Copy-Item -Recurse .\superyes-command-caption $skillsDir
```

如果要安装固定中文或固定英文版本，把命令里的目录名换成 `superyes-command-caption-CN` 或 `superyes-command-caption-EN`。

## 不依赖特定 agent

这些目录本质上是普通指令包。支持 skill 目录的 agent 可以直接安装整个目录；只支持 prompt 的 agent，可以导入对应的 `SKILL.md`，并在需要时读取同目录下的 `references/`。

这些指令包本身不执行命令，不批准命令，不阻止命令，不提供沙箱，也不记录审计日志。它们只规定 agent 在调用宿主环境的命令执行能力之前，应该先解释什么。

## 命令字幕约定

每次执行命令前，agent 应先判断：

- `risk`：`LOW`、`MEDIUM`、`HIGH`、`CRITICAL` 或 `UNKNOWN`
- `target`：会影响哪些文件、目录、服务、端口、远程地址或外部资源
- `operation`：读取、写入、删除、执行代码、传输数据、改权限、启动/停止进程，还是改变外部状态
- `impact`：会发生什么，或可能发生什么
- `consequence`：可能造成什么后果
- `uncertainty`：哪里判断不清

然后展示一句字幕，并执行同一条命令。不要解释 A，执行 B。

## 校验

如果你的 agent runtime 提供 skill 或 instruction-pack validator，发布或安装前建议对每个目录跑一次校验。包含中文的文件请保持 UTF-8。

## 发布前安全检查

发布修改前，建议扫描是否误提交凭据：

```sh
rg -n -i "api[_-]?key|secret|token|password|private key|authorization|bearer|ghp_|sk-|AKIA|BEGIN .*PRIVATE KEY" .
```

安全规则示例里出现这些词是正常的。真实 token、密码、私钥、个人用户名、本地路径和机器专属配置不应该提交。

## 仓库说明

这个仓库只发布 SuperYes Command Caption 指令包。它不是完整的命令执行工具、MCP server、沙箱、策略引擎或审计系统。
