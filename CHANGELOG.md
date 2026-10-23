# Changelog

All notable changes to connectors-definitions are recorded here, in the format of
[Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/). The repository releases from
source at bare-version tags such as `0.1.0`; nothing is on a registry. Each release on GitHub
carries `connectors-definitions-<version>.tar.gz` (the connector definitions, shared auth profiles,
vendored upstream documents, the ESS source and its generated schema) and its SHA-256 checksum.
Changes collect under `[Unreleased]` until the next release, and every merged change to a
definition is released.

## [Unreleased]

## [0.1.0] - 2026-10-23

The first release: 15 connector definitions with 158 operations, validated against the
`urn:beyond10x:connectors-definition:1` schema.

### Added

- `urn:beyond10x:connectors-definition:1`, the connector definition format, typed in ESS (`spec/domains/definition.yaml`) and projected to JSON Schema under `generated/`.
- `urn:beyond10x:connectors-auth:1`, shared auth profiles taken with `use:`: `auth/atlassian.yaml`, `auth/google.yaml`, `auth/slack.yaml`.
- 15 connectors in `connectors/`: `confluence`, `confluence-v1`, `gitlab`, `google-calendar`, `google-drive`, `google-gmail`, `google-slides`, `hubspot`, `jira`, `openrouter`, `opsgenie`, `runpod`, `slack`, `slack-user`, `zendesk`.
- Guards and closed write bodies in 9 connectors; Jira issue create and comment check that the answer names the new object; Slack one-user reads and file delete.
- Feeds of GitLab merge requests, Jira issues and Confluence pages, with cursor or offset paging, a query template and time zone bounds.
- Every upstream document vendored gzipped under `upstream/<id>/`, pinned by SHA-256 with URL, retrieval date, size and licence.
- `task check`: validates the ESS source, the drift of `generated/`, each vendored document's SHA-256 and every definition and auth profile against the projected schema; ess 0.68.0.

[Unreleased]: https://github.com/beyond10x/connectors-definitions/compare/0.1.0...HEAD
[0.1.0]: https://github.com/beyond10x/connectors-definitions/releases/tag/0.1.0
