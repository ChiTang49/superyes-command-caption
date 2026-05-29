# 危险模式目录

这是 SuperYes 命令字幕风险规则的首要维护面。它只为 caption 提供最低风险指引，不是执行拦截表。

## 主要学习来源

- Hermes 风格的 hardline 和 dangerous command pattern list。
- Claude/Cursor 风格的 deny、ask、approval、sensitive-action 模式。
- Codex/Gemini 风格的 sandbox、writable scope、network、execution boundary 模型。
- 真实使用中出现的模糊字幕、漏目标、漏后果案例。

## CRITICAL 模式族

可见时归为 `CRITICAL`：

- Root、home 或系统目录删除：`rm -rf /`、`rm -rf $HOME`、`Remove-Item -Recurse $HOME`、`rmdir /s C:\Users`，以及删除 `/etc`、`/usr`、`/var`、`C:\Windows` 或磁盘根目录。
- 磁盘格式化或裸块设备写入：`mkfs`、`format C:`、`diskpart`、`dd ... of=/dev/sd*`，或重定向写入 `/dev/sd*`、`/dev/nvme*` 等块设备。
- 主机关机、重启或拒绝服务：`shutdown`、`reboot`、`halt`、`poweroff`、`systemctl reboot`、`kill -1`、fork bomb。
- 密钥或凭据外传：上传 `.env`、SSH key、private key、token、`.npmrc`、`.pypirc`、`.netrc`、`.pgpass`、云凭据或 Authorization header。
- 生产级数据/资源破坏：`DROP DATABASE`、宽泛 `DROP TABLE`、`TRUNCATE`、无 `WHERE` 的 `DELETE FROM`、删除云 bucket、销毁基础设施或大规模删除远程资源。
- 密码/提权滥用：`sudo -S`、`--stdin`、`--askpass`，或疑似向提权命令管道输入猜测密码。

## HIGH 模式族

可见时至少归为 `HIGH`：

- 递归删除或大范围清理：`rm -rf`、`Remove-Item -Recurse`、`rmdir /s`、`git clean -fd`。
- 远程脚本执行：`curl ... | bash`、`wget ... | sh`、`irm ... | iex`、`iwr ... | iex`，或下载脚本后执行。
- 危险权限或所有权变更：`chmod -R 777`、`chmod -R 666`、`chmod o+w`、`chown -R root`、`icacls ... /grant Everyone:F`。
- 敏感文件写入或编辑：`/etc`、`/private/etc`、`~/.ssh`、shell rc 文件、`.env`、凭据文件、包发布配置或可能影响密钥的项目配置。
- 服务/进程中断：`systemctl stop/restart/disable/mask`、`pkill -9`、`killall -KILL`、`taskkill /F`、按正则大范围杀进程。
- find/xargs 删除：`find ... -delete`、`find ... -exec rm`、`find ... -execdir rm`、`xargs rm`。
- 内联或隐藏代码执行：`bash -c`、`sh -c`、`python -c`、`python - <<EOF`、`node -e`、`perl -e`、`ruby -e`、heredoc 输入解释器、`powershell -EncodedCommand`。
- Git 破坏性或远程历史变更：`git reset --hard`、`git clean -fd`、共享分支上的 `git rebase`、`git filter-branch`、`git push --force`、删除远程分支。
- 容器宿主风险：`docker run --privileged`、宽泛宿主挂载如 `-v /:/host`、挂载 Docker socket、host network/process namespace。
- 包管理器高风险变更：`npm audit fix --force`、带脚本的全局安装、执行未审查 lifecycle scripts 的安装命令、发布、登录/token 操作，或在当前项目外变更依赖环境。

## UNKNOWN 模式族

除非有更高可见风险，否则归为 `UNKNOWN`：

- 目标隐藏在变量、glob、subshell、命令替换、alias、函数、编码命令或生成命令字符串里。
- 链式命令中后续副作用难以解析。
- 陌生工具会写入、执行、传输数据或改变状态。
- 沙箱/权限边界不清楚。
- redirection 或 heredoc 的最终目标、解释器行为不清楚。

## 更新规则

新增模式时：

- 必须放入 `CRITICAL`、`HIGH` 或 `UNKNOWN`。
- 常见模式要补一条 smoke test。
- 保持边界：skill 只解释和提示风险，不执行策略拦截。
