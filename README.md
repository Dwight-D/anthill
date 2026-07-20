# Anthill

**An agentic harness framework — a template you adopt in your own project.**

Anthill is the framework for harnessing agents around a long-lived project:
how work is raised, cataloged, triaged, and dispatched; how agents
orchestrate other agents; and how autonomy is granted and bounded. It is
transported as **structured human language** — skills a capable agent reads
and follows, plus a configuration directory you derive for your environment —
not as a program you run.

This repository is a **template**. Clone it (or copy its two tiers into an
existing repo), then derive the project-specific layer with the user. See
[`INSTALLATION.md`](INSTALLATION.md).

## The two-tier principle

Every part of Anthill is split into two informational tiers, and the split is
**physical**:

1. **General tier — portable verbatim.** The orchestration skills in
   [`.claude/skills/`](.claude/skills). Roles, flows, invariants, and the
   reasoning behind them — they apply almost anywhere and are copied into a
   new project unchanged. Local edits to these are how installations
   diverge; don't make them.
2. **Specific tier — derived per project.** The configuration under
   [`.anthill/`](.anthill). Your work-type taxonomy, your exclusive-resource
   constraints, your tooling bindings. A skill loads its config from
   `.anthill/` on invocation, so composition is runtime, not rewrite.

Adopting Anthill ≈ *copy the skills, derive the config*. Upgrading it ≈
*replace the skill files*.

## The mechanisms

| Mechanism | Skill(s) | What it does |
|---|---|---|
| **Supervisor** | `supervisor` | User → supervisor → workers orchestration. The supervisor holds the narrative thread; workers are disposable executors over durable external state (task board, repo, `.anthill/`). |
| **Autonomy contract** | `autonomous` | The opt-in contract a worker brief invokes on line one: what to proceed on freely, what to log-and-continue, what to stop and ask. |
| **Backlog** | `triage`, `dispatch`, `expedite` | Frictionless work intake (a title + a value), typed **workstreams**, one-file-per-item, and the triage → dispatch pipeline. Submission is dumb; all judgment lives in triage. |
| **Dispatch loop** | `dispatch-loop`, `dispatch-receive` | An autonomous dispatcher tier beneath the supervisor that works the ready queue: select → hand off to a fresh worker → verify evidence → record → recycle. |
| **Escalation** | `escalate` | Durable decision requests that travel up the tiers (worker → dispatcher → supervisor → user) and back down as state — immune to signal loss. |
| **Wake-up protocol** | `wake-up` | The cross-cutting reliability contract every controller tier runs on every wake-up: drain signals, sweep escalations, refresh before acting. Encodes the lost-message invariant. |

## Core ideas that recur

- **Mode gating via skills.** Interactive behavior is the baseline; autonomy,
  supervision, and dispatching are opt-in contracts entered explicitly by
  invoking a skill — never ambient. A skill cannot self-escalate permissions;
  elevated modes are a launcher concern (see `tools/supervise.*`).
- **Evidence-based done.** Every definition of done names the evidence to
  attach. Controllers verify evidence, not assertions.
- **State first, signal second.** Durable artifacts (task board, backlog
  items, escalation records, agenda) carry every obligation; harness messages
  are best-effort nudges that only shorten latency.
- **Disposable workers, durable state.** Teardown-and-respawn is the designed
  lifecycle. Any tier can be killed and rehydrated from external state.

## Repository layout

```
.claude/skills/     GENERAL TIER — copy verbatim into an adopting project
.anthill/           SPECIFIC TIER — placeholder config templates to derive
tools/              launcher scripts (supervise.sh / .ps1)
BOOTSTRAP.md        agent entrypoint — the single link to start an install
INSTALLATION.md     the adoption guide
CLI_SPEC.md         interface contract for the anthill companion CLI
CLAUDE.template.md  starter for the project's always-on instruction file
```

To start an install, point an agent at [`BOOTSTRAP.md`](BOOTSTRAP.md) (or run
`anthill bootstrap`, which redirects there).

The `.anthill/` files in this template are **placeholders** — each marks what
to swap for your own domain concepts. The `.claude/skills/` files are the real
thing, ready to use.

## Provenance

Anthill was extracted from a working agent harness (the Nodachi Unity repo);
the worked examples throughout the config templates are that project's
bindings — they teach the *shape* of an adaptation, never content to copy.

This repository is the framework's upstream source of truth. A consuming
project records which upstream release it tracks in
[`.anthill/framework.md`](.anthill/framework.md) and upgrades by re-copying the
general-tier skills from here (or running `anthill sync`).
