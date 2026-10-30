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

## [0.6.0] - 2026-10-30

### Added

- `connectors/github.yaml`: write and check operations `branch.get` (`repos/get-branch`, read), `pull_request.create` (`pulls/create`, write, guarded by a read of `branch.get`), `pull_request.get` (`pulls/get`, read), `pull_request.merge` (`pulls/merge`, write, guarded by a read of `pull_request.get`), `check_runs.list` (`checks/list-for-ref`, read) and `commit.status` (`repos/get-combined-status-for-ref`, read). Needs Connectors 0.62.0 or newer, whose lowerer reads the `guard`, `body_enums` and `bounds` fields these use.
- `connectors/github.yaml`: auth profile `github.app` ("GitHub App installation", scheme `github_app`). The format's `AuthScheme` gains `github_app`. Needs Connectors 0.62.0 or newer, whose lowerer reads that scheme; an older Connectors installs the release and refuses to lower `github` alone.

### Changed

- `connectors/github.yaml`: the default profile is now `github.app`; `github.token` stays and must be named to be used. A consumer that configured `github` without naming a profile gets the App installation and needs its App id, installation id and key. `issues.assigned`, `notifications.list`, `user.me` and `review_requests.list` act for the authenticated user, which an installation token does not have (the pinned document marks the first three not enabled for GitHub Apps): use them under `github.token`.

## [0.5.2] - 2026-10-30

### Fixed

- `connectors/zendesk.yaml`, `ticket.audits` (`ListAuditsForTicket`) and `ticket.comments` (`ListTicketComments`): the walk rule is refined, equal to the Connectors built-in catalog. `meta.has_more`, when present, decides: false ends the walk, true continues even past an empty page and is an error without `meta.after_cursor`. Without it the walk ends on a page with no `meta.after_cursor` or no items. A cursor seen before is an error.

## [0.5.1] - 2026-10-30

### Fixed

- `connectors/zendesk.yaml`, `ticket.audits` (`ListAuditsForTicket`) and `ticket.comments` (`ListTicketComments`): the descriptions state an end condition the response carries. The pinned response schemas declare no `meta`, so `meta.has_more` may be absent; a walk now also ends on a page with no `meta.after_cursor` or with no audits (no comments, for ticket.comments). The incremental exports end on `end_of_stream`, which their pinned schemas declare, and are unchanged. The Connectors built-in catalog carries the same two descriptions.

## [0.5.0] - 2026-10-30

### Added

- `connectors/zendesk.yaml`: operation `ticket.audits` (`ListAuditsForTicket`, `GET /api/v2/tickets/{ticket_id}/audits`, read), one ticket's audits by cursor with `sort_order` withheld, as `ticket.comments` does.

## [0.4.0] - 2026-10-29

### Added

- `connectors/openai-responses.yaml`, connector `openai-responses` for any OpenAI-Responses-compatible API, with config `hostname` (default `api.openai.com`, base URL `https://{hostname}/v1`), auth profile `openai.api-key` (bearer, the key proven by `GET /models`) and 3 request/response operations: `models.list`, `response.create` (write, `stream` fixed to `false`) and `response.get` (`stream`, `starting_after` and `include_obfuscation` withheld); `upstream/openai-responses/`: the publisher's OpenAPI document at `openai/openai-openapi` `089b14f36` and its MIT licence.

### Changed

- `task check` runs on ess 0.75.0 (from 0.68.0); the projected schema in `generated/` is unchanged, and no connector, operation or format field changes.

## [0.3.1] - 2026-10-23

The format description follows Connectors `25ed378a2` and `7d2d19a75`; no definition changes.

### Changed

- `rate_limit_headers`: a `403` or `429` that carries `Retry-After` is `rate_limited` whatever `remaining` says, with the wait `Retry-After` names, read before `reset` (Connectors `25ed378a2`).
- Feeds: a `query` position's `first` holds no `{since}` (refusal `feed_query_first`), and the position's `time`, `query.parameter` and `page.parameter` each name a query parameter the items operation takes (`feed_position_parameter`).
- AGENTS.md: a connector id, file or bundle name never ends in `-read(s)` or `-write(s)`, as Connectors `7d2d19a75` states.

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

[Unreleased]: https://github.com/beyond10x/connectors-definitions/compare/0.6.0...HEAD
[0.6.0]: https://github.com/beyond10x/connectors-definitions/compare/0.5.2...0.6.0
[0.5.2]: https://github.com/beyond10x/connectors-definitions/compare/0.5.1...0.5.2
[0.5.1]: https://github.com/beyond10x/connectors-definitions/compare/0.5.0...0.5.1
[0.5.0]: https://github.com/beyond10x/connectors-definitions/compare/0.4.0...0.5.0
[0.4.0]: https://github.com/beyond10x/connectors-definitions/compare/0.3.1...0.4.0
[0.3.1]: https://github.com/beyond10x/connectors-definitions/compare/0.3.0...0.3.1
[0.3.0]: https://github.com/beyond10x/connectors-definitions/compare/0.2.0...0.3.0
[0.2.0]: https://github.com/beyond10x/connectors-definitions/compare/0.1.0...0.2.0
[0.1.0]: https://github.com/beyond10x/connectors-definitions/releases/tag/0.1.0
