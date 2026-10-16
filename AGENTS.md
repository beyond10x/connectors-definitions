# Repository instructions for agents

## Serves

From `atlas/ROADMAP.md`:

- **O1 — governed reach.** Every definition here is data that Connectors admits, executes and
  audits; nothing here grants authority.
- **O5 — the generic agent platform.** Adding an HTTP provider is a YAML change, with no Rust build.

## Boundary

This repository holds data only: one authored YAML file per provider in `providers/`, and pinned
upstream references. The format, the lowering frontends, the runtime, credential custody, admission,
approval and audit belong to [Connectors](https://github.com/beyond10x/connectors). The decision
and design are `architecture-decision-record:provider-definitions-and-runtimes` and
`architecture-design:provider-definitions-and-runtimes` in the Connectors planning store.

## Rules

- No code: no scripts, no build, no command, binary path or shell string in a definition.
- No credentials: no token, key, client secret or password in any file. Auth profiles name flows,
  URLs and scopes only.
- Every upstream document is pinned by URL and SHA-256. Vendor bytes are committed only where the
  license allows it, and the license is recorded beside the pin.
- Every auth profile cites the vendor's own documentation in `sources`. Do not copy endpoint URLs,
  PKCE modes or token shapes from another provider.
- Generated output (lowered JSON, pages, `index.json`) is produced by CI from a pinned `connectors`
  release. Never edit it by hand.
- Providers that start a host process or use host credentials do not belong here.
- Names of other organizations, their customers, products, hosts and tickets do not enter this
  repository, its commits or its releases.

## Delivery

Every write to GitHub is `b10x-bot[bot]`: commits and pushes through `b10x-gates bot`, pull
requests and merges through `b10x-gates api`. A personal `gh` login is read-only here.

## Commands

| Command | Does |
|---|---|
| `task check` | the gate: `ess specify validate`, a drift check of `generated/`, then every `providers/*.yaml` against the projected schema, file name equal to `provider`, and `upstream.sha256` as 64 lowercase hex |
| `task schema` | regenerates `generated/` from `spec/`; commit the result |

`spec/` is the source of the definition format. `generated/` is never edited by hand. Tool versions
are in `toolchain.json`; `yq` is mikefarah v4.

`task check` validates shape only. Whether an `operation_id` exists upstream, whether the pinned
document still hashes to `sha256`, and whether an operation mixes `operation_id` with
`method`/`path` are checked by the Connectors lowering frontend once it exists.
