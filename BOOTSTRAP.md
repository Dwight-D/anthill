# Bootstrap Anthill — start here (agent entrypoint)

**You are an agent installing the Anthill agent-harness framework into a
project.** This file is the single entrypoint: from here you get the framework
onto disk and then drive an interactive derivation session with the user. Read
it top to bottom and act.

Anthill harnesses agents around a long-lived project — work intake and routing
(backlog + workstreams), triage, autonomous dispatching, multi-tier
orchestration (supervisor → dispatcher → workers), and reliable upward
escalation. It is transported as **structured human language**: a set of
general-tier skills copied verbatim, plus a `.anthill/` configuration directory
derived per project. There is nothing to compile.

Installing has two halves, and the split is the whole point:

- **Mechanical** — copy the skills verbatim, scaffold the config skeleton.
  Deterministic; a tool does it so the "copy verbatim, never locally edit"
  invariant is guaranteed, not hoped for.
- **Judgment** — derive the project's workstreams, resources, and bindings with
  the user, author their always-on instruction file, and run a verification
  round-trip. Roughly an hour of decisions only the user can make. You drive it.

## Prerequisites

- A **git repository** for the target project (Anthill checks its config in
  alongside the code). If the working directory is not a git repo, stop and ask
  the user before proceeding.
- The **user present** for the derivation session.

## Step 1 — Get the mechanical scaffolding done

Pick the first branch that applies.

**A. `anthill` is installed** (check: `anthill version`):
```
anthill scaffold
```
This writes the general-tier skills verbatim, the `.anthill/` placeholder
config tree, `CLAUDE.template.md`, `.gitignore`, and the `tools/` launcher
template (which you opt into wiring per Step 5), all pinned to a known framework
version it stamps into `.anthill/framework.md`. It prints a manifest of what it
wrote. Then go to Step 2.

**B. `anthill` is not installed, but Go is** (check: `go version`):
```
go install github.com/Dwight-D/anthill-cli/cmd/anthill@latest   # or @<version> for a reproducible CLI build
anthill scaffold
```
The scaffolded framework's version is pinned regardless of how you obtained the
CLI: `scaffold` stamps the exact template ref it shipped (see `anthill version`)
into `.anthill/framework.md`. If `anthill` is not on `PATH` after install, it is
under Go's bin directory (`go env GOBIN`, else `$(go env GOPATH)/bin`). Then go
to Step 2.

**C. Neither** — do the mechanical copy by hand from the Anthill template
repository (the source of this document). Copy both tiers verbatim into the
target project: `.claude/skills/` (the nine general-tier skills) and the
`.anthill/` placeholder config tree, plus `CLAUDE.template.md`, `tools/`, and
`.gitignore`. See [`INSTALLATION.md`](INSTALLATION.md) **Step 2**. Then go to
Step 2 below.

> Whichever branch you took, the mechanical result is identical: the nine
> general-tier skills copied byte-for-byte, and a `.anthill/` tree of
> placeholder config to derive. The tool exists to make branch A's guarantee
> mechanical; it is an accelerator, never a hard dependency.

## Step 2 — Drive the derivation session

Open [`INSTALLATION.md`](INSTALLATION.md) and execute **Steps 3 through 6** with
the user:

- **Step 3** — author `.anthill/` from the target environment (workstream
  taxonomy, exclusive-resource inventory, never-auto change types, worker-cap
  derivation, sweep order). These are decisions, not copies — put the questions
  to the user in order.
- **Step 4** — help the user author their always-on instruction file from
  `CLAUDE.template.md`; keep it minimal.
- **Step 5** — optional hardening (launcher, schema-owner CLI, attention hook).
- **Step 6** — verify: config loads, an intake → triage → dispatch round-trip
  closes, and the escalation canary completes.

`INSTALLATION.md` is the authoritative procedure. This file only gets you to its
starting line and tells you which steps a tool already did for you.

## The CLI, in one line

`anthill bootstrap` prints a pointer back to this file — so a user can say
"run `anthill bootstrap` and follow the instructions" and you land here. The
full command surface is specified in [`CLI_SPEC.md`](CLI_SPEC.md).
