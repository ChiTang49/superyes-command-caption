# SuperYes Command Caption

[English](README.md) | [简体中文](README.zh-CN.md)

![SuperYes Command Caption hero](assets/superyes-command-caption-hero.png)

**Managed Democracy for Agent Commands.**

Your agent wants to run a command. The prompt says:

```text
Do you want to proceed?
> 1. Yes
  2. Yes, and allow all access to project
  3. No
```

That prompt is technically asking for consent. It is not telling you much.

SuperYes Command Caption adds the missing line: a short caption before the command runs.

```text
[LOW] Searches for "TODO" in src; only reads matching text; no files are modified.
[MEDIUM] Installs project dependencies with npm; may create node_modules and modify the lockfile; install scripts may run local code.
[HIGH] Recursively deletes .\dist; removes all files and subdirectories inside it; deleted data may be hard to recover.
```

This repository ships reusable instruction packs for AI coding agents. The packs teach an agent to describe terminal commands in terms a human can act on: target, operation, impact, uncertainty, and risk level.

## The Problem

Agent approval flows usually focus on whether a command is allowed to run. The harder question is whether the user understands what they are approving.

SuperYes keeps that loop small:

- explain low-risk commands too, so the user can follow the work;
- call out destructive, networked, privileged, or ambiguous commands;
- mark unclear commands as `UNKNOWN` instead of pretending;
- preserve command names, paths, flags, URLs, and other technical literals.

It is not a sandbox or policy engine. It is a command-caption layer.

## Packs

- `superyes-command-caption`: Default pack. It follows the user's language automatically: Chinese prompts get Chinese captions, English prompts get English captions, and mixed-language sessions follow the instruction language. If the language is unclear, it prefers English.
- `superyes-command-caption-CN`: Always use Chinese captions.
- `superyes-command-caption-EN`: Always use English captions.

Each pack is a complete folder:

- `SKILL.md`: the main instructions.
- `agents/openai.yaml`: optional agent interface metadata.
- `references/dangerous-patterns.md`: extra rules for destructive, privileged, networked, encoded, chained, or unfamiliar commands.

## Quick Install

Install one pack folder, not the repository root. The root contains README files and images; the pack folder is what belongs in your agent's skill or instruction directory.

Most users should start with the default pack:

```text
https://github.com/ChiTang49/superyes-command-caption/tree/main/superyes-command-caption
```

Use a fixed-language pack only when the workflow requires it:

```text
https://github.com/ChiTang49/superyes-command-caption/tree/main/superyes-command-caption-CN
https://github.com/ChiTang49/superyes-command-caption/tree/main/superyes-command-caption-EN
```

If your installer accepts a GitHub folder URL, use one of the URLs above. For manual installation, clone the repo somewhere temporary and copy only the pack you want:

```sh
git clone https://github.com/ChiTang49/superyes-command-caption.git
cd superyes-command-caption
cp -R superyes-command-caption <your-agent-skills-dir>/
```

Windows PowerShell:

```powershell
git clone https://github.com/ChiTang49/superyes-command-caption.git
Set-Location superyes-command-caption
$skillsDir = "<your-agent-skills-dir>"
Copy-Item -Recurse .\superyes-command-caption $skillsDir
```

Replace `superyes-command-caption` with `superyes-command-caption-CN` or `superyes-command-caption-EN` if you want a fixed-language pack.

## Using It Outside Skill-Aware Agents

The packs are plain instruction bundles. If your agent supports skills, install the full pack folder. If it only supports prompts, import the relevant `SKILL.md` and keep the `references/` folder available for cases where the agent needs more detail.

The packs do not execute, approve, block, sandbox, or audit commands. They only define what the agent should explain before using whatever command runner the host already provides.

## Caption Contract

Before running a command, the agent should identify:

- `risk`: `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`, or `UNKNOWN`
- `target`: files, directories, remotes, services, ports, environments, or external resources affected
- `operation`: read, write, delete, execute code, transfer data, change permissions, start or stop processes, or change external state
- `impact`: what will or may change
- `consequence`: what could go wrong
- `uncertainty`: what is unclear

Then it shows one concise caption and runs the same command it explained.

## Validate

If your agent runtime provides a validator for skills or instruction packs, run it against each pack folder before publishing or installing. Keep UTF-8 enabled for files that contain Chinese text.

## Safety Check

Before publishing changes, scan for accidental credentials:

```sh
rg -n -i "api[_-]?key|secret|token|password|private key|authorization|bearer|ghp_|sk-|AKIA|BEGIN .*PRIVATE KEY" .
```

Matches inside safety examples are expected. Real credentials, personal usernames, local workspace paths, and machine-specific setup paths must not be committed.

## Repository Notes

This repository publishes the instruction packs only. It is not the full SuperYes command execution tool, MCP server, sandbox, policy engine, or audit system.
