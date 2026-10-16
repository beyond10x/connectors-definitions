# Repository instructions for agents

## Serves

From `atlas/ROADMAP.md`:

- **O1 — governed reach.** Every definition here is data that Connectors admits, executes and
  audits; nothing here grants authority.
- **O5 — the generic agent platform.** Adding an HTTP provider is a YAML change, with no Rust build.

## Boundary

This repository holds data only: one YAML file per provider in `providers/`, format
`connectors-provider/1`, and the vendored upstream documents in `upstream/<id>/`. The importers,
the runtime, credential custody, admission, approval and audit belong to [Connectors](https://github.com/beyond10x/connectors). The decision
and design are `architecture-decision-record:provider-definitions-and-runtimes` and
`architecture-design:provider-definitions-and-runtimes` in the Connectors planning store.

## Rules

- No code: no scripts, no build, no command, binary path or shell string in a definition.
- No credentials: no token, key, client secret or password in any file. Auth profiles name flows,
  URLs and scopes only.
- Every upstream document is vendored gzipped under `upstream/<id>/` and pinned by URL, retrieval
  date, SHA-256 and size. Its license file is vendored beside it where the document declares one.
- Every auth profile cites the vendor's own documentation in `sources`. Do not copy endpoint URLs,
  PKCE modes or token shapes from another provider.
- Operations are imported from the upstream document, not retyped. `generated/` is produced by
  `task schema`. Never edit generated output by hand.
- Providers that start a host process or use host credentials do not belong here.
- Names of other organizations, their customers, products, hosts and tickets do not enter this
  repository, its commits or its releases.

## Delivery

Every write to GitHub is `b10x-bot[bot]`: commits and pushes through `b10x-gates bot`, pull
requests and merges through `b10x-gates api`. A personal `gh` login is read-only here.

## Commands

| Command | Does |
|---|---|
| `task check` | the gate: `ess specify validate`, a drift check of `generated/`, then for every `providers/*.yaml`: file name equal to `provider`, the vendored source hashing to `upstream.sha256`, and the file against the projected schema |
| `task schema` | regenerates `generated/` from `spec/`; commit the result |

`spec/` is the source of the definition format. `generated/` is never edited by hand. Tool versions
are in `toolchain.json`; `yq` is mikefarah v4.

Data files carry `format: connectors-provider/1`. The JSON Schema `$id` is
`urn:b10x:connectors-provider:1`; `task check` sets the `schema` field the ESS validator selects by.
Do not write `schema:` into a provider file.

`task check` does not check that each `operation_id`, `method` and `path` match the vendored
document; the Connectors importer will.
