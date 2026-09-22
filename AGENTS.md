# Website maintenance notes

Quarto academic site for Leonel A. Regalado Cardoso. Source files live in the repo root;
Quarto renders to `docs/`, which is what GitHub Pages serves at https://leoregalado.com.
Both the source and the rendered `docs/` output must be committed for a change to go live.

## Content lives in more than one place — update all of it

Most requests here are small content edits ("this paper got accepted", "add this workshop").
The same fact is duplicated across several files, so a change is not done until every
relevant file below is updated. Check this list every time.

### When a paper changes status (accepted / under review / desk rejected / new paper)

1. **`cv.tex`** — move the entry between `Peer-Reviewed Papers`, `Revise \& Resubmit`,
   `Under Review`, and `Working Papers`. Delete a subsection heading if it ends up empty. Then recompile
   (see "Building the CV" below).
2. **`research.qmd`** — same sections (`## Revise & Resubmit` sits between publications
   and `## Under Review`). Keep the `---` separators between entries
   balanced, and drop a `##` section heading if nothing is left under it.
3. **`explore-research.qmd`** — find the paper's `<div class="paper-card">` and update all
   four spots: the `data-status` attribute (`published` / `under-review` / `working`), the
   `status-badge` class, the badge's visible text, and the `card-meta` line naming the
   journal. The `data-status` value drives the sidebar status filter, so it must match one
   of the three filter values. An R&R uses `data-status="under-review"` (the filter is
   labelled "Under Review / R&R") with the badge text "R&R".
   Card figures are static PNGs exported from the manuscript into `images/research/`
   (no hand-coded chart data). When a revision changes a paper's results, replace the
   figure files and rewrite the summary from the paper's own text; spot-checking numbers
   misses changed findings. Keep `llms.txt` and the homepage blurbs consistent with it.
4. **`index.qmd`** — check the Featured Research cards. Only some papers appear here, but
   the ones that do carry their own `featured-meta` status line, and it is easy to miss.
5. **`llms.txt`** — move the paper's entry between the `## Peer-Reviewed Publications`,
   `## Revise & Resubmit`, `## Under Review`, and `## Working Papers` sections, update its `Status:` / `Journal:`
   line, and bump the `# Last updated:` date at the top of the file.
6. **Job market folder** (iCloud `job market/`) — after pushing, sync the CV (see
   "Job market CV" below) and fix the paper's status/title in the three fact files the
   job market session drafts from: `JOB_MARKET_CONTEXT.md`,
   `00_inventory/factual-inventory.md`, and `00_inventory/Research Facts Reference.docx`.
   Stale facts there end up in cover letters. Check first whether the .docx is open in
   Word (a `~$` lock file in the folder) and edit only the affected text runs.

### When adding a conference, invited talk, or workshop

- **`cv.tex`** — `Academic Conferences and Invited Talks*` (grouped by year; `*` marks an
  invited talk) or the `Workshops` subsection.
- **`presentations.qmd`** — the matching section.
- **`llms.txt`** — the `## Presentations` section.

### Other content

Referee service appears in `cv.tex` (`Journal Referee`), `research.qmd` (`## Referee
Service` at the bottom), and `llms.txt`. Awards, teaching, and employment appear in
`cv.tex` and `llms.txt`, with teaching also in `teaching.qmd`.

## Building the CV

Two CVs are built from one source, and only one of them is published.

- **`cv.tex` → `cv.pdf` (2 pages)** — the focused CV. This is the version the website
  serves: the navbar "CV" link, `cv.qmd`'s redirect, and the `CV:` line in `llms.txt` all
  point at `cv.pdf`. Compact spacing and no Leadership section are the DEFAULT.
- **`cv-extended.tex` → `cv-extended.pdf` (3 pages)** — the full CV. It sets
  `\def\extendedcv{1}` and inputs `cv.tex`, which restores the looser spacing and adds the
  Leadership section. **This one is deliberately NOT published** — keep it out of `docs/`
  and out of `resources:` in `_quarto.yml`.

Edit content in `cv.tex` only; `cv-extended.tex` is a four-line wrapper. Anything wrapped
in `\ifdefined\extendedcv ... \fi` appears in the extended version alone.

`pdflatex` is not on `PATH`. After any CV edit, build BOTH (twice each), verify page
counts, and copy only `cv.pdf` into `docs/`:

```bash
/Library/TeX/texbin/pdflatex -interaction=nonstopmode cv.tex
/Library/TeX/texbin/pdflatex -interaction=nonstopmode cv.tex
/Library/TeX/texbin/pdflatex -interaction=nonstopmode cv-extended.tex
/Library/TeX/texbin/pdflatex -interaction=nonstopmode cv-extended.tex
pdfinfo cv.pdf | awk '/^Pages/{print}'            # must be 2
pdfinfo cv-extended.pdf | awk '/^Pages/{print}'   # 3
cp cv.pdf docs/cv.pdf                             # do NOT copy cv-extended.pdf
rm -f cv.aux cv.log cv.out cv-extended.aux cv-extended.log cv-extended.out
```

Commit `cv.tex`, `cv.pdf`, `docs/cv.pdf`, `cv-extended.tex`, and `cv-extended.pdf`. The
extended PDF lives in the repo root for convenience but never reaches leoregalado.com.

## Job market CV

`job market/0. CV/` in iCloud holds the CVs that go into applications. They are copies of
this repo's `cv.pdf` and `cv-extended.pdf` (this repo is the source of truth), so after
any CV change is built and pushed, run:

```bash
python3 ~/Library/Mobile\ Documents/com~apple~CloudDocs/job\ market/tools/sync_cv.py --extended
```

Run it after pushing, so the application copy never runs ahead of the live site. The script
warns if a PDF is older than its `.tex` or if `cv.tex` has uncommitted changes, and it
verifies checksums after copying. Add `--check` for a dry run.

## Rendering

Render one file per command. **Never pass multiple `.qmd` files to a single
`quarto render`** — it concatenates them into the first file's output and corrupts the
page. This broke the live research page once (fixed in commit `159ab59`).

```bash
quarto render research.qmd
quarto render explore-research.qmd
```

Rendering also rewrites `docs/search.json` and `docs/sitemap.xml`; commit those too.
`llms.txt` reaches the site because it is listed under `resources:` in `_quarto.yml` —
keep that entry if the config is reorganized.

## Verifying

The Quarto preview server is configured in `.Codex/launch.json` as `website` (port 8080).
Use it to check visual changes. Note that the preview server serves `.txt` without a
charset, so accented characters in `llms.txt` look garbled locally; GitHub Pages serves it
as `text/plain; charset=utf-8`, so this is a local artifact only, not a real bug.

## Conventions

- Accepted but not yet assigned a volume/issue: list as "Forthcoming", not with a year.
  Swap in the full citation and link once the DOI exists.
- Presentations follow the format used in JP Bastos's job market CV: conferences and
  invited talks merged into one year-by-year list, with `*` marking invited talks.
- Workshops are listed only when Leonel presented or workshopped a paper. Attendance-only
  colloquia are deliberately left off.
- **Contact email is `lregalad@ttu.edu` everywhere until Leonel graduates** (expected Spring
  2027), then switches to his personal `leoregalado31@gmail.com`, because the TTU address
  will be lost. When that happens, change it in: `cv.tex` (header `\href`), `_quarto.yml`
  (navbar envelope icon), `index.qmd` (hero social icon), and `llms.txt` (`Contact:` line and
  the last Notes bullet); rebuild both CVs and sync the job market folder. Leave the BECO 3310
  course pages as they are — that address was correct for that course.
- Untracked files in the repo root (logo drafts, loose figures, the `publications/` folder)
  are pre-existing and deliberately uncommitted. Stage files explicitly by name; do not
  `git add -A`.
