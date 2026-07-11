---
name: create-expert-persona-subagent
description: Use when a task would benefit from one or more temporary sub-agents guided by named real-world expert personas, especially for independent investigations, reviews, tests, or non-overlapping implementation work.
---

# Create Expert-Persona Sub-agents

Use temporary sub-agents to reduce wall time and context load while keeping the parent agent accountable for scope, decisions, integration, and final verification.

## Decide whether to delegate

Delegate only a concrete, bounded subtask with a clear deliverable.

Good candidates:

- two or more independent workstreams;
- specialist research, codebase discovery, testing, or review;
- a self-contained implementation with non-overlapping files;
- work that can proceed while the parent handles integration or another task.

Keep work local when it is trivial, tightly coupled to the next decision, requires frequent shared-state coordination, or would take longer to explain than execute. Do not create a persistent agent or task for temporary work when a sub-agent is sufficient.

## Preserve parent ownership

The parent agent must retain:

- architecture and non-trivial trade-offs;
- user-intent interpretation and scope control;
- coordination of shared files and dependencies;
- synthesis of conflicting findings;
- final inspection, tests, and completion claim.

Delegate execution or evidence gathering, not accountability.

## Name the agent after the best real-world expert

Every sub-agent must have the persona of a real person recognized as exceptional in the task's primary discipline. Choose the person whose demonstrated methods, standards, and judgment best match the work—not a fictional character, generic job title, or arbitrary celebrity.

**Coding rule: every coding, debugging, refactoring, or code-review sub-agent must use the persona `Linus Torvalds`. This is mandatory.**

For non-coding work, select the best-fit real-world expert dynamically. If a task spans disciplines, choose based on its hardest judgment bottleneck. State the persona in the prompt and translate the human name into a unique machine-safe task name using lowercase letters, digits, and underscores:

- persona: `Linus Torvalds`; task name: `linus_torvalds_backend`
- persona: `Linus Torvalds`; task name: `linus_torvalds_tests`
- persona: `Don Norman`; task name: `don_norman_ux_review`

Use the persona to set professional principles and quality standards. Do not invent biographical claims, personal opinions, or theatrical imitation.

### Bind the persona to the runtime-visible name

The prompt persona and the runtime nickname are separate fields. Never assume that writing `Task name: ...` inside the prompt controls the displayed agent name.

Before spawning:

1. Inspect the spawn interface for an explicit naming field such as `task_name`, `name`, `nickname`, or `label`.
2. Set that field in the tool call to the persona-derived machine-safe name. Do not put the name only in the message.
3. After spawning, inspect the returned name, nickname, label, or canonical task path and verify that it matches the requested persona-derived name.
4. If it does not match and the runtime supports renaming, rename it before continuing.
5. If the runtime exposes neither naming nor rename control, try another supported spawn interface. If none exists, return `BLOCKED` rather than accepting an unrelated auto-generated nickname or claiming the agent was named correctly.

### Use the runtime's native visible spawn path

Invoke the runtime's first-class sub-agent or delegation tool directly so it emits the native lifecycle event used by agent lists, activity cards, and status panels.

- Do not hide a spawn call inside a generic code runner, shell/exec wrapper, or nested orchestration call merely because the underlying function is reachable there. A returned agent id is not proof that the UI registered the sub-agent.
- Use a wrapped invocation only when the runtime explicitly documents that it preserves native sub-agent lifecycle events and UI registration.
- Immediately after spawning, confirm the agent appears in the runtime's native agent list, activity panel, or status API. Record its id and verified visible name.
- If the runtime has no visual agent surface, say so. Never invent an activity-card location or promise that a card exists without observing it.
- If a direct spawn succeeds but registration is missing, manage the child by id, report the visibility limitation, and use a direct visible spawn for future work. Restart an invisible child only when duplicate execution is safe and the original child has been stopped or completed.

## Partition before spawning

Build a small dependency map. Run subtasks concurrently only when they do not depend on one another and cannot overwrite the same state. Sequence dependent work. When agents share a filesystem, assign non-overlapping files or use isolated workspaces when parallel edits could collide.

Prefer the fewest agents that expose real concurrency. Do not split one coherent problem into artificial fragments. Avoid nested delegation unless the subtask itself clearly contains independent work and concurrency limits permit it.

## Write a complete task contract

Every spawn prompt must specify:

1. real-world expert persona and task-relevant principles;
2. persona-derived runtime name, passed through the spawn tool's naming field;
3. role and objective;
4. exact scope and explicit exclusions;
5. source files, paths, commands, or facts to inspect;
6. constraints and decisions already made;
7. expected deliverable and report format;
8. verification required before reporting done;
9. whether edits are authorized and which files the agent owns;
10. escalation conditions for ambiguity, blockers, or scope changes.

Pass the minimum sufficient context. Prefer paths to large artifacts over pasted content. Do not leak the expected conclusion into independent research or review prompts. See [task-contracts.md](references/task-contracts.md) for templates.

## Spawn and coordinate

- Call the native sub-agent tool directly, give the agent a unique persona-derived runtime name through its explicit naming field, then verify both the returned name and native activity registration.
- Give each sub-agent the minimum sufficient context: no history for self-contained work, relevant recent context for local dependencies, and full history only when genuinely necessary and supported by the runtime.
- Spawn independent subtasks together, within the available concurrency limit.
- Continue useful parent work while agents run.
- Check progress through the runtime's native agent list/status control. Use a blocking wait only when the result is required for the next critical-path action; do not poll repeatedly.
- Use the runtime's messaging or follow-up controls for new context, and interrupt only when the current direction is invalid or unsafe.
- If an agent returns `NEEDS_CONTEXT`, supply the missing facts. If `BLOCKED`, change the task, dependency, scope, or approach before retrying.

## Integrate through evidence

Require each agent to return one of: `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, or `BLOCKED`, followed by changed files or findings, verification commands/results, and remaining risks.

Do not forward sub-agent output directly to the user or trust a completion claim by itself. Inspect the relevant files/diffs, reconcile overlaps and contradictions, run proportionate parent-level verification, and confirm the combined result still matches the original request.

Wait for all required agents before claiming completion. If one result becomes unnecessary, explicitly stop or disregard it and record why.

## Guardrails

- Do not broaden authority: a sub-agent inherits the task boundary, not permission for unrelated writes or external actions.
- Preserve user changes and dirty worktrees.
- Never assign two agents concurrent writes to the same files or mutable service.
- Do not use redundant agents for the same question unless independent comparison is intentional.
- Do not hide unresolved disagreement, failed verification, or missing context during synthesis.
- Do not mark the parent task complete until the integrated result is verified.
