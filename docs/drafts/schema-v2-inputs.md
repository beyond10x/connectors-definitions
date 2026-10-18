# Inputs for definition schema v2 (collected 2026-10-16)

Not a design. Everything here is to be checked against Jira and GitLab before it enters `spec/`.

## Principles

- A definition holds only what the vendored source cannot say. Methods, paths, parameters, OAuth
  URLs and scopes come from the source; a definition overrides a fact only by setting it.
- Few tokens in the file. How a runtime makes names atomically unique (`jira.issue.get`) is the
  lowerer's concern.
- Every section binds to a Connectors contract by name, so each runtime exposes it as one trait or
  interface (`operations/v1alpha1`, `datasource.feed/v1alpha1`, `datasource.records/v1alpha1`, …).
- A definition may name a renderer or an auth mechanism, never contain one.

## Sections proposed so far

| Section | Holds | Example | Status |
|---|---|---|---|
| `config` | variables with type, default, allowed values | `hostname: {type: string, default: gitlab.com}` | in v1 |
| `auth` | profiles by id; shared profiles in `auth/<name>.yaml`, used with `use:` plus variables; inline when specific | `atlassian.basic: {use: atlassian}` | v1 has `scheme` only |
| `paging` | named styles, one `default`, an operation overrides by name; next value from body pointer or header; RFC 8288 `link` | `token`, `offset`, `offset-last`, `keyset` | not in v1 |
| `operations` | map by local key; `op`, `effect` override, `paging`, `guard`, `body`, `bounds`, `hide`, `response`, `options` | `issue.get: {op: getIssue}` | v1 has `op`, `effect`, `description` |
| `datasources` | kind (contract), containers, items, position, field map by JSON pointer, `filters` | `issues.filters.mine_open` | not in v1 |
| `references` | extractors (RFC 6570 URI templates reversed, linear-time regex), resolve via an operation, named outputs | `DEV-1234`, `/browse/{key}` | not in v1 |
| `views` | named field selections for an item (card, row); candidate format `ess-ui/1` | issue summary | not in v1 |

## Open questions

- ESS unions are tagged, so a string-or-map short form (`issue.get: getIssue`) cannot be typed.
- YAML anchors work within one file only; the `<<` merge key is YAML 1.1. Untested with
  `serde_yaml_ng` and `yq`.
- GitLab `{path}` spans several URL segments; RFC 6570 reverse matching may not cover it.
- Where paging edge cases live (runtime style contract or definition).

## Sources

- [gitlab-sketch.yaml](gitlab-sketch.yaml): the GitLab sketch with paging, guards, bodies and a
  datasource; not valid against v1.
- [consumer-a-requirements.md](consumer-a-requirements.md): a downstream consumer's ingest needs.
- GitLab v1 gaps: writes with guards and bodies, the `PRIVATE-TOKEN` header, identity and scope
  probes, `per_page` bounds, header paging.
