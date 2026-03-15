---
description: "Fully automated ship workflow: merge main, run tests, review diff, bump version, update changelog, commit, push, create PR."
---

# Ship: Fully Automated Ship Workflow

You are running the `/ship` workflow. This is a **non-interactive, fully automated** workflow. Do NOT ask for confirmation at any step. The user said `/ship` which means DO IT. Run straight through and output the PR URL at the end.

**Only stop for:**
- On `main` branch (abort)
- Merge conflicts that can't be auto-resolved (stop, show conflicts)
- Test failures (stop, show failures)
- Pre-landing review finds CRITICAL issues (ask: A: Fix now, B: Acknowledge, C: False positive)
- MINOR or MAJOR version bump needed (ask)

**Never stop for:**
- Uncommitted changes (always include them)
- Version bump choice (auto-pick MICRO or PATCH)
- CHANGELOG content (auto-generate from diff)
- Commit message approval (auto-commit)

---

## Step 1: Pre-flight

1. Check the current branch. If on `main`, **abort**: "You're on main. Ship from a feature branch."
2. Run `git status`. Uncommitted changes are always included.
3. Run `git diff main...HEAD --stat` and `git log main..HEAD --oneline` to understand what's being shipped.

---

## Step 2: Merge origin/main (BEFORE tests)

Fetch and merge `origin/main` into the feature branch so tests run against the merged state:

```bash
git fetch origin main && git merge origin/main --no-edit
```

**If merge conflicts:** Try to auto-resolve if simple (VERSION, CHANGELOG ordering). If complex, **STOP** and show them.
**If already up to date:** Continue silently.

---

## Step 3: Run tests (on merged code)

**Auto-detect and run the project's test suite:**

```bash
# Detect test runner
if [ -f "package.json" ]; then
  # Check for test script in package.json
  npm test 2>&1 | tee /tmp/ship_tests.txt
elif [ -f "Makefile" ] && grep -q "^test:" Makefile; then
  make test 2>&1 | tee /tmp/ship_tests.txt
elif [ -f "go.mod" ]; then
  go test ./... 2>&1 | tee /tmp/ship_tests.txt
elif [ -f "pyproject.toml" ] || [ -f "pytest.ini" ]; then
  pytest 2>&1 | tee /tmp/ship_tests.txt
elif [ -f "Cargo.toml" ]; then
  cargo test 2>&1 | tee /tmp/ship_tests.txt
elif [ -f "Gemfile" ]; then
  bundle exec rake test 2>&1 | tee /tmp/ship_tests.txt
fi
```

**If any test fails:** Show the failures and **STOP**. Do not proceed.
**If all pass:** Continue silently -- just note the counts briefly.

---

## Step 3.5: Pre-Landing Review

Review the diff for structural issues that tests don't catch.

1. Run `git diff origin/main` to get the full diff.

2. Apply a two-pass review:
   - **Pass 1 (CRITICAL):** SQL & Data Safety, Race Conditions, LLM Trust Boundary, XSS
   - **Pass 2 (INFORMATIONAL):** Dead code, magic numbers, test gaps, type coercion issues

3. **Always output ALL findings.**

4. Output a summary header: `Pre-Landing Review: N issues (X critical, Y informational)`

5. **If CRITICAL issues found:** For EACH critical issue, ask:
   - The problem (`file:line` + description)
   - Your recommended fix
   - Options: A) Fix it now (recommend), B) Acknowledge and ship anyway, C) False positive
   After resolving: if user chose A on any issue, apply fixes, commit them, then tell user to run `/ship` again. If only B/C, continue.

6. **If only non-critical:** Output them and continue.

7. **If no issues:** Output `Pre-Landing Review: No issues found.` and continue.

---

## Step 4: Version bump (auto-decide)

1. Check if a `VERSION` file exists. If not, skip this step.

2. **Auto-decide the bump level based on the diff:**
   - Count lines changed (`git diff origin/main...HEAD --stat | tail -1`)
   - **MICRO/PATCH** (auto): < 200 lines changed, bug fixes, small features
   - **MINOR**: **ASK** -- only for major features or significant changes
   - **MAJOR**: **ASK** -- only for breaking changes

3. Write the new version to the VERSION file.

---

## Step 5: CHANGELOG (auto-generate)

1. Check if `CHANGELOG.md` exists. If not, skip.

2. Auto-generate the entry from ALL commits on the branch:
   - Use `git log main..HEAD --oneline` for commit history
   - Use `git diff main...HEAD` for the full diff
   - Categorize: `### Added`, `### Changed`, `### Fixed`, `### Removed`
   - Write concise bullet points
   - Insert after the file header, dated today
   - Format: `## [X.Y.Z] - YYYY-MM-DD`

**Do NOT ask the user to describe changes.** Infer from the diff and commit history.

---

## Step 5.5: TODOS.md (auto-update)

If `TODOS.md` exists, cross-reference against changes being shipped:

1. Check if any TODOs are completed by this PR (match commit messages and diff against TODO descriptions)
2. **Be conservative:** Only mark as completed with clear evidence
3. Move completed items to a `## Completed` section with version and date
4. Output summary: `TODOS.md: N items marked complete. M remaining.`

If TODOS.md doesn't exist, skip silently.

---

## Step 6: Commit (bisectable chunks)

**Goal:** Create small, logical commits that work well with `git bisect`.

1. Analyze the diff and group changes into logical commits. Each commit = one coherent change.

2. **Commit ordering** (earlier first):
   - Infrastructure: migrations, config, routes
   - Models & services (with their tests)
   - Controllers & views (with their tests)
   - VERSION + CHANGELOG + TODOS.md: always in the final commit

3. **Rules for splitting:**
   - A source file and its test file go in the same commit
   - Config changes group with the feature they enable
   - If total diff is small (< 50 lines across < 4 files), a single commit is fine

4. **Each commit must be independently valid** -- no broken imports.

5. Commit message format: `<type>: <summary>` (type = feat/fix/chore/refactor/docs)

---

## Step 7: Push

```bash
git push -u origin $(git branch --show-current)
```

---

## Step 8: Create PR

Create a pull request using `gh`:

```bash
gh pr create --title "<type>: <summary>" --body "$(cat <<'EOF'
## Summary
<bullet points from CHANGELOG or diff analysis>

## Pre-Landing Review
<findings from Step 3.5, or "No issues found.">

## TODOS
<If items marked complete: list them. Otherwise omit.>

## Test Results
- [x] All tests pass (N tests)
EOF
)"
```

**Output the PR URL** -- this should be the final output the user sees.

---

## Important Rules

- **Never skip tests.** If tests fail, stop.
- **Never force push.** Use regular `git push` only.
- **Never ask for confirmation** except for CRITICAL review findings and MINOR/MAJOR version bumps.
- **Split commits for bisectability** -- each commit = one logical change.
- **TODOS.md completion detection must be conservative.**
- **The goal is: user says `/ship`, next thing they see is the review + PR URL.**
