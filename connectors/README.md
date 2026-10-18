# connectors

One file per connector: `<id>.yaml`, where `<id>` is the catalog name Connectors uses today (for
example `jira`, `gitlab`, `confluence-v1`), or, for a second selection over one catalog bundle, that
name and a suffix, with `catalog:` naming the bundle (`slack-user`). Where the connector id is not
that name, `id:` gives it (`id: calendar` in `google-calendar.yaml`).

Each file starts with `schema: urn:beyond10x:connectors-definition:1`. The format is defined in
[`spec/domains/definition.yaml`](../spec/domains/definition.yaml), and `task check` validates every
file against it. Auth profiles several connectors share live in [`../auth/`](../auth/) and are
taken with `use:`.

A write's `body` names each key the caller may send and its type. A rich-text value the provider
takes as a document, such as Jira's Atlassian Document Format in `jira.yaml`, is declared `any`
and sent as the caller gives it; the format has no conversion from plain text or Markdown. A
document's fixed parts are fixed keys (`/body/type: =doc` in `issue.comment.add`), which a caller
cannot send.
