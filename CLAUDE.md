# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Anthill is an **agentic-harness framework distributed as a template**. There is no
program to build, no test suite, no lint step. The entire artifact is **structured
human language**: skill files a capable agent reads and follows, plus a
configuration directory a consumer derives for their environment. "Working with the
code" here means editing Markdown contracts, not compiling anything.

This repository is the framework's **upstream source of truth** — the template
consumers copy from, not a consumer installation. That distinction changes the
editing rules (see below).

## Two-tier architecture (the load-bearing idea)

Every part of Anthill is physically split into two tiers:

1. **General tier — `.claude/skills/`.** Eleven orchestration skills (`supervisor`,
   `autonomous`, `triage`, `submit`, `dispatch`, `dispatch-loop`,
   `dispatch-receive`, `expedite`, `escalate`, `wake-up`, `sync`). Portable roles/flows/invariants. In a
   *consumer* install these are copied **verbatim and never locally edited** —
   local divergence across installations is the exact failure mode the split
   prevents; upgrading = replacing skill files.
2. **Specific tier — `.anthill/`.** Per-project config: workstream taxonomy,
   exclusive-resource inventory, tooling bindings, launcher. A skill loads its
   config from `.anthill/` at invocation, so composition is runtime, not rewrite.

A consumer adopts Anthill by *copy the skills, derive the config*.

## Editing rules in THIS repo (upstream vs. consumer)

Because this is the upstream source, not a consumer:

- **`.claude/skills/*` are canonical source here** — editing them is legitimate
  framework maintenance. In a consumer they would be immutable. When you change a
  skill, treat it as an upstream release: it propagates to consumers by them
  re-copying the file.
- **`.anthill/*` files are placeholder templates**, not live config. They carry
  `> **PROJECT-SPECIFIC TIER — TEMPLATE.**` quote blocks and `<angle-bracket>`
  fill-ins that a consumer replaces during their derivation session. Preserve that
  template character — do not fill them with concrete values as if configuring a
  real project. The Nodachi worked examples in comments teach the *shape* of an
  adaptation and must stay marked as examples, never promoted to defaults.
- **The general tier is byte-identical across installations — no exceptions.**
  Every skill copies verbatim; nothing in `.claude/skills/*` is locally edited
  in a consumer. Per-project autonomy inputs (the proceed-list and decisions-log
  path) live in `.anthill/autonomy.md`, which the `autonomous` skill loads at
  invocation — config, not a skill edit.

## The mechanisms (what the skills orchestrate)

- **Supervisor** (`supervisor`) — user → supervisor → workers. Supervisor holds
  the narrative thread; workers are disposable executors over durable external
  state.
- **Autonomy contract** (`autonomous`) — the opt-in contract a worker brief invokes
  on line one: proceed-freely / log-and-continue / stop-and-ask.
- **Backlog** (`triage`, `dispatch`, `expedite`) — frictionless intake (title +
  value), typed workstreams, one file per item, triage → dispatch pipeline.
  Submission is dumb; all judgment lives in triage.
- **Dispatch loop** (`dispatch-loop`, `dispatch-receive`) — autonomous dispatcher
  tier: select → hand off to fresh worker → verify evidence → record → recycle.
- **Escalation** (`escalate`) — durable decision requests traveling up tiers
  (worker → dispatcher → supervisor → user) and back down as state.
- **Wake-up protocol** (`wake-up`) — the reliability contract every controller tier
  runs on each wake-up: drain signals, sweep escalations, refresh before acting.
- **Sync** (`sync`) — advance an install's general-tier skills to the CLI's
  embedded template ref via `anthill sync`, reconciling any local-edit conflict
  by reverting the skill to upstream (rescuing trapped config into `.anthill/`
  first); the consumer-side of "upgrading = replacing skill files."

## Recurring invariants (respect these when editing any contract)

- **Mode gating via skills.** Interactive one-request-at-a-time behavior is the
  baseline. Autonomy/supervision/dispatching are opt-in, entered only by invoking a
  skill — never ambient. A skill cannot self-escalate permissions; elevated modes
  are a launcher concern (`tools/supervise.*`).
- **Evidence-based done.** Every definition of done names the evidence to attach;
  controllers verify evidence, not assertions.
- **State first, signal second.** Durable artifacts (task board, backlog items,
  escalation records, agenda) carry every obligation; harness messages are
  best-effort nudges. This "lost-message invariant" is why sweeps re-read state.
- **Sender owns queue state.** Dispatch senders write all backlog/queue state;
  workers (`dispatch-receive`) never write it.
- **Disposable workers, durable state.** Any tier can be killed and rehydrated from
  external state; teardown-and-respawn is the designed lifecycle.

## Writing conventions for this repo

- **Self-contained knowledge artifacts.** Every skill/config/doc must read with
  zero prior context — state what a thing *is*, never define by contrast with a
  previous design ("unchanged from X", "no longer Y"). Contrast belongs only in
  dedicated migration/changelog sections (`.anthill/framework.md` sync log,
  `CHANGELOG.md`, `LOG.md`).
- **Flag upstream, don't fork.** A gap in the framework itself is filed as
  feedback upstream (an issue or PR on this repository), never patched by
  locally editing a skill in a consumer. Since this repo *is* the upstream
  source, fixes land directly — but keep the general/specific split intact.
- **Keep the always-on file small.** `CLAUDE.template.md` (the consumer's future
  always-on instructions) stays minimal: safety invariants, the modes-via-skills
  rule, always-on improvement flagging, backlog scope pointer. Reference material
  goes to scoped homes, not the always-on file.

## Key files to orient from

- `README.md` — the model and the two-tier principle.
- `BOOTSTRAP.md` — the agent entrypoint; the single link that starts an install
  and routes into `INSTALLATION.md`.
- `INSTALLATION.md` — the consumer adoption procedure (also the spec for what the
  `.anthill/` templates must support).
- `CLI_SPEC.md` — the `anthill` companion-CLI bootstrapping interface contract
  (bootstrap redirect, verbatim scaffold, version, doctor, sync). The runtime
  schema-owner surface (backlog/escalation) is specified separately in the CLI
  repo.
- `.anthill/framework.md` — provenance and downstream sync procedure.
- `.anthill/backlog/workstreams.md` + `bindings.md` — the backlog contract and
  per-item frontmatter schema.
- `.anthill/escalations/README.md` — the escalation record schema.
- `tools/supervise.sh` / `.ps1` — the unattended-supervisor launcher pattern.
