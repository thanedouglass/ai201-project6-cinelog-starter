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
**What I did:**
**How I verified:**

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->