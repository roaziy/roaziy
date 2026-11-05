---
name: commit-push
description: Commit changes in the roaziy profile repo so the release workflow cuts the right semver tag, then ask before pushing. Use whenever staging, committing, or pushing in this repo (git add, git commit, /commit, git push, "ship it", "release it"). Overrides the global git-commit-and-cicd-management type rules here.
---

# commit-push (roaziy profile repo)

Every push to `main` runs `.github/workflows/release.yml`, which reads the commit
messages since the last `v*` tag and publishes a GitHub release. **The commit type
decides the version**, so pick it on purpose.

## Commit type → version

| Type | Bump | Use for |
|---|---|---|
| `feat!:` (+ `BREAKING CHANGE:` footer) | major `2.0.0` | whole README / SVG redesign, user says "breaking" or "new README" |
| `feat:` | minor `x.2.0` | new card, section, block, stats widget, new terminal command in the SVG |
| `fix:` | patch `x.y.1` | **default**: typo, copy, colors, timing, links, CI, chores, docs |

- Default `fix:`. Use `feat:` / `feat!:` only when the change matches the row above or the user says so.
- Any other type (`docs:`, `chore:`...) is ignored by the release workflow; use `fix:` instead.
- Several commits in one push: the highest bump wins, so one push = one release.

## Message format

- `<type>: <description>`. No scope: `feat: add stats card`, never `feat(card): ...`.
- Subject ≤ 50 chars, imperative ("add", not "added"). Body wrapped at 72, says what and why.
- Breaking: subject `feat!: ...` and body footer `BREAKING CHANGE: <what changed>`.

## No Claude attribution

**Never** add `Co-Authored-By: Claude ...`, "Generated with Claude Code", or any agent
marker to commit messages, trailers, or sign-offs. This repo rule overrides the default
attribution guidance. Commits use the user's own git identity only.

## Steps

1. `git status --porcelain` and `git diff`: read what changed.
2. Scan the diff for secrets (`.env`, `*.pem`, tokens, keys). Found one → stop and tell the user.
3. Stage by purpose with `git add <path>`. Unrelated changes go in separate commits. No blind `git add .`.
4. Commit with the type from the table. Verify with `git log --oneline -5`.
5. **Stop after committing. Do not push on your own.** Ask the user with
   AskUserQuestion: "Push to main?" Include the version the push would release
   (highest type since the last `v*` tag, from `git describe --tags --abbrev=0`)
   in the question. Options: "Push" / "Don't push".
   - "Don't push" → done; commits stay local.
   - "Push" → `git push origin main`. Never `--force`, never `--no-verify`.
6. Only after a push, confirm the release:
   ```sh
   gh run watch "$(gh run list --workflow release.yml -L1 --json databaseId -q '.[0].databaseId')" --exit-status
   gh release view --json tagName,url -q '.tagName + "  " + .url'
   ```
   Report the new tag, or say why none was cut (only non-`feat`/`fix` commits).
