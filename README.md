# Agent Recovery Watchdog

A tiny Codex skill for diagnosing stuck coding agents before blindly prompting them to "continue".

Coding agents are not always lazy when they stop making progress. They may be blocked by approval mode, sandbox policy, auth, network, MCP, hooks, context pressure, or a real model loop. This skill makes Codex classify the stall first, then recommend the smallest safe recovery action.

## What It Does

- Inspects local evidence before recommending another prompt.
- Looks for Codex Whip recovery logs when available.
- Classifies stalls into concrete failure modes.
- Produces an incident-style report: status, evidence, likely cause, next action, and risk.
- Provides recovery prompts for model loops, sandbox confusion, failed background workers, and context pressure.

## Example Diagnoses

### Sandbox Or Approval Block

Before:

```text
Continue. Make progress.
```

After:

```text
Status: waiting
Likely cause: approval-or-sandbox-blocked
Evidence: the agent is trying to perform a file or command action that requires approval.
Next action: report current approval mode, sandbox mode, workspace path, and the exact blocked operation before retrying.
Risk: blindly retrying may loop without changing the permission state.
```

### Context Pressure

```text
Status: stalled
Likely cause: context-pressure
Evidence: repeated summaries, compacting, or no concrete file/test action after a long thread.
Next action: create a short handoff summary and restart with the smallest next task.
Risk: continuing in the same thread may keep losing task state.
```

### Tool Or MCP Failure

```text
Status: failed
Likely cause: tool-or-permission-failure
Evidence: repeated MCP, hook, shell, auth, or filesystem errors.
Next action: inspect the failing tool output and fix that dependency before prompting the model again.
Risk: a stronger prompt will not fix a broken tool path.
```

## Install

Copy the skill folder into your Codex skills directory:

```sh
mkdir -p ~/.codex/skills
cp -R . ~/.codex/skills/agent-recovery-watchdog
```

Then start a new Codex session and invoke it:

```text
Use $agent-recovery-watchdog to diagnose why this coding agent appears stuck.
```

## Files

```text
agent-recovery-watchdog/
├── SKILL.md
└── agents/
    └── openai.yaml
```

## Notes

This is not a replacement for Codex App, Claude Code, or any agent command center. It is a small recovery workflow for debugging stuck local coding-agent sessions.

If you use Codex Whip, the skill can also inspect:

```text
~/Library/Application Support/Codex Whip/recovery-log.json
```

## License

MIT
