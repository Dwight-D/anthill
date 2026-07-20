# Changelog

Upstream release notes for the Anthill framework template. Each entry states
the change and the **consumer action** a downstream install takes to adopt it
(usually: re-copy named general-tier skills verbatim, or re-derive a named
binding — `anthill sync` automates the skill re-copy). Newest first.

## Unreleased

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
