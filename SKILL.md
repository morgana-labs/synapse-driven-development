---
name: synapse-driven-development
description: Synapse-Driven Development (SDD) - author the cross-function invariant first, prove it with a point-mutation-checked test, and register it in Synapse as a durable constraint that re-surfaces to the next editor. This skill should be used when implementing a feature or bugfix, writing tests, porting code that must match a reference, defining architectural or UI-parity contracts, or when the user mentions SDD, IDD, invariants, adversarial TDD, mutation testing, edges, seeding invariants, or Synapse.
version: 0.1.0
---

# Synapse-Driven Development (SDD)

**Thesis.** TDD says write the test first. SDD says write the *invariant* first - the
cross-function, cross-file property a single test cannot see - then prove it with an adversarial,
mutation-checked test and register it in Synapse as a durable constraint that re-surfaces to
whoever edits the code next.

**The method lives in the tool.** The complete methodology - the loop, the four tiers, the
mandatory cross-seam final review, the anti-patterns - is embedded in the `synapse_cli` binary, so
this skill stays thin. It only gets the tool running; `guide` is the source of truth for the
practice. At the start of any SDD session, read and follow:

```bash
synapse_cli guide
```

## Get the tool running (do this first)

The tool is a single binary, `synapse_cli`, installed at **`~/.synapse/bin/synapse_cli`**. Pin it
and use that absolute path - do not rely on a `synapse`/`synapse_cli` on `PATH` (there may be none,
or a different build):

```bash
SYN="$HOME/.synapse/bin/synapse_cli"
```

**1. Check the version at session start.** Print the installed version and the latest published
release, then compare:

```bash
"$SYN" --version 2>/dev/null || echo "not installed"
curl -fsSL https://api.github.com/repos/morgana-labs/synapse-driven-development/releases/latest \
  | sed -n 's/.*"tag_name": *"v\{0,1\}\([^"]*\)".*/\1/p' | head -1
```

- Installed and current -> continue.
- Not installed, or the installed version is older than the latest release -> **tell the user and
  ask before installing.** Never swap the binary without the user's go-ahead. On approval, run the
  official installer (resolves the platform build, verifies its checksum, installs to
  `~/.synapse/bin`, clears the macOS quarantine flag):

  ```bash
  curl -fsSL https://github.com/morgana-labs/synapse-driven-development/releases/latest/download/install.sh | sh
  ```

  To pin a specific version, pass it: `... | sh -s -- 0.2.3`.

**2. Confirm it runs and is licensed.** The binary refuses to run without a valid `synapse.license`
file (searched: `$SYNAPSE_LICENSE_FILE`, `./synapse.license`, `<git-root>/synapse.license`,
`~/.synapse/synapse.license`):

```bash
"$SYN" doctor    # version, license status, tier, store, git root
```

If `doctor` reports the license MISSING or EXPIRED, stop and have the user obtain/renew a
`synapse.license`. Do not fall back to an unlicensed binary.

## Then follow the guide

With the tool running, `synapse_cli guide` drives the actual practice. Two entry points worth
knowing up front:

- **New project, no `docs/syn/` yet** - onboard it before applying SDD (`"$SYN" tool synapse_onboard`).
- **Existing code, no invariants yet** - `"$SYN" seed` auto-registers the gap engine's candidate
  invariants over the existing code at zero model cost, giving a codebase with none a starting
  layer of coverage. Per the guide, run it *after* authoring the deep hotspot invariants by hand
  (`"$SYN" hotspots` / `find-gaps` locate them); `seed` then fills in the rest.

Everything else - naming the invariant, choosing the tier, recording the point-mutation proof,
registering it, the drift gate (`"$SYN" check`), and the mandatory cross-seam review before a
session is "done" - is in the guide. Run it and follow it exactly.
