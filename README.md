# connectors-definitions

Connector definitions for [Connectors](https://github.com/beyond10x/connectors). Each connector, one
service or API such as Jira, Gmail or GitLab, is one reviewed YAML file under
[`connectors/`](connectors/README.md), schema
`urn:beyond10x:connectors-definition:1`. The file is short: it names a vendored upstream document
and says which of its operations to expose, under which local names, with what configuration and
auth. Everything the upstream document states (methods, paths, parameters) is left to a lowerer,
which builds a runtime's internal representation from the two.

**Status: early.** Three connectors are here, [`jira`](connectors/jira.yaml),
[`gitlab`](connectors/gitlab.yaml) and [`confluence-v1`](connectors/confluence-v1.yaml), and
`task check` validates them. The Rust lowerer in Connectors is being built as a vertical slice; the install command
is designed and not yet built. The design is in the
Connectors planning store:
`architecture-decision-record:provider-definitions-and-runtimes` and
`architecture-design:provider-definitions-and-runtimes`.

## Vocabulary

As Connectors defines it (`architecture-decision-record:connector-vocabulary`): a provider is the
company (Atlassian, Google, GitLab); a connector is one service or API it offers (Jira, Gmail,
GitLab); an auth profile is one way to authenticate to a connector, and a connector may have
several. A connector definition is one file in `connectors/`.

## Specifications

| Specification | What it fixes | Where |
|---|---|---|
| `urn:beyond10x:connectors-definition:1` | the shape of `connectors/<id>.yaml` | [ESS source](spec/domains/definition.yaml), projected to [JSON Schema](generated/schema/types/connectors_definitions.definition.ProviderDefinition.schema.json) |
| `auth.profile/v1alpha1` | what an auth profile declares | [Connectors contract](https://github.com/beyond10x/connectors/blob/main/contracts/auth/profile/v1alpha1/semantics.md) |
| `catalog/v1alpha1` | bundles, the index and generic HTTP execution | [Connectors contract](https://github.com/beyond10x/connectors/blob/main/contracts/catalog/v1alpha1/semantics.md) |

## Upstream sources

Every upstream document a definition was imported from is vendored, gzipped, under
`upstream/<id>/`, with its license file where the document declares one. The definition records
the vendored path and SHA-256; `upstream/<id>/README.md` records the URL, retrieval date, size and
license. `task check` fails if the vendored file does not hash to that SHA-256. A vendor can take the URL down; the definition stays reproducible.

## Check

```console
task check
```

It needs [`ess`](https://github.com/beyond10x/ess) at the version in [`toolchain.json`](toolchain.json),
[`yq`](https://github.com/mikefarah/yq) v4 and [`task`](https://taskfile.dev). It validates the ESS
source, fails if `generated/` drifted from it, checks every vendored source against its SHA-256,
and validates every `connectors/*.yaml` against the projected schema. CI runs the same task. After a
change to `spec/`, run `task schema` and commit `generated/`.

## How it will work

1. You vendor the upstream document under `upstream/<id>/` and write `connectors/<id>.yaml`:
   operations by their upstream `operationId`, plus what the document does not say.
2. A tag publishes the files and an `index.json` with each file's SHA-256.
3. `connectors definitions install --ref <tag>` fetches a tag, verifies every digest and records it
   in a lock file. Installing grants nothing: connections, admission and approvals stay in
   Connectors.

A runtime in any language needs a lowerer for this schema and the source kinds it names. The Rust
lowerer in Connectors is the reference.

## Without a runtime

A generated page per connector will list the authentication steps and one `curl` command per
operation, with credentials as environment-variable references. A call made that way goes around Connectors' credential custody, approvals and audit
log. Use `connectors` when those are needed.

## What does not belong here

Code of any kind, credentials, and connectors that start a host process or read host credentials,
such as Kubernetes, SQL or MCP. Those stay native in Connectors.

## License

Apache-2.0. Vendored upstream documents keep their own licenses, recorded per source.
