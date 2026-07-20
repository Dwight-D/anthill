# Installing Anthill in a project

How to adopt the Anthill agent-harness framework in a new project. Written for
a **capable agent performing the installation with the user present** —
several steps derive project-specific configuration that only the user can
decide (budget ~an hour of decisions).

## What you are installing

Anthill harnesses agents around a long-lived project: work intake and routing
(backlog + workstreams), triage, autonomous dispatching, multi-tier
orchestration (supervisor → dispatcher → workers), and reliable upward
escalation. It is transported as **structured human language**: general-tier
skills you copy verbatim, plus a `.anthill/` configuration directory you
*derive* from the target environment. **Never port another project's
bindings — derive your own;** the worked examples in this template's config
files teach the shape of an adaptation, not its content.

> **Agent entrypoint.** If you arrived here from a link or from
> `anthill bootstrap`, [`BOOTSTRAP.md`](BOOTSTRAP.md) is the one-page start:
> it gets the framework onto disk (via `anthill scaffold` or a manual copy)
> and then routes you into the derivation steps below.

> **Mechanical vs. judgment.** Steps 1–2 and the *empty skeleton* of Step 3 are
> **mechanical** — a verbatim copy plus scaffolding, which `anthill scaffold`
> does deterministically (see [`CLI_SPEC.md`](CLI_SPEC.md)). Steps 3–6 are
> **judgment** — decisions only the user can make. If the CLI already scaffolded,
> skip to Step 3 and derive; the copy in Step 2 is done.

## Prerequisites

- An agent harness with: skill/command files loaded on invocation, spawnable
  subagents that inherit the parent session's permission mode, and (for the
  supervisor tier) background/named agents with messaging. Claude Code
  satisfies all of this.
- A git repository for the project.
- The user, for the derivation session.

## Source of truth

This repository is the framework's upstream source of truth — everything you
need to install and reason about Anthill is in it:

1. **The general-tier skills in `.claude/skills/`** — the live, canonical
   texts and the reasoning behind them, complete enough to author every
   artifact. Copy them verbatim; they run unmodified in any project.
2. **The `.anthill/` placeholder config** — the shape of every artifact you
   derive, with worked examples marked as illustrative.
3. **This guide and [`README.md`](README.md)** — the model and the procedure.

Where a template's embedded sample and a live skill disagree, the live skill
wins (samples are illustrative snapshots).

## Step 1 — Read the framework

Read [`README.md`](README.md) for the model, then skim each skill in
`.claude/skills/`. You need the whole model before deriving config, because
the pieces share concepts (evidence-based done, exclusive resources,
never-auto types, the sender-owns-queue-state rule, the lost-message
invariant).

## Step 2 — Install the general-tier skills (verbatim)

> **Mechanical — `anthill scaffold` does this.** If the CLI already
> scaffolded, this copy is done and byte-identical; verify with
> `anthill doctor` and move to Step 3. Do it by hand only on the manual path.

Copy the ten skills from this template's `.claude/skills/` into the new
project's skills directory:

| Skill | Role |
|---|---|
| `supervisor` | Orchestration tier contract (opt-in, never ambient) |
| `autonomous` | The autonomy contract worker briefs invoke on line one |
| `triage` | Dedup → route to workstream → classify per profile; proposes, never implements |
| `submit` | Frictionless backlog intake (title + value); any mode, never triages or approves |
| `dispatch` | Sender/handoff: claim → brief → spawn worker → verify evidence → close |
| `dispatch-loop` | Autonomous dispatcher tier (agent-only) |
| `dispatch-receive` | Worker contract (agent-only; first line of dispatch worker briefs) |
| `expedite` | User-invoked triage+approve fast lane |
| `escalate` | Durable escalation records: raise / receive / answer / apply |
| `wake-up` | Controller wake-up protocol; referenced by supervisor/dispatch-loop/escalate |

**All ten skills copy verbatim — no exceptions.** There is nothing to edit in a
skill during install. The one thing the `autonomous` skill needs per project —
its proceed-list and decisions-log path — lives in `.anthill/autonomy.md`, which
the skill loads at invocation; you derive that file in Step 3 like any other
`.anthill/` config. **Local edits to general-tier skills are how installations
diverge; don't.** Adaptations belong in `.anthill/`.

Not part of Anthill: any project-specific skills (compile checks, domain
tooling). The adopting project references its own equivalents as *dispatch
routes* and *evidence commands* inside its config.

## Step 3 — Derivation session (with the user): author `.anthill/`

This template ships `.anthill/` as placeholder config. Derive each file in
conversation with the user. Put these questions to them, in order:

1. **What is the *product* workstream called, and what belongs in it?**
   Rename `product` everywhere: the `workstreams.md` frontmatter + heading,
   and the directory `.anthill/backlog/product/`.
2. **What exclusive resources exist?** (a shared checkout, a build server, an
   editor/IDE, a device, a database…) and what are each one's health states +
   remedies + lease rules? → `resources.md`.
3. **What headless verification exists?** It becomes the evidence
   requirements. If none exists, flag building some as the first `dev`
   workstream item. → `supervisor/bindings.md`, `workstreams.md`.
4. **Which change types are this domain's "permanent, cross-cutting, or
   taste-laden" ones?** They become the workstreams' **never-auto** types.
5. **What is the worker-cap derivation, and the dispatch parallelism posture?**
   Both fall out of the resource inventory. → `resources.md`.
6. **What is the sweep order?** → `workstreams.md` frontmatter.
7. **What may an autonomous worker do without asking?** The concrete
   proceed-list, in the project's real skills and commands — every entry a
   routine action the safety invariants do not gate. → `autonomy.md`.

Runtime artifacts (`agenda.md`, `intake/`, the workstream dirs, both
`CHANGELOG.md`/`LOG.md`, `decisions.md`) start empty; the agenda seeds from
the user's first briefing.

## Step 4 — Always-on instruction file (user-owned)

The project's always-on agent instructions (CLAUDE.md or equivalent) are the
user's territory. [`CLAUDE.template.md`](CLAUDE.template.md) is a starting
point — help the user author the minimal Anthill-relevant set: **safety
invariants** (destructive-action ask-firsts, config-file ownership, anything
irreversible in this domain), the rule that **elevated modes are entered only
via skills, never ambient**, the **always-on improvement-flagging directive**
(submit friction to the backlog — a title + value is enough), and a scope
pointer to `.anthill/backlog/`. Keep it small.

## Step 5 — Optional hardening

- **Launcher** (unattended supervisor): the `tools/supervise.sh` / `.ps1`
  templates ship with the scaffold; opt into them here. They start a session
  with elevated permissions + remote steering + `/supervisor <mission>` —
  required because skills cannot self-escalate (harness design) and child agents
  inherit the launched mode. Adjust for your environment and name the launcher
  in `supervisor/bindings.md`.
- **Schema-owner CLI**: when a backlog/escalation CLI is available, install it
  and set it as the schema owner in `backlog/bindings.md`. Until then, operate
  convention-first with care — skills reference the schema owner only through
  bindings, so the swap later is a bindings edit, not a skill edit.
- **User-attention hook**: a harness hook surfacing the count of open
  escalations addressed to `user` at session start.

## Step 6 — Verify the installation

1. **Config loads.** Invoke `supervisor` interactively — it must read bindings
   + agenda and, on a fresh install, correctly identify anything missing
   rather than proceeding.
2. **Intake → triage → dispatch round trip.** Submit a trivial real
   improvement (title + value), triage it (routed, classified per profile),
   have the user approve it, `dispatch` it; confirm the worker report's
   evidence is verified and the item closes into the changelog.
3. **The escalation canary (mandatory).** The user approves a deliberately
   unimplementable item; dispatch it. Pass criteria: the worker returns an
   escalation record inline; the sender persists it to `.anthill/escalations/`
   and blocks the item; the record reaches the user with the **Question
   verbatim**; after the user writes `## Decision`, the next dispatch tick
   applies it back to the item. Harness message channels may flatten (a nested
   agent's message may reach the user directly) — that is tolerated; the
   record and sweeps are the reliable layer being tested.

## Ongoing discipline

- **Two-tier.** General skills stay byte-identical to upstream; every
  adaptation lives in `.anthill/`. Upgrading the framework = replacing skill
  files.
- **Flag upstream, don't fork.** Gaps or findings about the framework itself
  are filed upstream to the Anthill repository (an issue or PR) — never fixed
  via local skill edits. An urgently needed local mitigation goes into
  `.anthill/` config and is named in the upstream report so the maintainer can
  supersede it.
- **Sync downstream.** Periodically, re-copy the general-tier skills from the
  upstream Anthill repository at a newer release than `.anthill/framework.md`'s
  `synced-through` ref, then bump the ref. `anthill sync` does this
  mechanically.
- **Posture.** Start propose-only (triage recommends, the user approves);
  loosen per workstream as the gates earn trust.
