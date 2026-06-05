# Posting Drafts

Use these drafts when sharing the skill in focused Codex, Claude Code, or AI coding-agent communities.

## Short GitHub Discussion Post

Title:

```text
Tiny skill for diagnosing stuck coding agents before prompting "continue"
```

Body:

```text
I kept seeing coding agents appear "stuck", but the causes were different: approval mode, sandbox policy, auth/network failures, MCP or hook errors, context pressure, or actual model loops.

I made a tiny Codex skill that forces the agent to classify the stall before retrying. It outputs:

- Status
- Evidence
- Likely cause
- Next action
- Risk

It is intentionally just a SKILL.md workflow, not a product or agent command center.

Repo: <link>

I am looking for real failure cases that this does not classify well yet.
```

## Reddit / Community Post

Title:

```text
I made a small SKILL.md for diagnosing stuck coding agents instead of blindly saying "continue"
```

Body:

```text
I noticed that when Codex/Claude Code appears stuck, my first instinct was to prompt "continue" or "make progress".

That often misses the real cause. The agent may be blocked by sandbox/approval state, auth/network issues, MCP or hook failures, context pressure, or a genuine model loop.

So I made a tiny Codex skill: agent-recovery-watchdog.

It asks Codex to inspect evidence first, classify the stall, then suggest the smallest safe recovery action.

Example:

Status: waiting
Likely cause: approval-or-sandbox-blocked
Evidence: command/file operation requires a permission state change
Next action: report approval mode, sandbox mode, workspace, and exact blocked operation before retrying
Risk: retrying blindly loops without changing anything

Repo: <link>

I would like feedback from people who have real stuck-agent cases. The most useful comments are cases this classification misses.
```

## One-Line Pitch

```text
Agent Recovery Watchdog is a tiny Codex skill that diagnoses why a coding agent is stuck before telling it to continue.
```
