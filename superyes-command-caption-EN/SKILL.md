---
name: superyes-command-caption-en
description: Generate concise English command captions before running shell, PowerShell, bash, cmd, git, npm, pip, file operation, network download, or other terminal commands. Use when an agent is about to execute any command so it explains the exact target, operation, impact, consequence, uncertainty, and advisory risk level in English before running the original command.
---

# SuperYes Command Caption EN

Use this skill whenever you are about to run a terminal command and the user-facing caption should be in English. Its job is to make command execution legible to the user and the host agent system before the command runs.

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

Then show only a concise English caption by default:

```text
[RISK] Performs <operation> on <target>; will/may <impact>; risk/consequence: <reason or consequence>.
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

- Download-and-execute patterns such as `curl ... | bash`, `wget ... | sh`, `irm ... | iex`, or `iwr ... | iex`: at least `HIGH`.
- Deleting or recursively modifying `$HOME`, `%USERPROFILE%`, `/`, `C:\Users`, `C:\Windows`, system/root paths, or disk roots: `CRITICAL`.
- Secret exfiltration patterns such as uploading `.env`, private keys, tokens, or credential files: `CRITICAL`.
- Git commands that discard work, hard reset, clean untracked files, rewrite history, or force-push: at least `HIGH`.
- Recursive delete with a variable, glob, subshell, or unclear target: at least `UNKNOWN`; use `HIGH` if deletion is likely.
- Encoded, obfuscated, or unfamiliar commands with unclear side effects: `UNKNOWN`, or higher if a destructive/network/execute pattern is visible.

## Dangerous Pattern Catalog

The detailed catalog lives in `references/dangerous-patterns.md`. Read it when a command is destructive, networked, privileged, encoded, chained, unfamiliar, or hard to classify. The catalog is the primary maintenance surface for this skill.

Core summary:

- `CRITICAL`: home/system/root/disk damage, secret exfiltration, destructive production-like database/resource operations, host shutdown/reboot/DoS, or password/privilege abuse.
- `HIGH`: recursive deletion, remote script execution, dangerous permission changes, sensitive config writes, service/process disruption, find/xargs deletion, inline hidden code execution, destructive Git operations, container host escape risk, or high-risk package manager changes.
- `UNKNOWN`: dynamic targets, shell expansion, encoded commands, unfamiliar tools, unclear sandbox boundaries, redirections/heredocs with unclear behavior, or complex chained side effects.

## Pattern Match Priority

When multiple rules apply, choose the highest applicable risk:

1. `CRITICAL` dangerous pattern
2. visible `HIGH` dangerous pattern
3. `UNKNOWN` uncertainty that could hide write/delete/execute/transfer behavior
4. risk classification decision tree
5. general risk axes

If `UNKNOWN` and `HIGH`/`CRITICAL` both apply, use the higher visible risk and still mention the uncertainty.

## Limited-Scope Risk Wording

Do not downgrade destructive commands merely because they target a temporary directory, but make the limited scope clear. Example: recursive deletion inside a test directory is still `HIGH`, with a caption that says the target is temporary and workspace-contained.

## Sensitive Data Redaction

Never repeat secrets verbatim in captions. Redact or summarize tokens, passwords, private keys, authorization headers, URL query secrets, connection strings, `.env` values, and credential file contents. Say that sensitive data is present and name the affected file or argument shape without exposing the value.

## Post-Execution Result Wording

The caption describes intended behavior before execution. After execution, distinguish planned impact from actual result:

- If the command succeeds, summarize the observed result normally.
- If it fails, say the command attempted the operation but failed; do not imply the planned write/delete/upload actually happened.
- If output is partial or timed out, say the result is incomplete.

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

## Domain-Specific Rules

Git:

- `git status`, `git diff`, `git log`, `git show`: usually `LOW`.
- `git add`, `git commit`, `git stash`: usually `MEDIUM`; mention staged files, commit creation, or worktree state.
- `git reset --hard`, `git clean -fd`, `git checkout -- <path>`, `git restore --source`, history rewrites, force push, or remote branch deletion: at least `HIGH`.
- Remote destructive Git operations should mention external repository state.

Docker/containers:

- `docker build`: usually `MEDIUM`; mention image build context and Dockerfile.
- `docker run` without host-sensitive flags: usually `MEDIUM`; mention container start, ports, and volumes.
- `--privileged`, Docker socket mounts, host namespace flags, or broad host bind mounts: at least `HIGH`, sometimes `CRITICAL`.
- Deleting images/volumes/containers can be `HIGH` when data loss is possible.

Package managers:

- Dependency install in the current project: usually `MEDIUM`; mention dependency download, lockfile changes, and lifecycle scripts.
- Global installs, forced upgrades, `npm audit fix --force`, publishing, login/token commands, or installs outside the workspace: at least `HIGH` when state or credentials may be affected.
- Never repeat package registry tokens or auth config values in captions.

## Risk Levels

`LOW`: Read-only local inspection with a clear target and no expected write, delete, upload, remote execution, permission change, or secret exposure.

`MEDIUM`: Writes or changes state inside the current project, a clearly temporary workspace path, or the local development environment, without obvious destructive behavior.

`HIGH`: May cause local data loss, discard work, run unreviewed code, transfer project or secret-like data, change permissions, alter external resources, or affect paths outside the current project.

`CRITICAL`: Could cause broad or hard-to-recover damage: home/system/root/disk paths, production-like database destruction, broad recursive permission changes, private key/token exfiltration, or external resource destruction.

`UNKNOWN`: Cannot be confidently classified because parsing, targets, sandbox boundary, variables, globs, subshells, encoded commands, unfamiliar tools, or chained side effects are unclear.

## Caption Examples

Good examples:

```text
[LOW] Searches for "TODO" in src; only reads matching text; no files are modified.
[MEDIUM] Installs current project dependencies with npm; may create node_modules and modify the lockfile; install scripts may change the local dev environment.
[HIGH] Recursively deletes .\dist; removes all files and subdirectories inside it; deleted data may be hard to recover from the command line.
[CRITICAL] Deletes $HOME; may remove many personal files and settings; the consequence may be unrecoverable.
[UNKNOWN] Deletes a path through $target; the actual deletion target depends on variable expansion; unexpected files or directories may be removed.
```

Bad examples:

```text
Bad: [HIGH] Deletes files.
Good: [HIGH] Recursively deletes .\dist; removes all files and subdirectories inside it; deleted data may be hard to recover from the command line.

Bad: [MEDIUM] Installs dependencies.
Good: [MEDIUM] Installs current project dependencies with npm; may create node_modules and modify the lockfile; install scripts may change the local dev environment.

Bad: [UNKNOWN] Runs a complex command.
Good: [UNKNOWN] Runs a command with variables and pipes; the final target cannot be determined; unexpected writes, deletes, or code execution cannot be ruled out.
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
- Treat `Dangerous Pattern Catalog` as the highest-priority maintenance surface.
- When learning from mainstream agent safety systems, update the catalog first, then update risk rules, examples, and smoke tests.
- When changing risk rules, update examples and smoke tests.
- Run `quick_validate.py` after edits.
- Do not claim MCP, sandbox, audit logging, confirmation, approval, blocking, or execution unless an actual tool provides it.
- Keep captions concise and English by default; preserve command names, paths, flags, URLs, and technical terms exactly.

## Changelog

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
