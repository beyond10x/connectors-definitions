# Consumer A: issue-tracker ingest requirements (received 2026-10-16, peer session)

Input for the definition schema design. Consumer A is a downstream knowledge store.

Today's walk:
1. issues.search per project key, JQL `project = "<KEY>" AND updated >= "<day>" ORDER BY updated ASC`,
   fields summary,status,issuetype,created,updated,reporter,creator,description, maxResults 100.
   Paged by nextPageToken until absent/null or isLast; an empty page naming a next token is not the end;
   max 50 pages; a repeated token stops as truncated.
2. issue.comments per issue, paged by startAt until total; a total changing mid-walk stops that list
   (truncated). Per-issue refusal (forbidden/not_found/gone/rate_limited, 404/410/429) drops that
   issue's comments and continues; any other refusal fails the run.
3. Not fetched: changelog, attachments, worklogs.
4. Records: issue (native id = key, time = fields.updated, author = reporter else creator displayName);
   comment (native id `<KEY>#<id>`, time = updated, author = author.displayName). Rows carry system,
   channel `jira:<KEY>`, project, kind.
5. ADF bodies rendered to plain text by the consumer (mentions, emoji, dates, card URLs).
   Timestamps with `+0000` offsets must parse.
6. Delta: next `<day>` = day before last cut-off (JQL in account zone). Comments unfiltered by time;
   re-read per returned issue, deduped by id and time.
7. Only projects every employee can read; listed and verified explicitly.

Needs:
- Stable ids (issue key, comment id), comment linked to parent key.
- Updated timestamps with offsets on issues and comments.
- Author display names (reporter, creator, comment author); account ids a bonus.
- Issue type, status, summary, description, comment body; ADF or plain text, stated which; if
  rendered, mentions, links and dates kept as text.
- Resumable paging (token or startAt) with explicit end (isLast or total); truncation reason at a cap.
- Per-item refusals distinguishable from fatal ones.
- Wanted next: changelog (transitions with time and author), assignee, labels/components,
  parent/epic, created, project key and name, issue links.
- Deletions: visible, or at least absent from the next listing.
- Consumable by a `connectors` source that walks children per parent with record templates using
  `{parent}`.

## Follow-up (2026-10-16): generic-consumer requirements

1 identity/provenance; 2 time/delta/deletions; 3 paging, walks, container discovery, globs, caps;
4 content formats, renderers, mentions, trailers, attachments; 5 value kinds per field; 6 people
(account id + name, email personal); 7 visibility per container, live check, reading identity and
scopes; 8 personal-data hints; 9 per-item vs fatal refusals, retryable vs permanent; 10 entity
type and relation defaults; 11 call estimates, rate limits, synthetic fixtures, headless credential.
Full text lives in the consumer's private repo design note.
