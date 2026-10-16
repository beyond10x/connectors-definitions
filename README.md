# connectors-definitions

Declarative provider definitions for [Connectors](https://github.com/beyond10x/connectors). Each
provider is one reviewed YAML file under [`providers/`](providers/README.md). It says where the
provider's API is, how to authenticate, and which operations are exposed.

**Status: early.** One provider, [`jira`](providers/jira.yaml), is here, and `task check` validates
it. The lowering step and the `connectors definitions install` command are designed and not yet
built. The design is in the Connectors planning store, not yet merged there:
`architecture-decision-record:provider-definitions-and-runtimes` and
`architecture-design:provider-definitions-and-runtimes`.

## Specifications

| Specification | What it fixes | Where |
|---|---|---|
| `connectors-definition/1` | the shape of `providers/<id>.yaml` | [ESS source](spec/domains/definition.yaml), projected to [JSON Schema](generated/schema/types/connectors_definitions.definition.ProviderDefinition.schema.json) |
| `auth.profile/v1alpha1` | what an auth profile declares | [Connectors contract](https://github.com/beyond10x/connectors/blob/main/contracts/auth/profile/v1alpha1/semantics.md) |
| `catalog/v1alpha1` | bundles, the index and generic HTTP execution | [Connectors contract](https://github.com/beyond10x/connectors/blob/main/contracts/catalog/v1alpha1/semantics.md) |
| `connectors-provider/1` | the lowered document runtimes load | not written yet; design § 5 |

## Check

```console
task check
```

It needs [`ess`](https://github.com/beyond10x/ess) at the version in [`toolchain.json`](toolchain.json),
[`yq`](https://github.com/mikefarah/yq) v4 and [`task`](https://taskfile.dev). It validates the ESS
source, fails if `generated/` drifted from it, and validates every `providers/*.yaml` against the
projected schema. CI runs the same task. After a change to `spec/`, run `task schema` and commit
`generated/`.

## How it will work

1. You add or change `providers/<id>.yaml`: the pinned upstream document (URL and SHA-256), the
   base URL, the auth profiles and the selected operations.
2. CI lowers every file with a pinned `connectors` release into one `connectors-provider/1` JSON
   document per provider, and generates a Markdown page per provider.
3. A tag publishes the JSON documents, the pages and an `index.json` with each file's SHA-256.
4. `connectors definitions install --ref <tag>` fetches a tag, verifies every digest and records it
   in a lock file. Installing grants nothing: connections, admission and approvals stay in
   Connectors.

Any runtime that implements `connectors-provider/1` can execute the definitions. The Rust runtime in
Connectors is the reference.

## Without a runtime

Each provider's generated page lists the authentication steps and one `curl` command per operation,
with credentials as environment-variable references. A developer or an agent can make the call from
the page alone. That call goes around Connectors' credential custody, approvals and audit log. Use
`connectors` when those are needed.

## What does not belong here

Code of any kind, credentials, and providers that start a host process or read host credentials,
such as Kubernetes, SQL or MCP. Those stay native in Connectors.

## License

Apache-2.0. Vendor API documents keep their own licenses, recorded per source.
