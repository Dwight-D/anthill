# Changelog

Upstream release notes for the Anthill framework template. Each entry states
the change and the **consumer action** a downstream install takes to adopt it
(usually: re-copy named general-tier skills verbatim, or re-derive a named
binding — `anthill sync` automates the skill re-copy). Newest first.

## Unreleased

- **Add the `rehydrate` skill (twelfth general-tier skill): supervisor
  sessions survive recycling.** The harness task board is session-scoped — it
  outlives workers but dies with the session, so "rehydrate from the task
  board" was impossible across a session boundary. `rehydrate` closes the
  gap: land workers, snapshot the board (position only) plus any in-flight
  plans to `.anthill/supervisor/board.md`, commit, end. The `supervisor`
  skill checks for that snapshot on invocation, re-creates the live board
  from it, and deletes the file — the board is the single source of truth
  during a session; the file exists only between sessions, so state never
  lives in two places. Recycling a supervisor is now as routine as recycling
  a worker: `/rehydrate`, new session, `/supervisor`. The supervisor skill's
  durable-state and agenda prose now states the board's session scope and
  the snapshot-based successor chain.
  - **Consumer action:** re-copy `.claude/skills/rehydrate/` (new directory)
    and `.claude/skills/supervisor/` verbatim (`anthill sync` covers both).
    No `.anthill/` config change; `board.md` is a runtime artifact seeded by
    the skill.

- **Rewrite the controller skills for concision; encode the yield-by-cause
  rule and worker-liveness safety.** `wake-up`, `supervisor`, and
  `dispatch-loop` are rewritten ~40% shorter in plain language, one home per
  concept (the progress ledger is now explained only in `dispatch-loop`).
  Substantive changes riding along: yield discipline is stated by cause —
  yield when out of work, never merely to report; acknowledgements and status
  land in durable files (#6). A missing or ambiguous completion signal is
  never evidence of worker death, and destructive recovery (resetting paths)
  from a suspected orphan is forbidden — escalate instead (#7, mitigation
  half). A missing control flag reads as `run` (#4, skill-side). The
  `autonomous` skill defers branch discipline to project config
  (`.anthill/resources.md` / `autonomy.md`) instead of mandating a feature
  branch. The brief template's REPORTING section now requires a
  spawner-injected resolved reply handle and durable-state-before-signal
  (#5, template half). Dropped as not worth their weight: message
  timestamp-prefixing, the push-notification rule, and the
  sequencing-dispute/reassignment micro-rules (folded into one steering
  bullet).
  - **Consumer action:** re-copy `wake-up`, `supervisor`, `dispatch-loop`,
    and `autonomous` verbatim, plus the framework-invariant
    `supervisor/brief-template.md` and `dispatch/README.md` (`anthill sync`
    covers all of these). Optionally state a branch discipline in
    `.anthill/resources.md` / `autonomy.md`.

- **Add the `sync` skill (eleventh general-tier skill).** A discoverable
  general-tier skill wrapping `anthill sync`: advance an install's `.claude/skills/`
  to the CLI's embedded template ref and reconcile any conflict. Because
  general-tier skills carry no adaptation regions (all per-project config lives in
  `.anthill/`), a sync conflict is an illegal local edit to a skill; the skill
  encodes reverting it to upstream, rescuing any trapped project config into its
  `.anthill/` home (e.g. a legacy proceed-list into `.anthill/autonomy.md`) before
  overwriting. General-tier skill count is now eleven.
  - **Consumer action:** re-copy `.claude/skills/sync/` verbatim (new directory;
    `anthill sync` copies it automatically). No `.anthill/` change required.

- **Extract autonomy config to `.anthill/autonomy.md`; the `autonomous` skill
  is now byte-identical across installs.** The proceed-list and decisions-log
  path — previously in-place adaptation regions inside the skill — move to a new
  specific-tier config file the skill loads at invocation. This removes the last
  general-tier skill that mandated local editing, eliminating the "two sanctioned
  adaptations" special case: `doctor` now checks every skill uniformly (no
  exempted regions) and `sync` is a plain verbatim re-copy (no per-skill merge,
  no adaptation-conflict class).
  - **Consumer action:** re-copy `.claude/skills/autonomous/` verbatim
    (`anthill sync`), then derive `.anthill/autonomy.md` — move your existing
    proceed-list out of the old skill copy into it (Step 3, question 7 in
    `INSTALLATION.md`). Your decisions-log path defaults to `.anthill/decisions.md`.

- **Add the `submit` skill (tenth general-tier skill).** Frictionless backlog
  intake as a discoverable, always-available skill (title + value; triage
  decides the rest). It points at the intake command in
  `.anthill/backlog/bindings.md` — no new config, no schema change. General-tier
  skill count is now ten (`supervisor`, `autonomous`, `triage`, `submit`,
  `dispatch`, `dispatch-loop`, `dispatch-receive`, `expedite`, `escalate`,
  `wake-up`).
  - **Consumer action:** re-copy `.claude/skills/submit/` verbatim (new
    directory; `anthill sync` copies it automatically). No `.anthill/` change
    required.
