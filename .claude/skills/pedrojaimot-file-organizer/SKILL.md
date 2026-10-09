---
name: pedrojaimot-file-organizer
description: Organizes Pedro Jaimot's personal files (Downloads, Documents, Desktop, Google Drive at pedrojaimot@gmail.com) into his standard personal structure, finds duplicates, and enforces his naming rules. Use when Pedro asks to clean up, sort, organize, dedupe, or archive personal files or folders. Not for Atlas Ocean Voyages / Mystic Cruises work files.
---

# Pedro Jaimot — Personal File Organizer

Personal files only. Employer files (Atlas Ocean Voyages, Mystic Cruises, charter/MICE client material) are **out of scope**: flag them, never sort them into personal folders.

## Operating rules (non-negotiable)

1. **Plan before action.** Analyze → propose plan → wait for explicit "yes" → execute.
2. **No deletes without approval.** Duplicates and junk go to `99_Archive/_To-Delete/` first; permanent delete only on a second explicit approval.
3. **Log every move** to `_organizer-log/YYYY-MM-DD.tsv` (`old_path<TAB>new_path`) at the target root so any run can be undone.
4. **Preserve timestamps** (`mv`, or `cp -p` across volumes). Never overwrite: on name clash, append `_v2`, `_v3`.
5. **Stop and ask** on anything sensitive (IDs, passwords, financial or medical records) or ambiguous. Never print their contents.
6. **Concise output.** Tables and lists only. No long explanations.

## Standard structure

```
Personal/
├── 00_Inbox/            # landing zone; empty weekly
├── 01_Admin/            # IDs, passports, visas, licenses, insurance, legal, home, vehicles
├── 02_Finance/          # banking, taxes/<YYYY>, investments, receipts/<YYYY>, contracts
├── 03_Career/           # CV, certifications, training, personal copies of performance records, network
├── 04_Health/           # medical, fitness, wellness, stress and focus
├── 05_Projects/         # personal ventures and business ideas, one folder each
├── 06_Learning/         # courses, books, research, AI/automation notes
├── 07_Travel/           # personal trips: <YYYY>-<destination>/
├── 08_Media/            # photos/<YYYY>, videos/<YYYY>, creative work
├── 09_Reference/        # manuals, templates, saved articles
├── 99_Archive/          # inactive >12 months, mirrored by area; _To-Delete/
└── _Work-Flagged/       # Atlas / Mystic files found in personal space, untouched, for Pedro to move
```

Rules:
- Max depth: 3 levels below an area folder. Flatten anything deeper.
- New top-level folders only with Pedro's approval.
- Personal travel → `07_Travel`. Charter/MICE operations → `_Work-Flagged`.

## Naming convention

`YYYY-MM-DD_area_short-description.ext`

- Lowercase, hyphens, no spaces, no `final`, `v2 (1)`, `copy`.
- Date = document date, not download date (use the file's mtime if no date appears in the content or name).
- Examples: `2026-04-15_finance_irs-1040-return.pdf`, `2026-09-02_career_cv-director-charter-mice.pdf`
- Photos and videos: keep camera names, sort into `<YYYY>/<YYYY-MM>/` instead.

## Workflow

### 1. Scope (ask only what's unknown)
Target folder? Conservative (sort and rename only) or full (also archive and dedupe)? Anything to exclude?

### 2. Analyze
```bash
T="<target>"
find "$T" -type f | wc -l; du -sh "$T"
find "$T" -type f | sed -n 's/.*\.\([^./]*\)$/\1/p' | tr A-Z a-z | sort | uniq -c | sort -rn | head -15
find "$T" -type f -size +100M -exec du -h {} + | sort -rh | head -10
find "$T" -type f -mtime +365 | wc -l   # archive candidates
# Work-file detection
grep -rliE 'atlas|mystic|charter|mice|manifest|rfp|contract.*(cruise|vessel)' --include='*.txt' --include='*.md' --include='*.csv' "$T" 2>/dev/null | head
find "$T" -type f -iregex '.*\(atlas\|mystic\|charter\|mice\|manifest\|rfp\|beo\).*' | head
```

### 3. Duplicates
```bash
find "$T" -type f -size +0 -print0 | xargs -0 sha256sum | sort | awk 'seen[$1]++{print}'   # exact dupes (Linux); macOS: shasum -a 256
```
Keep the copy with the best name or best location (newest if they tie). The rest go to `_To-Delete/`.

### 4. Plan (present exactly this)

| Area | Files | Action |
|---|---|---|
| 02_Finance | 34 | move + rename |
| _Work-Flagged | 6 | isolate, untouched |
| _To-Delete | 12 dupes (1.4 GB) | needs approval |

Plus: decisions needed (list), risks (list). End with: **Proceed? yes / modify / no**

### 5. Execute
After a "yes": create the folders, run the moves, write the log, and print the counts.

### 6. Close-out
- Summary: files moved, renamed, archived, flagged; space recoverable.
- Undo command: `while IFS=$'\t' read -r o n; do mv -n "$n" "$o"; done < _organizer-log/<date>.tsv`
- Upkeep: **weekly** empty `00_Inbox`, **monthly** archive and dedupe, **January** roll `Finance/taxes` and `Media` year folders.

## Google Drive (pedrojaimot@gmail.com)
If the Google Drive connector is available, use the same structure and rules. Search and read with the connector, propose moves in the same plan table, and only change or trash files after a "yes". Never change sharing permissions.
