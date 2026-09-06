# Website maintenance notes

Quarto academic site for Leonel A. Regalado Cardoso. Source files live in the repo root;
Quarto renders to `docs/`, which is what GitHub Pages serves at https://leoregalado.com.
Both the source and the rendered `docs/` output must be committed for a change to go live.

## Content lives in more than one place — update all of it

Most requests here are small content edits ("this paper got accepted", "add this workshop").
The same fact is duplicated across several files, so a change is not done until every
relevant file below is updated. Check this list every time.

### When a paper changes status (accepted / under review / desk rejected / new paper)

1. **`cv.tex`** — move the entry between `Peer-Reviewed Papers`, `Under Review`, and
   `Working Papers`. Delete a subsection heading if it ends up empty. Then recompile
   (see "Building the CV" below).
2. **`research.qmd`** — same three sections. Keep the `---` separators between entries
   balanced, and drop a `##` section heading if nothing is left under it.
3. **`explore-research.qmd`** — find the paper's `<div class="paper-card">` and update all
   four spots: the `data-status` attribute (`published` / `under-review` / `working`), the
   `status-badge` class, the badge's visible text, and the `card-meta` line naming the
   journal. The `data-status` value drives the sidebar status filter, so it must match one
   of the three filter values.
4. **`index.qmd`** — check the Featured Research cards. Only some papers appear here, but
   the ones that do carry their own `featured-meta` status line, and it is easy to miss.
5. **`llms.txt`** — move the paper's entry between the `## Peer-Reviewed Publications`,
   `## Under Review`, and `## Working Papers` sections, update its `Status:` / `Journal:`
   line, and bump the `# Last updated:` date at the top of the file.

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

Two CVs are built from one source. `cv.tex` is the full CV (3 pages). `focused.tex` sets
`\def\focusedcv{1}` and then inputs `cv.tex`, omitting Leadership and using compact
formatting to stay within two pages. **The website navbar serves the focused version** —
`focused.pdf` is the CV a visitor gets. The full `cv.pdf` stays in the repo and is still
deployed, so the older `/cv.pdf` URL keeps working.

`pdflatex` is not on `PATH`. After any CV edit, build BOTH (twice each) and verify the
focused one is still two pages:

```bash
/Library/TeX/texbin/pdflatex -interaction=nonstopmode cv.tex
/Library/TeX/texbin/pdflatex -interaction=nonstopmode cv.tex
/Library/TeX/texbin/pdflatex -interaction=nonstopmode focused.tex
/Library/TeX/texbin/pdflatex -interaction=nonstopmode focused.tex
pdfinfo focused.pdf | awk '/^Pages/{print}'   # must be 2
cp cv.pdf docs/cv.pdf
cp focused.pdf docs/focused.pdf
rm -f cv.aux cv.log cv.out focused.aux focused.log focused.out
```

Commit all four PDFs (`cv.pdf`, `docs/cv.pdf`, `focused.pdf`, `docs/focused.pdf`). Both
are listed under `resources:` in `_quarto.yml`. The navbar and `cv.qmd`'s redirect both
point at `focused.pdf`, as does the `CV:` line in `llms.txt`.

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
- Untracked files in the repo root (logo drafts, loose figures, the `publications/` folder)
  are pre-existing and deliberately uncommitted. Stage files explicitly by name; do not
  `git add -A`.
