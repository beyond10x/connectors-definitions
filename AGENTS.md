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

One connector per service: its read and write operations live in the same definition. Never split a
service into a per-direction pair (`<id>-read` and `<id>-write`) of connectors, files, bundles or
instances; writes added later go into the existing file (Connectors
`architecture-decision-record:connector-vocabulary`).

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
| `task release-check TAG=<version>` | refuses unless the tag is a bare version and `CHANGELOG.md` has `## [<version>] - <date>` and its `[<version>]:` link |

`spec/` is the source of the definition format. `generated/` is never edited by hand. Tool versions
are in `toolchain.json`; `yq` is mikefarah v4.

Each connector file starts with `schema: urn:beyond10x:connectors-definition:1`, the projected
schema's `$id`, which `ess generate schema validate` selects by.

`task check` validates shape only. That each `op` exists in the vendored document, and that each
`{var}` in `base_url` is declared in `config`, is checked by the lowerer.

## Releases

Released from source at bare-version tags (`0.1.0`, no `v`); nothing is on a registry. Every merged
change to a definition, an auth profile, a vendored document or the format is released. The pull request
that makes the change also dates its CHANGELOG entry, and its merge commit is tagged.

- `CHANGELOG.md` follows Keep a Changelog 1.1.0. Each change adds one line under `## [Unreleased]`
  naming the connector and operation ids it touches. A removed or renamed connector, operation or
  format field goes under a `**Breaking**` line.
- Version: a breaking change or a new connector, operation or format field is a minor bump while the
  version is below 1.0.0; a fix to an existing definition is a patch bump.
- The release commit turns `## [Unreleased]` into `## [<version>] - <date>` under a new empty
  `## [Unreleased]` and moves the link references at the bottom. `task release-check TAG=<version>`
  passes on it, and `Check` is green on the merge commit on `main` before it is tagged.
- The tag is the bot's, on that merge commit:
  `b10x-gates bot --repository beyond10x/connectors-definitions -- tag -a <version> -m "connectors-definitions <version>" <commit>`,
  then `b10x-gates bot --repository beyond10x/connectors-definitions -- push origin <version>`.
- `.github/workflows/release.yml` runs on the tag: `release-check`, then `check.yml`, then it creates
  the GitHub Release with the version's CHANGELOG section as notes,
  `connectors-definitions-<version>.tar.gz` and its `.sha256`.
- A release is finished when the tag, the green `release` run and the Release with both assets are
  verified. Never move or delete a pushed tag; fix forward with the next version.
