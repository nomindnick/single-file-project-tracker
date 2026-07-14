# single-file-project-tracker

A project tracker contained in a single file. The file lives in NetDocuments;
the browser is the runtime and the file itself is the database. No server, no
hosting, no dependencies.

## Files

- **`ProjectTracker.html`** — the tracker (Phase 1). Project dashboard,
  per-project detail (deadlines, tasks, notes), cross-project "due soon" view,
  and in-place saving via the File System Access API with a download fallback.
- **`SPIKE.html`** — the Phase 0 spike that validated the save mechanism and
  the NetDocuments round trip. Kept for reference; not needed day to day.

## Workflow

1. In NetDocuments, **Check Out & Download** the file (don't just click it
   open — ndOffice's automatic check-in does not round-trip browser edits, so
   changes made that way are silently lost).
2. Open the downloaded file in Edge or Chrome (usually from Downloads).
3. Click **Connect to this file** once and pick the same file in the picker.
4. Edit; click **Save** (or Ctrl+S).
5. **Check the file back in** to NetDocuments, selecting it from Downloads.

The NetDocuments check-out lock is the concurrency mechanism between the two
users; every check-in is a versioned snapshot.

## Architecture notes

- All state lives in a JSON "data island":
  `<script type="application/json" id="tracker-data">`.
- Saving string-replaces the island inside the pristine source text read at
  connect time, then writes the whole file back through the retained file
  handle — the saved file is byte-identical outside the island.
- Every `/` in the serialized JSON is escaped as `\/` so user text containing
  `</script>` can never terminate the island and corrupt the file.
- No network egress of any kind (client-confidential data), and no
  localStorage/IndexedDB (data must travel with the file, not the machine).
