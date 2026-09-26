# Changelog

All notable changes to DreamRise are tracked here.

## [Unreleased] — Answer Key: fixes, design refresh & number toggle

### Added
- Answer Key PDF: question-number badge can now be hidden/revealed with a
  toggle button — only the number hides, the "প্রশ্ন" label badge itself
  stays, for exams whose question text already carries its own numbering.
- Answer Key PDF: design refresh — options laid out in a 2x2 grid (fewer
  pages to print), a numbered badge per question, accent-bordered cards,
  and an explicit A4 `@page` size/margin for print.

### Changed
- Removed the Answer Key's separate per-exam mark override
  (`akPosMark`/`akNegMark`, previously set via its own menu prompt) — the
  Answer Key now always reads its positive/negative marking directly from
  the main Setup Wizard, so it can never drift out of sync with the exam
  it's describing.
- Removed the separate online-link version of the Answer Key (the
  `?page=answerkey` web app route, its `AnswerPage.html` template, and the
  "🔗 Answer Key Web App নির্দেশিকা" menu item) — the "📄 Answer Key (দেখুন ও
  PDF নিন)" dialog is now the only way to view, print, or download it.

### Fixed
- **Logo not showing on the Answer Key PDF** — `DR_LOGO_LIGHT_URL` pointed
  at `Logo_For_Light.png`, but the actual file in this repo is
  `Logo_For_light.png` (lowercase "l"); GitHub's raw-file links are
  case-sensitive, so the mismatch silently 404'd every time. Corrected the
  constant to match the real file name.
- **Header / marking-scheme line not repeating on every printed page** —
  the previous `div` + `table-header-group` technique broke because its
  contents used `display:flex`, a known Chrome print-engine bug where
  flex children inside a repeating table header aren't reliably repeated.
  Rebuilt with a real `<table><thead>`, which Chrome repeats correctly on
  every page without any extra CSS trick.
- **Stuck negative-marking override showing "no negative marking" on exams
  that had one** — the per-exam override above, once set for one exam
  (e.g. one without negative marking), persisted in Script Properties
  indefinitely and silently overrode every later exam's real value. Fixed
  by removing the override feature entirely (see *Changed*, above).
- Print/PDF text on the Answer Key sheet was inherited from the small
  on-screen `px` font sizes; print now uses explicit, larger `pt` sizes so
  printed/PDF copies are legible.

## [Unreleased] — Portal badges & branding polish

### Added
- Result portal: per-subject and per-Additional-Mark **pass/fail badges**
  (`✅ পাস` / `❌ ফেইল`) directly on each subject card, so a specific weak
  subject is visible without reading raw numbers.
- Failed subject/Additional-Mark cards get a red left-border + tinted
  background and a red progress bar; passed ones get a subtle green accent.
  Colors are tuned separately for light and dark themes for contrast.
- Additional Mark items not yet manually entered show a neutral
  "⏳ এখনো বসানো হয়নি" badge instead of being wrongly marked as failed.
- Setup Wizard: **Overall** Additional Mark mode now has its own pass-mark
  field (Subject-wise mode already had this). A student can now fail overall
  purely on a weak written/homework mark, not just on percentage.

### Changed
- Print/PDF header in the result portal now shows only the logo — the
  redundant "DreamRise" text title next to it was removed.

### Fixed
- **Portal "প্রতি প্রশ্নের মান" / "নেগেটিভ মার্ক" showing wrong values**
  (e.g. 1.20 instead of 1.25, 0.63 instead of 0.25) when Additional Mark was
  enabled. These were being derived as `fullMarks ÷ totalQ`, but `fullMarks`
  includes the Additional Mark ceiling, so the division no longer equaled
  the configured positive/negative mark. Now read directly from
  `meta.posMark` / `meta.negMark`.
- **Written/Additional Mark not showing on the result portal at all.** The
  cached per-student array never carried the Additional Mark or Grand Total,
  so the portal had nothing to display even after a successful sync. Cache
  rows now include Grand Total and raw Additional Mark data, and
  `findStudentInMinifiedCache()` returns them to the portal.
- **Additional Mark columns going blank on re-sync.** Column matching used
  to rely on locating headers by Bengali text, which was fragile (invisible
  characters, header reordering); it now locates columns by computed
  position via a shared `buildRankingHeaders()` helper used by both the
  renderer and the reader, so structure is always self-consistent.
- **Race condition causing scores to change after auto-sync** — added a
  `LockService` lock around `calculateAndRank()` and around the fresh-payload
  read/merge inside `computeSingleStudentAndMerge()`, closing a lost-update
  window that could occur during burst form submissions or near-simultaneous
  portal searches.
- **Phone-number dedupe mismatch** — dedupe and every search path now share
  one `normalizePhone()` function (last 10 digits, digits-only), so a student
  entering their number in two formats no longer creates two ranking entries.
- Header-row detection is now bounded to the first few rows (previously
  scanned the whole sheet, risking a false match on student answer text).
- A toast now warns if the Answer Key row appears to have moved (e.g. after
  a manual sort), instead of silently mis-scoring everyone.

## Earlier history

- Auto-Sync on Portal Search, permanent `DR_Backup` sheet (survives cache
  expiry), exact full-text answer matching (replacing a comma/slash-split
  heuristic that broke on answers containing punctuation), Exam Summary bar,
  dynamic light/dark logo switching, multi-page print support, and the
  Statistics dialog were all introduced in earlier iterations and remain in
  place.
