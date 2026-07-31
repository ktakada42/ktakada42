# Plan: Integrate README stats cache refresh into existing streak stats workflow

## Goal
Use the existing streak stats GitHub Actions workflow to also refresh cache-busting query parameters for the README profile stats images once per day, so GitHub Profile Stats data updates automatically without needing a separate workflow.

## Repository
- `ktakada42/ktakada42`

## Background
The README currently embeds profile stats images from `github-stats-extended.vercel.app`.
These images can appear stale because they are cached.
A fixed `cache_bust=YYYYMMDD` query parameter works only once and does not update automatically.
There is already a daily workflow for streak stats generation, so the cleanest approach is to extend that workflow instead of adding another scheduled workflow.

## Requirements
1. Update the existing `.github/workflows/streak-stats.yml` workflow rather than creating a new workflow.
2. Add a step that updates `cache_bust=YYYYMMDD` in README image URLs using the current UTC date.
3. Ensure the step updates all relevant `github-stats-extended.vercel.app` URLs in `README.md`, including:
   - GitHub Profile Stats
   - Top Languages
   - WakaTime
4. Preserve existing streak stats behavior.
5. Ensure the commit step includes both `assets/streak-stats.svg` and `README.md`.
6. Avoid creating empty commits when nothing changed.

## Suggested implementation
### README changes
- Ensure each relevant stats URL in `README.md` includes a `cache_bust` query parameter.
- If a `cache_bust` parameter already exists, replace it with the current date.
- If it does not exist, append it.

### Workflow changes
In `.github/workflows/streak-stats.yml`:
- After the streak SVG generation step, add a step that updates cache-busting values in `README.md`.
- This can be implemented inline with Python for simplicity.
- Use the current UTC date formatted as `%Y%m%d`.

Pseudo-logic:
1. Read `README.md`
2. Find all `github-stats-extended.vercel.app` URLs for `/api`, `/api/top-langs/`, and `/api/wakatime`
3. Remove any existing `cache_bust=...`
4. Append `cache_bust=<current UTC YYYYMMDD>`
5. Write the updated file back

### Commit behavior
- Update the existing commit step to stage both:
  - `assets/streak-stats.svg`
  - `README.md`
- Keep the `git diff --cached --quiet || git commit ...` pattern to avoid empty commits.

## Acceptance criteria
- The existing scheduled streak stats workflow also refreshes README stats URLs daily.
- `README.md` contains `cache_bust=<today in UTC>` for all relevant stats image URLs after the workflow runs.
- No new workflow file is introduced.
- Existing streak stats SVG generation still works.
- The workflow commits only when files changed.

## Nice to have
- Keep the implementation compact and easy to maintain.
- Prefer inline scripting in the workflow unless a separate script is clearly cleaner based on the current repository structure.
