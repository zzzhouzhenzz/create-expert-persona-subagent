# Create Expert-Persona Sub-agent

An Agent Skill for delegating bounded work to temporary sub-agents modeled on exceptional real-world experts.

The central rule is simple:

> Give every sub-agent the persona of the real-world expert whose methods, standards, and judgment best fit the work.

For coding, debugging, refactoring, and code review, the required persona is **Linus Torvalds**.

## Why use it?

The skill helps a coordinating agent:

- decide whether delegation will create real leverage;
- split work without overlapping files or mutable state;
- select a meaningful expert persona rather than a generic role;
- give each sub-agent a bounded task contract;
- run independent work concurrently when safe;
- require evidence, verification, risks, and explicit blocker states;
- retain responsibility for synthesis and final verification.

Personas are used to establish task-relevant professional principles and quality standards. The skill does not encourage theatrical impersonation or invented personal opinions.

## Install

### Codex

```sh
git clone https://github.com/zzzhouzhenzz/create-expert-persona-subagent.git \
  ~/.codex/skills/create-expert-persona-subagent
```

Start a new Codex session after installation so the skill is discovered.

### Other Agent Skills runtimes

Clone or copy this repository into the runtime's skills directory. The portable behavior is defined by [`SKILL.md`](SKILL.md); [`agents/openai.yaml`](agents/openai.yaml) supplies optional Codex UI metadata and can be ignored by other runtimes.

## Use

Invoke it explicitly:

```text
Use $create-expert-persona-subagent to delegate the independent parts of this task.
```

It can also trigger automatically when a task contains independent investigations, reviews, tests, or non-overlapping implementation work suitable for temporary sub-agents.

## Persona examples

- Coding or code review: **Linus Torvalds**
- UX review: **Don Norman**
- Other domains: choose the real-world expert whose demonstrated work best matches the task's primary discipline or hardest judgment bottleneck

Machine task names remain unique and tool-safe:

```text
linus_torvalds_backend
linus_torvalds_tests
don_norman_ux_review
```

The runtime-visible name must be set through the spawn tool's explicit naming argument, such as `task_name`, `name`, `nickname`, or `label`. Putting the desired name only inside the prompt does not control an auto-generated UI nickname. The coordinating agent verifies both the canonical task name/path and every UI-visible nickname or label; a correct path does not excuse an unrelated nickname.

The spawn must also use the runtime's first-class sub-agent tool directly. Wrapping a spawn call inside a generic `exec`, shell, or code-runner invocation can create a backend child without emitting the native lifecycle event required by an activity card or Subagents panel. After spawning, the coordinating agent verifies both the returned identity and registration in the runtime's native agent list or status surface. An unregistered child is stopped rather than managed invisibly by id. The coordinator never promises that an activity card exists without observing one.

Some agent UIs collapse active workers into a count such as `1 working`. The skill therefore requires the coordinating agent to publish a named roster in the parent task after every spawn and status change:

```text
Sub-agents
- Linus Torvalds (linus_torvalds_backend) — running — implementing the backend boundary
- Don Norman (don_norman_ux_review) — completed — reviewed the interaction flow
```

The roster remains visible in the task transcript even when the host application's side panel groups or hides individual names.

## Task contract

Every delegated task specifies:

1. expert persona and task-relevant principles;
2. persona-derived runtime name bound through the spawn tool;
3. objective, scope, and explicit exclusions;
4. relevant files, facts, constraints, and prior decisions;
5. expected deliverable;
6. required verification;
7. authorized edits and owned files or state;
8. escalation conditions;
9. a structured result status: `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, or `BLOCKED`.

Reusable prompt templates are in [`references/task-contracts.md`](references/task-contracts.md).

## Files

```text
SKILL.md                     Portable skill instructions
references/task-contracts.md Delegation prompt templates
agents/openai.yaml            Optional Codex UI metadata
```
