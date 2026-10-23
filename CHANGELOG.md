# Changelog

All notable changes to connectors-definitions are recorded here, in the format of
[Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/). The repository releases from
source at bare-version tags such as `0.1.0`; nothing is on a registry. Each release on GitHub
carries `connectors-definitions-<version>.tar.gz` (the connector definitions, shared auth profiles,
vendored upstream documents, the ESS source and its generated schema), its SHA-256 checksum and,
from 0.3.0, `index.json`.
Changes collect under `[Unreleased]` until the next release, and every merged change to a
definition is released.

## [Unreleased]

## [0.3.0] - 2026-10-23

Each release publishes `index.json`.

### Added

- `index.json` as a release asset, format `urn:beyond10x:connectors-definitions-index:1` (`spec/domains/release.yaml`, projected to `generated/`): the tag, the tagged commit, the tarball's SHA-256, every file in the tarball sorted by path with its SHA-256 and size, and the optional format features the connectors use (`embedded_text`, `query_fixed`, `query_folds`, `rate_limit_headers` in this release). 0.1.0 and 0.2.0 have none.
- `task release-assets TAG=<version>` writes the tarball, its `.sha256` and `index.json`, and refuses unless the index names exactly the tarball's files with their SHA-256; `task check` runs it on `HEAD` and validates the index against the projected schema.

### Changed

- The release workflow refuses a tag whose GitHub Release exists instead of replacing its assets.

## [0.2.0] - 2026-10-23

GitHub reads, and the format members they need.

### Added

- `connectors/github.yaml`, connector `github` with auth profile `github.token` (bearer, identity `GET /user`, subject `/id`) and 10 reads: `review_requests.list`, `issues.assigned`, `notifications.list`, `user.me`, `repos.list`, `repo.get`, `releases.list`, `release.latest`, `readme.raw`, `file.raw`; writes will join the same file.
- `upstream/github/`: GitHub's REST description `v2.1.0` (`api.github.com.yaml`, redacted under `upstream-redaction/4`) and its MIT licence.
- Format: an operation's `query_fixed`, `query_folds`, `embedded_text` and `rate_limit_headers`, and `rate_limit_headers` in `defaults`.

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

[Unreleased]: https://github.com/beyond10x/connectors-definitions/compare/0.3.0...HEAD
[0.3.0]: https://github.com/beyond10x/connectors-definitions/compare/0.2.0...0.3.0
[0.2.0]: https://github.com/beyond10x/connectors-definitions/compare/0.1.0...0.2.0
[0.1.0]: https://github.com/beyond10x/connectors-definitions/releases/tag/0.1.0
