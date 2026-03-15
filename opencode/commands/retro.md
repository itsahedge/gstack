---
description: "Weekly engineering retrospective. Analyzes commit history, work patterns, and code quality metrics with persistent history and trend tracking. Team-aware."
---

# /retro -- Weekly Engineering Retrospective

Generates a comprehensive engineering retrospective analyzing commit history, work patterns, and code quality metrics. Team-aware: identifies the user running the command, then analyzes every contributor with per-person praise and growth opportunities.

## Arguments
- `/retro` -- default: last 7 days
- `/retro 24h` -- last 24 hours
- `/retro 14d` -- last 14 days
- `/retro 30d` -- last 30 days
- `/retro compare` -- compare current window vs prior same-length window
- `/retro compare 14d` -- compare with explicit window

## Instructions

Parse the argument to determine the time window. Default to 7 days if no argument given. Use `--since="N days ago"`, `--since="N hours ago"`, or `--since="N weeks ago"` for git log queries.

**Argument validation:** If the argument doesn't match a number followed by `d`, `h`, or `w`, the word `compare`, or `compare` followed by a number and `d`/`h`/`w`, show usage and stop.

### Step 1: Gather Raw Data

First, fetch origin and identify the current user:
```bash
git fetch origin main --quiet
git config user.name
git config user.email
```

The name returned by `git config user.name` is **"you"** -- the person reading this retro.

Run ALL of these git commands (they are independent):

```bash
# 1. All commits in window with timestamps, subject, hash, AUTHOR
git log origin/main --since="<window>" --format="%H|%aN|%ae|%ai|%s" --shortstat

# 2. Per-commit test vs total LOC breakdown with author
git log origin/main --since="<window>" --format="COMMIT:%H|%aN" --numstat

# 3. Commit timestamps for session detection
git log origin/main --since="<window>" --format="%at|%aN|%ai|%s" | sort -n

# 4. Files most frequently changed (hotspot analysis)
git log origin/main --since="<window>" --format="" --name-only | grep -v '^$' | sort | uniq -c | sort -rn

# 5. PR numbers from commit messages
git log origin/main --since="<window>" --format="%s" | grep -oE '#[0-9]+' | sed 's/^#//' | sort -n | uniq | sed 's/^/#/'

# 6. Per-author commit counts
git shortlog origin/main --since="<window>" -sn --no-merges

# 7. TODOS.md backlog (if available)
cat TODOS.md 2>/dev/null || true
```

### Step 2: Compute Metrics

Calculate and present in a summary table:

| Metric | Value |
|--------|-------|
| Commits to main | N |
| Contributors | N |
| PRs merged | N |
| Total insertions | N |
| Total deletions | N |
| Net LOC added | N |
| Test LOC (insertions) | N |
| Test LOC ratio | N% |
| Active days | N |
| Detected sessions | N |
| Avg LOC/session-hour | N |

Then show a **per-author leaderboard**:

```
Contributor         Commits   +/-          Top area
You (name)               32   +2400/-300   src/services/
alice                    12   +800/-150    src/components/
```

Sort by commits descending. Current user always first, labeled "You (name)".

**Backlog Health (if TODOS.md exists):** Count total open, P0/P1, completed this period.

### Step 3: Commit Time Distribution

Show hourly histogram using bar chart. Identify peak hours, dead zones, bimodal patterns, late-night clusters.

### Step 4: Work Session Detection

Detect sessions using **45-minute gap** threshold. Classify:
- **Deep sessions** (50+ min)
- **Medium sessions** (20-50 min)
- **Micro sessions** (<20 min)

Calculate total active time, average session length, LOC per hour.

### Step 5: Commit Type Breakdown

Categorize by conventional commit prefix (feat/fix/refactor/test/chore/docs). Show as percentage bar.

Flag if fix ratio exceeds 50% -- signals "ship fast, fix fast" pattern.

### Step 6: Hotspot Analysis

Top 10 most-changed files. Flag files changed 5+ times (churn hotspots).

### Step 7: PR Size Distribution

Bucket: Small (<100 LOC), Medium (100-500), Large (500-1500), XL (1500+). Flag XL PRs.

### Step 8: Focus Score + Ship of the Week

**Focus score:** % of commits touching the single most-changed top-level directory.

**Ship of the week:** Highest-LOC PR. Highlight PR number, title, LOC, why it matters.

### Step 9: Team Member Analysis

For each contributor:
1. Commits, LOC, areas of focus (top 3 dirs)
2. Commit type mix
3. Session patterns and peak hours
4. Test discipline (test LOC ratio)
5. Biggest ship

**For the current user ("You"):** Deepest treatment. Personal patterns, focus, biggest ship, what you did well, where to level up.

**For each teammate:**
- **Praise** (1-2 specific things anchored in actual commits)
- **Opportunity for growth** (1 specific, actionable suggestion)

**If solo repo:** Skip team breakdown.

**Co-Authored-By:** Track AI-assisted commits as a separate metric.

### Step 10: Week-over-Week Trends (if window >= 14d)

Split into weekly buckets. Show trends for commits, LOC, test ratio, fix ratio, sessions.

### Step 11: Streak Tracking

```bash
# All unique commit dates -- no hard cutoff
git log origin/main --format="%ad" --date=format:"%Y-%m-%d" | sort -u
# Personal streak
git log origin/main --author="<user_name>" --format="%ad" --date=format:"%Y-%m-%d" | sort -u
```

Count consecutive days backward from today.

### Step 12: Load History & Compare

```bash
ls -t .context/retros/*.json 2>/dev/null
```

If prior retros exist, load the most recent. Calculate deltas:
```
                    Last        Now         Delta
Test ratio:         22%    ->   41%         +19pp
Sessions:           10     ->   14          +4
```

If no prior retros: "First retro recorded -- run again next week to see trends."

### Step 13: Save Retro History

```bash
mkdir -p .context/retros
```

Save JSON snapshot with metrics, authors, streaks, version range.

### Step 14: Write the Narrative

Structure:

---

**Tweetable summary** (first line):
```
Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs | Streak: 47d
```

## Engineering Retro: [date range]

### Summary Table
### Trends vs Last Retro (if available)
### Time & Session Patterns
- When productive hours are and what drives them
- Whether sessions are getting longer or shorter
- Estimated hours per day of active coding

### Shipping Velocity
- Commit type mix and what it reveals
- PR size discipline
- Fix-chain detection
- Version bump discipline

### Code Quality Signals
- Test LOC ratio trend
- Hotspot analysis
- Any XL PRs that should have been split

### Focus & Highlights
- Focus score with interpretation
- Ship of the week callout

### Your Week (personal deep-dive)
- Personal stats, session patterns, peak hours
- Focus areas and biggest ship
- **What you did well** (2-3 specific things)
- **Where to level up** (1-2 specific suggestions)

### Team Breakdown (skip if solo)
For each teammate: what they shipped, praise, growth opportunity.

### Top 3 Team Wins
### 3 Things to Improve
### 3 Habits for Next Week

---

## Compare Mode

When `/retro compare`:
1. Compute metrics for current window
2. Compute metrics for prior same-length window using `--since` and `--until`
3. Show side-by-side comparison with deltas
4. Brief narrative on improvements and regressions

## Tone

- Encouraging but candid, no coddling
- Specific and concrete -- always anchor in actual commits
- Skip generic praise -- say exactly what was good and why
- Frame improvements as leveling up, not criticism
- Never compare teammates against each other negatively
- Keep total output around 3000-4500 words
- Use markdown tables and code blocks for data, prose for narrative
- Output directly to the conversation -- do NOT write files (except `.context/retros/` JSON snapshot)

## Important Rules

- ALL narrative output goes directly to the user. Only file written is `.context/retros/` JSON.
- Use `origin/main` for all git queries (not local main which may be stale)
- If the window has zero commits, say so and suggest a different window
- Round LOC/hour to nearest 50
- On first run, skip comparison sections gracefully
