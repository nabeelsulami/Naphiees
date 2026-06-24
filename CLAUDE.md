# CLAUDE.md

Guidance for AI assistants (Claude Code and others) working in this repository.

## What this project is

**Naphiees** is a smart search engine for **Nphies** medical service codes (the
Saudi national health insurance / claims platform, "نفيس"). It lets staff and
users look up a unified Nphies code by typing the code, the service name in
Arabic or English, or any descriptive / synonym keyword. An admin panel (behind
a password) lets the maintainer add codes one at a time or bulk-import them from
an Excel file.

The UI is in Arabic (RTL) with bilingual (AR/EN) service data.

It is built as a single-file **Streamlit** application in Python, using a local
**Excel file** (`nphies_data.xlsx`) as its database.

## Current repository state

> **Important:** As of this writing the working tree is essentially empty. The
> original prototype (`app.py.txt` plus a sample `.xlsx`) was uploaded and then
> removed (see `git log`). The prototype source still lives in git history at
> commit `65eccfd` (`git show 65eccfd:app.py.txt`).
>
> Treat this as an early-stage project. When asked to "build the app" or
> "restore it," start from that prototype as the reference implementation rather
> than inventing a new design. The conventions below are derived from it.

## Architecture & conventions

The intended application is a single Streamlit script (e.g. `app.py`). Its
structure, in order:

1. **Page config & styling** — `st.set_page_config(...)` plus an injected CSS
   block (`st.markdown(..., unsafe_allow_html=True)`) for the title, subtitle,
   and footer.
2. **Database bootstrap** — `init_database()` creates `nphies_data.xlsx` with a
   few seed rows if the file does not exist.
3. **Data loading** — `load_data()` reads the Excel file, normalizes columns to
   strings (trim, replace `nan` with `''`), and is cached with
   `@st.cache_data(ttl=2)` so admin edits show up almost immediately.
4. **Public search UI** — a single text input that filters the dataframe with a
   case-insensitive `str.contains` across all four columns; results render in
   `st.dataframe` with Arabic display headers.
5. **Admin panel** — a sidebar gated by a checkbox + password
   (`st.sidebar.text_input(type="password")`). Two modes: manual single-record
   entry (a form) and bulk Excel upload (concatenate + `drop_duplicates` on the
   code). Both write back to `nphies_data.xlsx` and call `st.rerun()`.
6. **Footer** — a "Created by Nabeel" credit line.

### Data model

The Excel database has exactly these four columns (keep these names — both the
search and the bulk-upload validation depend on them):

| Column | Meaning |
| --- | --- |
| `Nphies_Code` | Unified Nphies service code (e.g. `83600-00-20`) |
| `Service_Name_EN` | Service name in English |
| `Service_Name_AR` | Service name in Arabic |
| `Keywords_Descriptions` | Comma-separated AR/EN synonyms & descriptive phrases used to widen search matches |

`Nphies_Code` is the de-facto primary key: bulk import deduplicates on it with
`keep='last'`.

### Conventions to follow

- **Bilingual, RTL-first UI.** User-facing strings are Arabic; data is AR + EN.
  Preserve this when editing UI text.
- **Excel is the database.** There is no SQL/ORM. Read and write
  `nphies_data.xlsx` with pandas (`read_excel` / `to_excel`, `openpyxl` engine).
- **Keep it a single, self-contained script** unless a refactor is explicitly
  requested.
- **Secrets:** the prototype hardcodes the admin password (`admin123`). This is
  a known weakness — prefer `st.secrets` or an environment variable for any real
  deployment, and never commit real credentials.
- **Cache invalidation:** any function that writes to the Excel file should be
  followed by `st.rerun()` so the cached `load_data()` reflects the change.

## Development workflow

There is no build config, requirements file, or CI yet. To run/develop:

```bash
# Recommended: a virtualenv
python -m venv .venv && source .venv/bin/activate

# Dependencies (no requirements.txt exists yet — create one if you add deps)
pip install streamlit pandas openpyxl

# Run the app (rename app.py.txt -> app.py first if restoring from history)
streamlit run app.py
```

The app auto-creates `nphies_data.xlsx` on first run. The admin password in the
prototype is `admin123`.

If you add dependencies, create/maintain a `requirements.txt`. If you set up
tests or linting, document the commands here.

## Git & contribution notes

- Active development branch for assistant work: `claude/claude-md-docs-xonzrp`.
  Develop, commit, and push there; do not push to `main` without explicit
  permission.
- Do **not** open a pull request unless explicitly asked.
- Commit messages should be clear and descriptive.
- `nphies_data.xlsx` is generated/user data — consider adding it (and `.venv/`,
  `__pycache__/`) to a `.gitignore` rather than committing local database state.

## When extending this project

- New service fields → add the column in `init_database()`, in the
  `required_cols` list in `load_data()`, and in the admin entry form; update the
  bulk-upload validation and the Arabic display-rename mapping accordingly.
- Keep the four canonical column names stable, or update **every** reference at
  once — search, upload validation, manual form, and display rename all hardcode
  them.
