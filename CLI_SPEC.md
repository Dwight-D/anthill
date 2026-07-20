# `anthill` — bootstrapping CLI specification

The companion CLI for the Anthill agent-harness framework, **bootstrapping
surface**. This is the durable, self-contained contract the
[`anthill-cli`](https://github.com/Dwight-D/anthill-cli) repository builds
against for install-time and integrity operations. It states what each command
does and why; it does not prescribe Go package layout. The binary is invoked as
`anthill`.

The runtime schema-owner surface (`backlog`, `esc`) is a separate initiative
specified in that repo's `docs/CLI_INTERFACE_SPEC.md`; this document covers only
bootstrapping, integrity, and sync. The two surfaces ship in one binary — see
"Two roles of one binary" below.

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
- **Idempotent and non-destructive.** Re-running converges; nothing overwrites
  derived config or user edits without an explicit `--force`.
- **Pinned provenance.** The embedded template corresponds to a tagged upstream
  release. Every scaffold stamps that ref into `.anthill/framework.md` so `sync`
  and `doctor` have a baseline.

## The embedded template

The CLI embeds the framework template (the `.claude/skills/`, `.anthill/`,
`tools/`, `.gitignore`, and `CLAUDE.template.md` payload) at a **tagged upstream
release** via `go:embed`. This makes `scaffold` work offline and guarantees the
verbatim copy. CI in the CLI repo re-syncs the embedded payload from the
upstream `anthill` repo on each tagged release; `anthill version` reports which
upstream ref is embedded. The upstream ref is what gets stamped as
`synced-through` in `.anthill/framework.md`.

## Global conventions

Shared with the runtime surface (`docs/CLI_INTERFACE_SPEC.md` §2):

- **`--json`** on every command for machine-readable output; human tables
  otherwise.
- **Exit-code table:** 0 ok · 1 internal · 2 usage · 3 validation · 4 not-found
  · 5 conflict · 6 precondition.
- **stdout/stderr split:** results to stdout, diagnostics to stderr.
- The bootstrapping commands are **idempotent and non-destructive**: re-running
  converges, and nothing overwrites derived config or user edits without an
  explicit `--force`.

---

## Commands

### `anthill bootstrap [--open]`

Pure redirect; the headline entrypoint. Prints the canonical `BOOTSTRAP.md` URL
plus a compact agent-directed preamble ("You are installing Anthill. Read this
document, run `anthill scaffold`, then drive the derivation session with the
user."). Zero side effects — safe to run in any directory, in or out of a repo.

- **Flags.** `--open` opens the URL in the platform browser (`start` on Windows,
  `open` on macOS, `xdg-open` on Linux) instead of printing.
- **`--json`.** `{ "entrypoint": "<url>", "preamble": "<text>" }`.
- **Exit codes.** 0 always (1 only if `--open` fails to launch a browser).
- Does **not** embed the install procedure — the fetched Markdown is
  authoritative so instructions cannot drift from the binary.

### `anthill scaffold [--into <dir>] [--force] [--dry-run]`

The mechanical install. Into `<dir>` (default: current directory) writes, from
the embedded pinned template: the nine general-tier skills verbatim, the
`.anthill/` placeholder tree (quote-blocks intact) with empty runtime dirs,
`CLAUDE.template.md`, `tools/`, and `.gitignore`; then stamps
`.anthill/framework.md` `synced-through` with the embedded ref + install date.

- **Preconditions.** Must run inside a git repository (exit 6 otherwise).
- **Non-destructive rule.** For each target path: write if absent; rewrite if
  present **and byte-identical to the pinned template** (an un-derived
  placeholder or an unmodified skill — safe); **refuse** (exit 6) if present and
  **differs** from the template (already derived or user-edited) unless
  `--force`. `--force` overwrites differing files.
- **Flags.** `--into <dir>` target root; `--force` overwrite differing files;
  `--dry-run` compute and print the manifest without writing.
- **Output.** A written / skipped(identical) / refused(differs) manifest, then
  the next agent step ("now derive `.anthill/` with the user — see
  `INSTALLATION.md` Steps 3–6"). `--json`: `{ "written": [...], "skipped":
  [...], "refused": [...], "ref": "<tag>" }`.
- **Exit codes.** 0 success (including a clean `--dry-run`); 3 if any path was
  refused (differs, no `--force`); 6 not in a git repo.

### `anthill version`

Prints the CLI's own version and the embedded upstream template ref (tag/commit)
— the value an agent records as `synced-through` on a manual install.

- **`--json`.** `{ "version": "<cli-version>", "template_ref": "<tag>" }`.
- **Exit codes.** 0.

### `anthill doctor [--strict]`

One `doctor` covering both roles, reported as two labeled sections. Read-only.

**Section A — install integrity** (this spec):

- **skill integrity** — each installed `.claude/skills/*` is byte-identical to
  the embedded pinned version. Any diff is flagged as an illegal local edit to a
  general-tier skill (the exact divergence the two-tier split prevents). The two
  sanctioned `autonomous` adaptation points — the proceed-list and the
  decisions-log path — are recognized and exempted.
- **structure** — the expected `.anthill/` tree is present.
- **derivation status** — which `.anthill/` files still hold template
  quote-blocks / `<angle-bracket>` fill-ins (i.e. remain un-derived). Reported
  as information, not a hard failure, except under `--strict`.
- **sync status** — installed `synced-through` vs the embedded ref (up-to-date /
  behind).

**Section B — runtime data integrity** (from `docs/CLI_INTERFACE_SPEC.md`):
`.anthill/` discoverable, required config present, `workstreams.md` sweep-order
names existing directories, `backlog validate --strict` clean, escalation
records well-formed, no answered-but-unapplied escalations.

- **Flags.** `--strict` — exit non-zero on any skill diff, any remaining
  placeholder, or any Section B failure (CI use).
- **`--json`.** `{ "ok": bool, "checks": [ { "section", "name", "ok",
  "detail" } ] }`.
- **Exit codes.** 0 healthy; 3 on an integrity problem (skill diff, structure
  gap, Section B failure; plus remaining placeholders under `--strict`).
- Serves `INSTALLATION.md` Step 6 and ongoing discipline.

### `anthill sync [--dry-run] [--force]`

Realizes `INSTALLATION.md`'s "Sync downstream". Diffs the installed general-tier
skills against the (newer) embedded version, re-copies changed skills verbatim,
and bumps `.anthill/framework.md` `synced-through` to the embedded ref. Touches
no other `.anthill/` config.

- **Preserves the sanctioned `autonomous` adaptations** — the derived
  proceed-list and decisions-log path are never clobbered; only the surrounding
  skill text is updated. If an upstream change conflicts with those adaptation
  regions, `sync` reports the conflict and leaves the file unchanged (exit 3)
  rather than guessing — resolve manually or re-derive.
- **Flags.** `--dry-run` show the skill-level diff without applying; `--force`
  apply even when a skill has an unexpected local edit (overwrites the local
  edit — the diff is shown first).
- **`--json`.** `{ "updated": [...], "unchanged": [...], "conflicts": [...],
  "from_ref": "<old>", "to_ref": "<new>" }`.
- **Exit codes.** 0 applied (or clean `--dry-run`); 3 on an unresolved
  conflict without `--force`.

---

## Two roles of one binary

The `anthill` binary carries two distinct responsibilities; keep them
conceptually separate:

1. **Bootstrapping / integrity** (this spec) — `bootstrap`, `scaffold`,
   `version`, `doctor` §A, `sync`. Install-time and upgrade; makes the two-tier
   "copy verbatim, don't fork" discipline enforceable rather than remembered.
2. **Schema ownership** (`docs/CLI_INTERFACE_SPEC.md`) — `backlog`, `esc`, and
   `doctor` §B. Runtime; the sole writer of backlog/escalation record
   frontmatter, invariant-checking every write. Skills reference it only through
   `.anthill/backlog/bindings.md`, so adopting it is a bindings edit, not a
   skill edit.

`doctor` is the one command spanning both roles, reported as two labeled
sections.

## Out of scope

- The derivation session, `CLAUDE.md` authoring, and verification round-trip —
  agent + user judgment, driven from `INSTALLATION.md`.
- Any elevation of permissions — mode gating is a launcher concern
  (`tools/supervise.*`); the CLI never escalates a session.
- Editing general-tier skills — the CLI copies and verifies them; it never
  authors or patches them. Framework changes originate upstream and propagate by
  re-embedding at a new tagged release.
