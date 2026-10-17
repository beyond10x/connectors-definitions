# connectors

One file per connector: `<id>.yaml`, where `<id>` is the catalog name Connectors uses today (for
example `jira`, `gitlab`, `confluence-v1`), or, for a second selection over one catalog bundle, that
name and a suffix, with `catalog:` naming the bundle (`slack-user`).

Each file starts with `schema: urn:beyond10x:connectors-definition:1`. The format is defined in
[`spec/domains/definition.yaml`](../spec/domains/definition.yaml), and `task check` validates every
file against it. Auth profiles several connectors share live in [`../auth/`](../auth/) and are
taken with `use:`.
