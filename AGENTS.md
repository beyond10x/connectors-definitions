# Repository instructions for agents

## Serves

From `atlas/ROADMAP.md`:

- **O1 — governed reach.** Every definition here is data that Connectors admits, executes and
  audits; nothing here grants authority.
- **O5 — the generic agent platform.** Adding an HTTP connector is a YAML change, with no Rust build.

## Boundary

This repository holds data only: one YAML file per connector in `connectors/`, schema
`urn:beyond10x:connectors-definition:1`, auth profiles several connectors share in `auth/<name>.yaml`
(`urn:beyond10x:connectors-auth:1`), and the vendored upstream documents in `upstream/<id>/`. The importers,
the runtime, credential custody, admission, approval and audit belong to [Connectors](https://github.com/beyond10x/connectors). The decision
and design are `architecture-decision-record:provider-definitions-and-runtimes` and
`architecture-design:provider-definitions-and-runtimes` in the Connectors planning store.

## Rules

- No code: no scripts, no build, no command, binary path or shell string in a definition.
- No credentials: no token, key, client secret or password in any file. Auth profiles name flows,
  URLs and scopes only.
- Every upstream document is vendored gzipped under `upstream/<id>/`, pinned by SHA-256 in the
  definition, with URL, retrieval date, size and license in `upstream/<id>/README.md` and the
  license file beside it where the document declares one.
- Every auth profile cites the vendor's own documentation in `sources`. Do not copy endpoint URLs,
  PKCE modes or token shapes from another provider.
- Operations are selected by their upstream `operationId`; nothing the upstream document states is
  copied into a definition. `generated/` is produced by
  `task schema`. Never edit generated output by hand.
- Connectors that start a host process or use host credentials do not belong here.
- Names of other organizations, their customers, products, hosts and tickets do not enter this
  repository, its commits or its releases.

## Delivery

Every write to GitHub is `b10x-bot[bot]`: commits and pushes through `b10x-gates bot`, pull
requests and merges through `b10x-gates api`. A personal `gh` login is read-only here.

## Commands

| Command | Does |
|---|---|
| `task check` | the gate: `ess specify validate`, a drift check of `generated/`, then for every `connectors/*.yaml`: the vendored source hashing to `source.sha256`, and the file against the projected schema |
| `task schema` | regenerates `generated/` from `spec/`; commit the result |

`spec/` is the source of the definition format. `generated/` is never edited by hand. Tool versions
are in `toolchain.json`; `yq` is mikefarah v4.

Each connector file starts with `schema: urn:beyond10x:connectors-definition:1`, the projected
schema's `$id`, which `ess generate schema validate` selects by.

`task check` validates shape only. That each `op` exists in the vendored document, and that each
`{var}` in `base_url` is declared in `config`, is checked by the lowerer.
