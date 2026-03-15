# gstack (OpenCode fork)

> Forked from [garrytan/gstack](https://github.com/garrytan/gstack). Adapted for [OpenCode](https://opencode.ai).

**gstack turns OpenCode from one generic assistant into a team of specialists you can summon on demand.**

Six opinionated workflow agents and commands for [OpenCode](https://opencode.ai). Plan review, code review, one-command shipping, QA testing, and engineering retrospectives — as agent tabs and slash commands.

### Without gstack

- The agent takes your request literally — it never asks if you're building the right thing
- It will implement exactly what you said, even when the real product is something bigger
- "Review my PR" gives inconsistent depth every time
- "Ship this" turns into a long back-and-forth about what to do
- You still do QA by hand: run tests, click around, hope nothing broke

### With gstack

| Name | Type | Mode | What it does |
|------|------|------|--------------|
| `plan-ceo` | Agent (tab) | Founder / CEO | Rethink the problem. Find the 10-star product hiding inside the request. |
| `plan-eng` | Agent (tab) | Eng manager / tech lead | Lock in architecture, data flow, diagrams, edge cases, and tests. |
| `review` | Agent (tab) | Paranoid staff engineer | Find the bugs that pass CI but blow up in production. |
| `qa` | Agent (tab) | QA engineer | Diff-aware test analysis. Runs tests, finds gaps, produces structured QA reports. |
| `/ship` | Command | Release engineer | Sync main, run tests, review diff, push, open PR. For a ready branch. |
| `/retro` | Command | Engineering manager | Team-aware retro: your deep-dive + per-person praise and growth for every contributor. |

## Demo: one feature, five modes

I start almost every feature in plan mode. I describe what I want to build, then switch to `plan-ceo` to pressure-test whether I am even building the right thing. Only after the product direction is locked do I shift into engineering, review, ship, and test.

```
You:   [switch to plan-ceo agent, describe the feature]
       I want to add seller photo upload to the listing app. Sellers
       should be able to upload a photo of their item and we create
       the listing from it.

Agent: "Photo upload" is not the feature. The real job is helping sellers
       create listings that actually sell. Here's what the 10-star version
       looks like: auto-identify the product from the photo, pull specs and
       pricing comps from the web, draft the title and description, suggest
       the best hero image...

You:   [switch to plan-eng agent]

Agent: [Architecture diagram, state machine for upload → classify → enrich
       → draft pipeline, async job boundaries, failure modes, test matrix]

You:   [switch to build agent, implement the plan]

You:   [switch to review agent]

Agent: Race condition: two tabs can overwrite cover-photo selection.
       Trust boundary: pulling web data into draft generation creates
       a prompt injection vector. Missing: orphan cleanup for failed uploads.

You:   (fix the issues)

You:   /ship

Agent: [Syncs main, runs tests, pushes branch, opens PR — done]

You:   [switch to qa agent]

Agent: Analyzing branch diff... 8 files changed, 3 modules affected.
       Running affected test suites...
       All tests pass. 2 test gaps identified in the enrichment service.
```

## Who this is for

You already use OpenCode heavily and want consistent, high-rigor workflows instead of one mushy generic mode. You want to tell the model what kind of brain to use right now — founder taste, engineering rigor, paranoid review, or fast execution.

This is not a prompt pack for beginners. It is an operating system for people who ship.

## Install

**Requirements:** [OpenCode](https://opencode.ai), [Git](https://git-scm.com/).

### Option 1: Global install (all projects)

Copy the agent and command files into your global OpenCode config:

```bash
# Clone the repo
git clone https://github.com/itsahedge/gstack.git ~/code/gstack

# Copy agents
cp ~/code/gstack/opencode/agents/*.md ~/.config/opencode/agents/

# Copy commands
cp ~/code/gstack/opencode/commands/*.md ~/.config/opencode/commands/
```

### Option 2: Per-project install

Copy into your project's `.opencode/` directory:

```bash
# From your project root
mkdir -p .opencode/agents .opencode/commands
cp ~/code/gstack/opencode/agents/*.md .opencode/agents/
cp ~/code/gstack/opencode/commands/*.md .opencode/commands/
```

### Option 3: Custom config directory

If you use a custom OpenCode config directory (e.g., `.opencode-work/config/opencode/`):

```bash
cp ~/code/gstack/opencode/agents/*.md .opencode-work/config/opencode/agents/
cp ~/code/gstack/opencode/commands/*.md .opencode-work/config/opencode/commands/
```

### What gets installed

- **4 agent files** in your `agents/` directory — show up as tabs in OpenCode
  - `plan-ceo.md` — CEO/founder plan review
  - `plan-eng.md` — Engineering manager plan review
  - `review.md` — Pre-landing PR review with inlined checklist
  - `qa.md` — QA engineer with diff-aware test analysis
- **2 command files** in your `commands/` directory — invoked as `/ship` and `/retro`
  - `ship.md` — Automated release workflow
  - `retro.md` — Engineering retrospective with trend tracking

All agents use `anthropic/claude-opus-4-6` by default. Edit the `model:` field in any agent's frontmatter to change.

---

## How I use these agents

Originally created by [Garry Tan](https://x.com/garrytan), President & CEO of [Y Combinator](https://www.ycombinator.com/). Adapted for OpenCode by [itsahedge](https://github.com/itsahedge).

The core insight: planning is not review. Review is not shipping. Founder taste is not engineering rigor. If you blur all of that together, you get a mediocre blend of all four.

These agents let you tell the model what kind of brain you want right now. Switch cognitive modes on demand — founder, eng manager, paranoid reviewer, release machine. That is the unlock.

---

## `plan-ceo` agent

This is **founder mode**.

This is where you want the model to think with taste, ambition, user empathy, and a long time horizon. You do not want it taking the request literally. You want it asking a more important question first:

**What is this product actually for?**

Think of this as **Brian Chesky mode**.

The point is not to implement the obvious ticket. The point is to rethink the problem from the user's point of view and find the version that feels inevitable, delightful, and maybe even a little magical.

### Example

Say you are building a listing app and you say:

> "Let sellers upload a photo for their item."

A weak assistant will add a file picker and save an image. That is not the real product.

In `plan-ceo`, the model asks whether "photo upload" is even the feature. Maybe the real feature is helping someone create a listing that actually sells.

If that is the real job, the whole plan changes. Now the model should ask:

* Can we identify the product from the photo?
* Can we infer the SKU or model number?
* Can we search the web and draft the title and description automatically?
* Can we pull specs, category, and pricing comps?
* Can we suggest which photo will convert best as the hero image?

It does not just ask, "how do I add this feature?"
It asks, **"what is the 10-star product hiding inside this request?"**

Three modes: **SCOPE EXPANSION** (dream big), **HOLD SCOPE** (maximum rigor), **SCOPE REDUCTION** (strip to essentials). The agent commits fully to whichever mode you choose.

---

## `plan-eng` agent

This is **eng manager mode**.

Once the product direction is right, you want a different kind of intelligence. Not more sprawling ideation. You want the model to become your best technical lead.

This mode nails:

* architecture and system boundaries
* data flow and state transitions
* failure modes and edge cases
* trust boundaries
* test coverage
* ASCII diagrams (forces hidden assumptions into the open)

Three review sizes: **SCOPE REDUCTION** (overbuilt — propose minimal), **BIG CHANGE** (interactive, section by section), **SMALL CHANGE** (compressed single-pass).

---

## `review` agent

This is **paranoid staff engineer mode**.

Passing tests do not mean the branch is safe.

`review` exists because there is a whole class of bugs that can survive CI and still punch you in the face in production. This is a structural audit, not a style nitpick pass.

It runs a two-pass review against `git diff origin/main`:

**Pass 1 — CRITICAL (blocks shipping):**
- SQL & Data Safety (injection, TOCTOU races, N+1)
- Race Conditions & Concurrency
- LLM Output Trust Boundary

**Pass 2 — INFORMATIONAL:**
- Conditional Side Effects
- Magic Numbers & String Coupling
- Dead Code & Consistency
- Test Gaps
- Crypto & Entropy
- Type Coercion at Boundaries
- View/Frontend

The checklist is inlined in the agent — no external file dependencies.

---

## `qa` agent

This is **QA engineer mode**.

The most common use case: you're on a feature branch, you just finished coding, and you want to verify everything works. Switch to the `qa` agent — it reads your git diff, identifies which modules your changes affect, runs the relevant tests, and finds coverage gaps.

Four modes:

- **Diff-aware** (automatic on feature branches) — reads `git diff main`, maps changed files to tests, runs them, finds gaps
- **Full** — systematic analysis of the entire test suite
- **Quick** — run the test suite once, report pass/fail
- **Regression** — compare against a previous QA report

---

## `/ship` command

This is **release machine mode**.

`/ship` is for the final mile. It is for a ready branch, not for deciding what to build.

The workflow is fully automated — you say `/ship`, it runs straight through:

1. Check branch (abort if on main)
2. Merge origin/main
3. Run tests (auto-detects framework)
4. Pre-landing review (two-pass checklist)
5. Version bump (auto-decide patch/micro)
6. CHANGELOG (auto-generate from diff)
7. TODOS.md (auto-mark completed items)
8. Commit (bisectable chunks)
9. Push
10. Create PR with full audit trail

It only stops for: merge conflicts, test failures, critical review findings, or major version bumps.

---

## `/retro` command

This is **engineering manager mode**.

At the end of the week, `/retro` analyzes commit history, work patterns, and shipping velocity and writes a candid retrospective.

It is team-aware. It identifies who is running the command, gives you the deepest treatment on your own work, then breaks down every contributor with specific praise and growth opportunities.

It computes: commits, LOC, test ratio, PR sizes, fix ratio, coding sessions, hotspot files, shipping streaks, and ship of the week.

```
You:   /retro

Agent: Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs | Streak: 47d

       ## Your Week
       32 commits, +2.4k LOC, 41% tests. Peak hours: 9-11pm.
       Biggest ship: payment service rewrite.

       ## Team Breakdown
       ### Alice — 12 commits, disciplined PR sizes, test ratio at 12%
       ### Bob — 3 commits, fixed the N+1 query. High impact.

       [Top 3 wins, 3 things to improve, 3 habits for next week]
```

Arguments: `/retro` (7d default), `/retro 24h`, `/retro 14d`, `/retro 30d`, `/retro compare`.

Saves JSON snapshots to `.context/retros/` for trend tracking across runs.

---

## Differences from upstream (garrytan/gstack)

This fork adapts gstack for OpenCode instead of Claude Code:

| Aspect | Upstream (Claude Code) | This fork (OpenCode) |
|--------|----------------------|---------------------|
| Format | SKILL.md files with Claude Code frontmatter | Agent `.md` files with OpenCode frontmatter |
| Invocation | Slash commands (`/plan-ceo-review`) | Agent tabs + slash commands |
| Install location | `~/.claude/skills/gstack/` | `~/.config/opencode/agents/` and `commands/` |
| User interaction | `AskUserQuestion` tool | Agent asks directly in conversation |
| Browse/browser | Compiled Playwright binary | Deferred (not yet ported) |
| Cookie import | macOS Keychain decryption | Deferred |
| Greptile integration | Built-in triage + reply | Removed (add back if using Greptile) |
| Template system | `.tmpl` → generated SKILL.md | Static markdown files |
| Update check | `gstack-update-check` binary | Removed |
| Conductor support | Multi-session orchestration | Removed |

### What's not ported (yet)

- **`/browse`** — The headless Playwright browser binary. This is a significant piece of infrastructure (~58MB compiled binary). Could be ported as an OpenCode tool or MCP server.
- **`/setup-browser-cookies`** — Cookie import from Chromium browsers. Depends on `/browse`.
- **Greptile integration** — PR review bot triage. The `/review` and `/ship` workflows had Greptile-aware comment classification. Can be re-added if you use Greptile.
- **Template system** — Upstream uses `.tmpl` files to auto-generate SKILL.md from source code metadata. This fork uses static markdown files directly.

---

## Customization

### Changing the model

Each agent's frontmatter has a `model:` field. Change it to any model your OpenCode config supports:

```yaml
model: anthropic/claude-sonnet-4-6    # faster, cheaper
model: anthropic/claude-opus-4-6      # deeper reasoning (default)
model: openai/o3                       # if you have OpenAI configured
```

### Adjusting tool permissions

Each agent's frontmatter has a `tools:` block. For example, to make the review agent read-only (no fixes):

```yaml
tools:
  edit: false
  write: false
  bash: false
  apply_patch: false
```

### Adding back Greptile

If you use [Greptile](https://greptile.com), you can re-add the triage workflow to `review.md` and `ship.md`. The upstream `review/greptile-triage.md` file in this repo has the full fetch/classify/reply protocol using `gh api`.

---

## Upstream

Original gstack by [Garry Tan](https://github.com/garrytan/gstack). MIT license.

## License

MIT
