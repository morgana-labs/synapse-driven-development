# synapse-driven-development

A Claude Code / agent skill for **Synapse-Driven Development (SDD)**: author the cross-function
invariant first, prove it with a point-mutation-checked test, and register it in Synapse as a
durable constraint that re-surfaces to the next editor.

The methodology itself is embedded in the `synapse_cli` binary (`synapse_cli guide`), so this skill
stays thin: `SKILL.md` covers only how to get the tool running (install, version check, license,
invocation) and defers the method to `guide`. The binary is the single source of truth for the
method; the skill cannot drift from it.

> **Make SDD your default.** Installing the skill lets the agent *route* to SDD; it does not make
> SDD the default way you build. To adopt it as your standard practice, add a line to your global
> agent instructions (`~/.claude/CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, etc.) — e.g. *"For feature
> development and bug fixes, use the `synapse-driven-development` skill (invariant-first, then a
> mutation-proven test) by default; fall back to plain TDD only when `synapse_cli` isn't
> available."* That instruction, not the skill's presence, is what makes every task start with SDD.

- **The tool:** `synapse_cli` - invariant store, drift + proof gates, context retrieval, embedded
  guide. Installed at `~/.synapse/bin/synapse_cli`. Requires a `synapse.license` file.
- **Releases:** a signed, notarized, and stapled macOS `.pkg` plus an `install.sh` are published on
  this repo's [GitHub releases](https://github.com/morgana-labs/synapse-driven-development/releases).
  The skill checks the installed version against the latest release and, on request, runs the
  official installer.
- **Derived from** the original invariant-driven-development skill, slimmed once the methodology
  moved into the binary.

## Installation

A skill is a directory containing `SKILL.md`. The agent auto-discovers it once it lives in a
`skills/` location the agent scans. Use the [`skills`](https://skills.sh) CLI to install it there
from this repo — no manual cloning or path juggling.

### With the `skills` CLI (recommended)

```bash
# Global — available across all your projects
npx skills add -g morgana-labs/synapse-driven-development

# Project scope — committed with the current repo, shared with your team
npx skills add morgana-labs/synapse-driven-development
```

Verify and manage:

```bash
npx skills list        # confirm synapse-driven-development is installed
npx skills remove morgana-labs/synapse-driven-development
```

Re-running `npx skills add` updates to the latest commit.

### Manual (editable dev copy)

To hack on the skill itself, clone it and drop it into a skills directory the agent scans
(e.g. `~/.claude/skills/` for global, or `<repo>/.claude/skills/` for a project). The directory name
must match the skill's `name` (`synapse-driven-development`):

```bash
git clone https://github.com/morgana-labs/synapse-driven-development.git ~/src/sdd
ln -s ~/src/sdd ~/.claude/skills/synapse-driven-development
```

### Activation

The skill activates from its `description`: the agent routes to it automatically when the work
involves invariants, SDD/IDD, mutation testing, porting against a reference, or Synapse. No slash
command or manual trigger is required.

## First run

The skill ships only `SKILL.md` — **not** the `synapse_cli` binary. On first use it will:

1. Compare the installed `~/.synapse/bin/synapse_cli` against the latest GitHub release. If the
   binary is missing or out of date, it asks before installing, then runs the official installer:
   `curl -fsSL https://github.com/morgana-labs/synapse-driven-development/releases/latest/download/install.sh | sh`
   The installer downloads the signed + notarized + stapled macOS `.pkg`, verifies its checksum, and
   installs it to `~/.synapse/bin` with `installer -target CurrentUserHomeDirectory` — **no sudo**.
   Because the `.pkg` staples its notarization ticket, the binary lands non-quarantined and launches
   with no online Gatekeeper check (a bare binary would force an online check that can hang).
2. Require a valid `synapse.license` file (searched: `$SYNAPSE_LICENSE_FILE`, `./synapse.license`,
   `<git-root>/synapse.license`, `~/.synapse/synapse.license`). `synapse_cli doctor` reports which
   file it loaded and whether it is valid.

From there, the method is `synapse_cli guide`. See `SKILL.md` for the full session flow.

## Architecture — what's in the binary

`synapse_cli` is one self-contained binary. Everything runs in-process: no daemon, no database to
stand up, and zero footprint in your repo (`synapse_cli doctor` shows where the store actually
lives).

**Tacitus — the embedded store and engine.** Synapse's system of record is [Tacitus](https://www.libmorgana.com),
morgana labs' embedded data engine, compiled straight into the CLI (`embedded Tacitus, in-process`):

- **Write-ahead log** — an append-only, per-tenant-encrypted log the invariants live in. It is
  bi-temporal, so the store's own history is queryable with git-shaped verbs (`synapse_cli log`,
  `synapse_cli diff <A> <B>`). The store is keyed to project identity, so it survives a folder move
  or a re-clone.
- **Relational + graph/vector** — a dynamic relational catalog holds the flows, members, proofs,
  edges, and gaps, alongside an HNSW vector index for similarity and graph retrieval. This is the
  relational-and-graphing layer the invariant model is built on.
- **Context retrieval** — the engine behind `synapse_cli context`: given a file, it returns the
  invariants, mutates, assumes, and edge position that govern it — the constraints a diff must
  respect that aren't visible in the source.

**The invariant logic — the SDD engine on top of the store:**

- **Model** — invariants attach to tracked *symbols* (members) grouped into *flows*, each backed by
  a proof at one of four strengths: a type-level guarantee, a point-mutation-checked test, a
  simulation harness, or a named manual gate.
- **Drift + proof gate** (`synapse_cli check`) — hashes tracked members and re-runs the recorded
  mutation proofs, so a changed symbol or a proof that stopped reddening blocks the commit.
- **Gap engine** (`synapse_cli gap-analysis`) — sweeps the invariant graph for confirmed gaps
  (coverage, cross-seam, coupling-seam, resource-seam, dependency, dangling, throw), each carrying a
  candidate invariant and the mutation that would prove it. `hotspots` and `coupling` rank *where*
  to look, from git churn and temporal co-change.

**Axon — the cloud brain and orchestrator (paid, optional).** With a paid license the local store
becomes a cache in front of [Axon](https://www.libmorgana.com), morgana labs' cloud brain and
orchestrator — the authoritative copy a team shares. Writes dual-write to cache and brain and replay
exactly-once after any offline period, and `synapse_cli deltas` / `watch` surface teammates' new
invariants and cross-seam heads-ups live while you edit. The free tier is fully local and offline.

> **Sibling products.** Tacitus and [HyperStack](https://www.libmorgana.com) (a single-binary
> Supabase alternative — auto-generated Data API, Auth, RLS, Realtime, Storage, Edge Functions on
> Postgres) are separate morgana labs products. `synapse_cli` embeds **Tacitus**; it does not
> include HyperStack.
