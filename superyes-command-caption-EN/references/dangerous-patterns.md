# Dangerous Patterns

This reference is the primary maintenance surface for SuperYes command-caption risk guidance. Use it as mandatory minimum risk guidance for captions, not as an enforcement blocklist.

## Sources To Learn From

- Hermes-style hardline and dangerous command pattern lists.
- Claude/Cursor-style deny, ask, approval, and sensitive-action patterns.
- Codex/Gemini-style sandbox, writable-scope, network, and execution-boundary models.
- Real user reports where command captions were vague, missed targets, or missed consequences.

## Critical Pattern Families

Classify as `CRITICAL` when visible:

- Root, home, or system deletion: `rm -rf /`, `rm -rf $HOME`, `Remove-Item -Recurse $HOME`, `rmdir /s C:\Users`, deletion under `/etc`, `/usr`, `/var`, `C:\Windows`, or disk roots.
- Disk formatting or raw block writes: `mkfs`, `format C:`, `diskpart`, `dd ... of=/dev/sd*`, redirects into `/dev/sd*`, `/dev/nvme*`, or similar block devices.
- Host shutdown, reboot, or denial of service: `shutdown`, `reboot`, `halt`, `poweroff`, `systemctl reboot`, `kill -1`, fork bombs.
- Secret or credential exfiltration: uploading `.env`, SSH keys, private keys, tokens, `.npmrc`, `.pypirc`, `.netrc`, `.pgpass`, cloud credentials, or authorization headers.
- Production-like data/resource destruction: `DROP DATABASE`, broad `DROP TABLE`, `TRUNCATE`, `DELETE FROM` without `WHERE`, deleting cloud buckets, destroying infrastructure, or deleting remote resources at scale.
- Password/privilege abuse: `sudo -S`, `--stdin`, `--askpass`, or commands that appear to pipe guessed passwords into privilege escalation.

## High Pattern Families

Classify as at least `HIGH` when visible:

- Recursive delete or broad cleanup: `rm -rf`, `Remove-Item -Recurse`, `rmdir /s`, `git clean -fd`.
- Remote script execution: `curl ... | bash`, `wget ... | sh`, `irm ... | iex`, `iwr ... | iex`, downloaded script made executable and run.
- Dangerous permission or ownership changes: `chmod -R 777`, `chmod -R 666`, `chmod o+w`, `chown -R root`, `icacls ... /grant Everyone:F`.
- Sensitive file writes or edits: `/etc`, `/private/etc`, `~/.ssh`, shell rc files, `.env`, credential files, package publishing config, or project config that may affect secrets.
- Service/process disruption: `systemctl stop/restart/disable/mask`, `pkill -9`, `killall -KILL`, `taskkill /F`, broad regex process killing.
- Find/xargs deletion: `find ... -delete`, `find ... -exec rm`, `find ... -execdir rm`, `xargs rm`.
- Inline or hidden code execution: `bash -c`, `sh -c`, `python -c`, `python - <<EOF`, `node -e`, `perl -e`, `ruby -e`, heredocs into interpreters, `powershell -EncodedCommand`.
- Git destructive or remote-history changes: `git reset --hard`, `git clean -fd`, `git rebase` on shared work, `git filter-branch`, `git push --force`, deleting remote branches.
- Container host escape risk: `docker run --privileged`, broad host bind mounts such as `-v /:/host`, Docker socket mounts, host network/process namespace.
- Package manager high-risk changes: `npm audit fix --force`, global installs with scripts, install commands that run unreviewed lifecycle scripts, publishing, login/token operations, or dependency operations outside the current project.

## Unknown Pattern Families

Classify as `UNKNOWN` unless a visible higher-risk family applies:

- Targets hidden by variables, globs, subshells, command substitution, aliases, functions, encoded commands, or generated command strings.
- Chained commands where later side effects are hard to parse.
- Unfamiliar tools that write, execute, transfer, or mutate state.
- Commands whose sandbox/permission boundary is unclear.
- Redirections or heredocs where the final target or interpreter behavior is unclear.

## Update Rule

When adding a pattern:

- Put it under `CRITICAL`, `HIGH`, or `UNKNOWN`.
- Add or update one smoke test if the pattern is common.
- Preserve the role boundary: this skill explains and signals risk; it does not enforce policy.
