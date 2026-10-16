# providers

One file per provider: `<id>.yaml`, where `<id>` is the provider id Connectors uses (for example
`jira`, `slack`, `zendesk`) and equals the file's `provider` field.

Each file starts with `schema: urn:beyond10x:connectors-definition:1`. The format is defined in
[`spec/domains/definition.yaml`](../spec/domains/definition.yaml), and `task check` validates every
file against it. [`jira.yaml`](jira.yaml) is the reference example.
