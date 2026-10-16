# connectors-definitions

Provider definitions for [Connectors](https://github.com/beyond10x/connectors). Each provider is one
reviewed YAML file under [`providers/`](providers/README.md), format `connectors-provider/1`. The
file is complete as it stands: base URL, auth profiles, and every exposed operation with its
method, path and parameters. A runtime loads it directly; a person or an agent can read it and make
the call.

**Status: early.** One provider, [`jira`](providers/jira.yaml), is here, and `task check` validates
it. The `connectors` import and install commands are designed and not yet built. The design is in
the Connectors planning store:
`architecture-decision-record:provider-definitions-and-runtimes` and
`architecture-design:provider-definitions-and-runtimes`.

## Specifications

| Specification | What it fixes | Where |
|---|---|---|
| `connectors-provider/1` | the shape of `providers/<id>.yaml` | [ESS source](spec/domains/definition.yaml), projected to [JSON Schema](generated/schema/types/connectors_definitions.definition.ProviderDefinition.schema.json) (`$id` `urn:b10x:connectors-provider:1`) |
| `auth.profile/v1alpha1` | what an auth profile declares | [Connectors contract](https://github.com/beyond10x/connectors/blob/main/contracts/auth/profile/v1alpha1/semantics.md) |
| `catalog/v1alpha1` | bundles, the index and generic HTTP execution | [Connectors contract](https://github.com/beyond10x/connectors/blob/main/contracts/catalog/v1alpha1/semantics.md) |

## Upstream sources

Every upstream document a definition was imported from is vendored, gzipped, under
`upstream/<id>/`, with its license file where the document declares one. The definition records the
URL, retrieval date, SHA-256 and size; `task check` fails if the vendored file does not hash to
that SHA-256. A vendor can take the URL down; the definition stays reproducible.

## Check

```console
task check
```

It needs [`ess`](https://github.com/beyond10x/ess) at the version in [`toolchain.json`](toolchain.json),
[`yq`](https://github.com/mikefarah/yq) v4 and [`task`](https://taskfile.dev). It validates the ESS
source, fails if `generated/` drifted from it, checks every vendored source against its SHA-256,
and validates every `providers/*.yaml` against the projected schema. CI runs the same task. After a
change to `spec/`, run `task schema` and commit `generated/`.

## How it will work

1. `connectors` imports an upstream document (OpenAPI, Swagger 2.0, Google Discovery) and writes
   the selected operations, resolved, into `providers/<id>.yaml`. Nobody retypes endpoints; you
   review the file and edit descriptions, auth and selection.
2. A tag publishes the files and an `index.json` with each file's SHA-256.
3. `connectors definitions install --ref <tag>` fetches a tag, verifies every digest and records it
   in a lock file. Installing grants nothing: connections, admission and approvals stay in
   Connectors.

Any runtime that implements `connectors-provider/1` can execute the definitions. The Rust runtime in
Connectors is the reference.

## Without a runtime

The file itself says how to authenticate and how to call each operation. A generated page per
provider will add one `curl` command per operation, with credentials as environment-variable
references. A call made that way goes around Connectors' credential custody, approvals and audit
log. Use `connectors` when those are needed.

## What does not belong here

Code of any kind, credentials, and providers that start a host process or read host credentials,
such as Kubernetes, SQL or MCP. Those stay native in Connectors.

## License

Apache-2.0. Vendored upstream documents keep their own licenses, recorded per source.
