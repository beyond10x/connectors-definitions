# connectors-definitions

Declarative provider definitions for [Connectors](https://github.com/beyond10x/connectors). Each
provider is one reviewed YAML file under [`providers/`](providers/README.md). It says where the
provider's API is, how to authenticate, and which operations are exposed.

**Status: scaffold.** No provider has been moved here yet. The format, the lowering step and the
`connectors definitions install` command are designed and not yet built. The design is in the
Connectors planning store, not yet merged there:
`architecture-decision-record:provider-definitions-and-runtimes` and
`architecture-design:provider-definitions-and-runtimes`.

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
