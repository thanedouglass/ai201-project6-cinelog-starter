# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in
`services/watchlist_service.py`, following the existing `verb_to_noun`
convention used by `add_to_collection()`. I updated the import and call in
`routes/watchlist/watchlist.py`; a project-wide search found no other
references to the old name.
**How I verified:**
I reviewed all search results for both function names and ran the full test
suite after the rename.

## Comment 2 — Deduplication
**What I did:** I followed the existing `add_to_collection()` pattern in
`services/collection_service.py`. Its duplicate check calls
`CollectionEntry.query.filter_by(user_id=user_id, film_id=film_id).first()`;
when an entry is found, it raises `AlreadyInCollectionError` instead of
creating another row. I applied the same check to `WatchlistEntry` in
`add_to_watchlist()`, raising `AlreadyInWatchlistError` when a duplicate is
found.
**How I verified:**
I verified the check is performed after confirming the film exists and before
the new entry is added or committed, then ran the full test suite.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` with the equivalent of
`test_add_to_collection_nonexistent_film_raises` from
`tests/test_collection.py`. It uses the same isolated in-memory app and
`sample_user` fixtures, supplies a nonexistent UUID, and asserts that
`FilmNotFoundError` is raised.
**How I verified:**
I ran `pytest tests/test_watchlist.py -v`, followed by `pytest tests/ -v`.

## Comment 4 — Default visibility
**My position:** Keep `public=True` as the default for new watchlist entries.
**Reasoning:**
Watchlists are a discovery and recommendation feature in CineLog: making a
list visible by default lets friends see what someone plans to watch and
creates useful social context without requiring an extra setup step. This
optimizes for the common behavior of sharing film interests while the user is
actively building a list.
**Tradeoff acknowledged:**
The alternative, `public=False`, would be safer for users who treat a
watchlist as private planning. A public default can expose preferences before
someone changes visibility, so the UI should make the setting clear and allow
users to make an entry private; this decision favors discoverability while
acknowledging that privacy needs a deliberate opt-out.

## Comment 5 — Sort order
**My position:** Use date-added order, newest first, and implement the
maintainer's preference in `get_watchlist()`.
**Reasoning:**
The watchlist is primarily a queue of films the user recently decided to
watch. Showing the most recent additions first makes the endpoint immediately
useful for returning to a current intention and matches the collection
service's existing newest-first behavior.
**Engagement with reviewer's point:**
I agree with the reviewer's reasoning that most users want to see recent
additions first. Alphabetical order is predictable for lookup, but it hides
recency and can make a newly added film difficult to find. I retained the
existing `date_added` metadata and changed the query to
`WatchlistEntry.date_added.desc()`; the tradeoff is that alphabetical
title-based browsing would now require a separate sort option if that becomes
a product need.

## Comment 6 — Rebase
**What conflicted:** Rebasing the original watchlist commit onto updated
`origin/main` produced a conflict in `models.py`: `WatchlistEntry.film_id`
was still `db.Integer` on the feature side while main had migrated `Film.id`
and collection foreign keys to UUID strings.
**How I resolved it:**
I kept the UUID-compatible `db.String(36)` foreign key from main and retained
the watchlist index. I also updated the watchlist endpoint documentation from
an integer film ID to a UUID representation, then committed that explicit
UUID update as `fix: update WatchlistEntry film_id to UUID after main branch
refactor`.
**How I verified no conflict remains:**
The rebase completed successfully after the conflict was staged and
continued. `git status` shows no unmerged paths, `git log --merges
origin/main..HEAD` is empty, the model diff uses UUID strings, and the full
test suite passes.

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->