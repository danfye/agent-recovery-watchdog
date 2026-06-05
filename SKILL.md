---
name: agent-recovery-watchdog
description: Diagnose, triage, and recover stalled or looping coding agents such as Codex CLI, Codex App, Claude Code, or other terminal-based AI coding tools. Use when a user says an agent is stuck, idle, looping, wasting time, not making progress, needs a nudge, needs a recovery plan, or asks to inspect a Codex Whip/Recovery Log file.
---

# Agent Recovery Watchdog

Use this skill to turn a vague "the agent is stuck" report into a concrete recovery diagnosis and next action. Stay evidence-driven: inspect local state before recommending another prompt or worker. Most real stalls are not motivation problems; they are approval, sandbox, auth, network, context, hook, MCP, or unclear-next-step problems.

## Workflow

1. Identify the target surface and failure symptom:
   - Codex CLI, Claude Code, Codex App, another terminal agent, or an unknown agent.
   - Active workspace path, if the user provides one or it can be inferred from the current directory.
   - Whether the user wants observation only, a recovery prompt, or direct intervention.
   - Visible symptom: waiting for approval, no output, repeated plan, repeated tool failure, login/auth error, network/API error, context/compact warning, MCP failure, hook failure, permission denied, or sandbox denial.

2. Gather local evidence:
   - Look for a Codex Whip recovery log at `~/Library/Application Support/Codex Whip/recovery-log.json`.
   - If the log exists, read recent entries first. Prioritize `stalled`, `unknown`, failed `foreground-whip`, failed `background-push`, command, duration, lastMessage, and error fields.
   - If no recovery log exists, inspect available terminal output, relevant logs named by the user, recent test/build output, and process hints only when practical.
   - For Codex CLI, prefer built-in state checks when visible or available: `/status` for sandbox/approval state, `/debug-config` for effective config, and the current approval/sandbox mode shown in the UI.
   - For Claude Code, prefer built-in health checks when available: `claude doctor`, `claude mcp list`, visible auth/login errors, and context/auto-compact warnings.
   - Do not assume a session is stuck solely because no output is visible; distinguish waiting, long-running execution, permission prompt, tool failure, model loop, and completed-but-unreported work.

3. Classify the failure:
   - `waiting-for-user`: user approval, permission, prompt, or input is needed.
   - `approval-or-sandbox-blocked`: command/file/network action is blocked by approval profile, sandbox, or workspace trust.
   - `auth-or-network-failure`: login, API key, rate limit, proxy, VPN, provider outage, or connectivity failure.
   - `tool-or-permission-failure`: shell, AppleScript, filesystem, executable, hook, MCP, or OS permission failure.
   - `context-pressure`: context window too full, auto-compact thrashing, huge attached file, or repeated summarization.
   - `model-loop`: repeated planning, repeated questions, or no concrete file/test action.
   - `blocked-by-context`: missing repo context, missing logs, unclear task, or conflicting instructions.
   - `long-running-but-healthy`: recent output or expected long command with no evidence of failure.
   - `completed-needs-summary`: work appears done but the agent did not report clearly.

4. Choose the least risky recovery action:
   - For approval/sandbox blocks: explain the current restriction and suggest the narrowest permission/config change; do not recommend bypassing approvals or sandboxing unless the user explicitly asks and accepts the risk.
   - For auth/network failures: fix the login, API key, proxy, VPN, rate limit, or provider status before retrying the model.
   - For permission/tool failures: fix or report the specific failing dependency before re-prompting the model.
   - For context pressure: compact, clear, split the task, reduce attached files, or start a fresh worker with a short handoff summary.
   - For MCP/hook failures: inspect MCP server status/config or hook exit output before retrying the same prompt.
   - For model loops: send a concrete continuation prompt that names the next file, command, or verification step.
   - For missing context: ask for only the minimum missing artifact, or inspect the repo/logs yourself if available.
   - For long-running healthy work: recommend waiting and specify what signal would change the diagnosis.
   - For repeated failures: recommend stopping the current agent and starting a clean worker with a narrower prompt.

5. Report in a compact incident format:
   - `Status`: one of healthy, stalled, failed, waiting, recovered, unknown.
   - `Evidence`: concrete log lines, recovery-log entries, commands, timestamps, or observed symptoms.
   - `Likely cause`: one sentence.
   - `Next action`: one concrete action, prompt, or command.
   - `Risk`: what could go wrong if the user proceeds.

## Fast Checks

Use these checks when they fit the observed tool. Do not run commands that would mutate files or credentials without approval.

Codex CLI:

```sh
codex --version
```

Inside an active Codex CLI session, ask the user to report `/status` and `/debug-config` output if sandbox or approval behavior is the issue.

Claude Code:

```sh
claude doctor
claude mcp list
```

For either tool:

```sh
pwd
git status --short
```

Use `git status --short` only to understand local changes; do not reset or revert unless the user explicitly asks.

## Recovery Prompts

When producing a prompt for a stuck coding agent, make it operational, not motivational.

Prefer:

```text
Inspect the current repository state before replying. Identify the next incomplete step, make one concrete code change or run one relevant verification command, then report exactly what changed and what remains. Do not repeat the previous plan unless you found new evidence.
```

For a loop:

```text
Stop restating the plan. Open the relevant files, choose the smallest next implementation step, apply it, and run the closest verification. If blocked, report the exact blocker with file paths or command output.
```

For a failed background worker:

```text
Review the previous worker output and error. Do not retry blindly. Explain the failure cause, inspect the affected files, then either fix the dependency or propose a narrower recovery task.
```

For approval or sandbox confusion:

```text
Before continuing, report the current approval mode, sandbox mode, workspace path, and the exact command or file operation that is blocked. Do not retry the blocked action until you explain the smallest safe permission change or alternative command.
```

For context pressure:

```text
Stop expanding the current thread. Create a concise handoff summary with: goal, completed work, current files, failing command or symptom, and the next smallest action. Then continue only from that summary.
```

## Codex Whip Recovery Log

When reading `recovery-log.json`, treat it as local evidence, not as proof by itself. Recent entries are more important than aggregate counts. Useful fields:

- `kind`: `watch`, `background-push`, `foreground-whip`, or `system`.
- `status`: `info`, `working`, `success`, or `failure`.
- `title` and `detail`: human-readable event context.
- `workspacePath`: where recovery ran.
- `durationMs`: elapsed worker time.
- `command`: background command that ran.
- `lastMessage`: final worker summary.
- `error`: failure text.

If the log path does not exist, say that no Codex Whip recovery log was found and continue with other evidence.

## Boundaries

- Do not present this skill as a replacement for official Codex App agent management.
- Do not start destructive commands, reset git state, force-push, publish, or install global tools without explicit user approval.
- Do not nudge or type into a live foreground app unless the user explicitly asks for intervention.
- Do not treat "no output" as failure without checking time, command type, and recent activity evidence.
