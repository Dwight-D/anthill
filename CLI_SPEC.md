# `anthill-cli` — interface specification

The companion CLI for the Anthill agent-harness framework. This is the durable,
self-contained contract the [`anthill-cli`](https://github.com/Dwight-D/anthill-cli)
repository builds against. It states what each command does and why; it does not
prescribe Go package layout.

## What the CLI is for

Anthill's install and operation are mostly **agent + user judgment**, not
automation — so the CLI is deliberately narrow. It exists to do the parts that
are purely mechanical and where determinism protects a framework invariant:

1. **Bootstrap** — hand an agent the authoritative instructions, then scaffold
   the framework onto disk *byte-identically* from a pinned release. The
   two-tier invariant ("general-tier skills are copied verbatim and never
   locally edited") is only as strong as the copy; a tool copying from an
   embedded pinned template makes it exact rather than hoped-for.
2. **Integrity & upgrade** — detect local edits to general-tier skills, and
   apply downstream syncs, turning the framework's "flag upstream, don't fork"
   and "sync downstream" discipline into commands.
3. **Schema ownership** (later phase) — be the sole writer of backlog and
   escalation frontmatter, invariant-checking every write.

The CLI is never a hard dependency: everything mechanical it does is also
documented as a manual procedure in `INSTALLATION.md`. It is an accelerator and
a guarantee, not a gate.

## Design principles

- **The doc is authoritative, the binary is thin.** Instructions live in fetched
  Markdown (`BOOTSTRAP.md`, `INSTALLATION.md`), not baked into the binary where
  they would drift. `bootstrap` redirects to the doc; it does not embed the
  procedure.
- **Mechanical only.** The CLI never makes a derivation decision. It copies,
  stamps, validates, and hands off. Judgment stays with the agent and user.
- **Idempotent and non-destructive.** Re-running a command converges; it never
  overwrites derived config or user edits without an explicit `--force`.
- **Pinned provenance.** The embedded template corresponds to a tagged upstream
  release. Every scaffold stamps that version into `.anthill/framework.md` so
  `sync` and `doctor` have a baseline.

## The embedded template

The CLI embeds the framework template (the `.claude/skills/`, `.anthill/`,
`tools/`, and `CLAUDE.template.md` payload) at a **tagged upstream release** via
`go:embed`. This makes `scaffold` work offline and guarantees the verbatim copy.
CI in the CLI repo re-syncs the embedded payload from the upstream `anthill`
repo on each tagged release; `anthill-cli version` reports which upstream ref is
embedded. The upstream ref is what gets stamped as `synced-through` in
`.anthill/framework.md`.

---

## Command surface

### Phase 1 — Bootstrap (build first)

#### `anthill-cli bootstrap [--open]`

Pure redirect; the headline command. Prints the `BOOTSTRAP.md` entrypoint
pointer plus a compact agent-directed preamble ("You are installing Anthill.
Read this, run `scaffold`, then drive the derivation session per
`INSTALLATION.md` §3–6"). **Zero side effects** — safe to run anywhere.

- `--open` opens the entrypoint URL in the default browser instead of only
  printing it.
- Exit 0 always (it is informational).

Rationale: matches the "run `anthill-cli bootstrap` and follow the instructions"
usage — the user says one line, the agent lands on the authoritative doc, and
the doc (not the binary) carries the procedure so it can never go stale.

#### `anthill-cli scaffold [--into <dir>] [--force] [--dry-run]`

The mechanical install — INSTALLATION.md **Step 2** plus the empty-skeleton part
of **Step 3**. Writes into `<dir>` (default: current directory):

- `.claude/skills/*` — the nine general-tier skills, **verbatim** from the
  embedded pinned template.
- `.anthill/` — the placeholder config tree (`framework.md`, `resources.md`,
  `decisions.md`, `supervisor/`, `backlog/`, `escalations/`) with template
  quote-blocks intact, plus empty runtime dirs (`.gitkeep`).
- `CLAUDE.template.md` and `tools/` (`supervise.sh` / `.ps1`).
- Stamps `.anthill/framework.md` `synced-through` with the embedded upstream ref
  + install date.

Behavior:

- **Refuses without `--force`** if it would overwrite a file that differs from
  the pinned template — i.e. anything the user has already derived or edited.
  Un-derived placeholders and byte-identical skills are safe to rewrite.
- Must be run inside a **git repository** (mirrors the prerequisite); refuse with
  a clear message otherwise.
- Prints a **manifest** of files written / skipped / refused, then the next
  agent step ("now drive `INSTALLATION.md` §3–6 with the user").
- `--dry-run` prints the manifest without writing.
- `--into` targets a directory other than the working directory.

#### `anthill-cli version`

Prints the CLI version and the **embedded upstream template ref** (tag/commit).
The agent uses the latter as the value to record in `.anthill/framework.md`.

### Phase 2 — Integrity & upgrade (build second)

These make the two-tier discipline enforceable rather than remembered.

#### `anthill-cli doctor [--strict]`

Verifies an existing install and reports:

- **Skill integrity** — each installed `.claude/skills/*` is **byte-identical**
  to the embedded pinned version. Any diff is flagged as an *illegal local edit
  to a general-tier skill* — the exact divergence the two-tier split exists to
  prevent. (The two sanctioned `autonomous` adaptation points — the proceed-list
  and the decisions-log path — are recognized and exempted.)
- **Structure** — the expected `.anthill/` tree exists.
- **Derivation status** — which `.anthill/` files still contain template
  quote-blocks / `<angle-bracket>` fill-ins (i.e. un-derived), so an installer
  can see what remains.
- **Sync status** — installed `synced-through` vs the embedded ref.

`--strict` exits non-zero on any skill diff or remaining un-derived placeholder
(for CI use). Serves INSTALLATION.md **Step 6** and ongoing discipline.

#### `anthill-cli sync [--dry-run] [--force]`

Realizes INSTALLATION.md "Sync downstream". Diffs the installed general-tier
skills against a newer embedded version, **re-copies changed skills verbatim**
(preserving the sanctioned `autonomous` adaptation points — never clobber the
derived proceed-list), and bumps `.anthill/framework.md` `synced-through`. Never
touches `.anthill/` config other than the sync-state stamp. `--dry-run` shows
the diff without applying.

### Phase 3 — Schema owner (design later, separate from bootstrap)

The Step-5 "schema-owner CLI": the sole writer of backlog/escalation
frontmatter, invariant-checking every write and owning id generation. Skills
reference it only through `.anthill/backlog/bindings.md`, so adopting it later is
a **bindings edit, not a skill edit**. The verb shape below matches the command
sketch already in `backlog/bindings.md` — keep them in lockstep.

```
anthill-cli backlog new --title "…" --value "…" [--source "…"] [--backlog <hint>]
anthill-cli backlog list [--workstream <ws>] [--untriaged] [--ready]
anthill-cli backlog set <id> workstream=<ws> key=value …     # workstream change moves the file
anthill-cli backlog next [--workstream <ws>]                 # default: sweep order
anthill-cli backlog claim <id>|--next [--workstream <ws>] [--force]
anthill-cli backlog close <id> --done|--discard "…"|--remove "…"|--block "…"
anthill-cli backlog validate [--strict]

anthill-cli escalation new --to <tier> --question "…" [--item <id>]
anthill-cli escalation list [--to <tier>] [--open]
anthill-cli escalation answer <id> --decision "…"
anthill-cli escalation close <id>
```

Invariants this tier must uphold (from the framework contracts):

- **Sole writer.** When installed, it is the ONLY writer of item frontmatter;
  sender skills (`dispatch`, `dispatch-loop`) call it, workers never write queue
  state.
- **Id scheme.** Filename = id = kebab slug of the title, ≤50 chars, unique
  across `intake/` and all workstream dirs, numeric suffix on collision, never
  changes once assigned (including the intake→workstream move).
- **Ready = `status: approved` with a non-empty `verify`.** `validate` enforces
  the full per-item schema in `backlog/bindings.md`.

---

## The bootstrap loop (both directions)

The CLI and the entrypoint doc bootstrap each other, so an agent reaches a
working install from either starting point.

**From the CLI** (user has it): `anthill-cli bootstrap` → prints the
`BOOTSTRAP.md` pointer → agent fetches it → doc says run `scaffold`, then derive.

**From the link** (no CLI): user hands the agent the raw `BOOTSTRAP.md` URL → its
Step 1 forks on whether `anthill-cli`/Go is present → agent installs the CLI and
runs `scaffold`, or falls back to the manual copy in `INSTALLATION.md` §2 → then
derives.

Either way the mechanical result is byte-identical and the derivation session is
the same.

## Out of scope

- The derivation session, `CLAUDE.md` authoring, and verification round-trip —
  agent + user judgment, driven from `INSTALLATION.md`.
- Any elevation of permissions — mode gating is a launcher concern
  (`tools/supervise.*`); the CLI never escalates a session.
- Editing general-tier skills — the CLI copies and verifies them; it never
  authors or patches them. Framework changes originate upstream and propagate by
  re-embedding at a new tagged release.
