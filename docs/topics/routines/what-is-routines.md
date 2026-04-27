# Claude Code Routines: What They Are and How They Work

## Overview

A **routine** is a saved Claude Code configuration — a prompt, one or more repositories, and a set of MCP connectors — packaged once and run automatically by Anthropic-managed cloud infrastructure. Routines keep working when your laptop is closed; they execute as full Claude Code cloud sessions, fully autonomous, with no human in the loop during a run.[1]

Routines are the answer to the question: *what do I do with a Claude Code workflow that needs to keep happening on its own?* Manual `/loop` runs require an open session. Local cron-style tasks need your machine to be on. GitHub Actions are bound to repository events. Routines bridge that gap with **three trigger types** that can be attached, in any combination, to a single saved configuration.[1]

The feature is currently in **research preview**. Behavior, limits, and the API surface may change. The HTTP API ships behind a beta header (`experimental-cc-routine-2026-04-01`), with the two prior versions kept working so callers have time to migrate.[1]

***

## The three trigger types

A single routine can have any combination of:[1]

- **Schedule** — runs on a recurring cadence (preset hourly, daily, weekdays, weekly; or a custom cron expression set via the CLI), or once at a specific future timestamp.
- **API** — gives the routine a dedicated HTTP endpoint. Anything that can make an authenticated POST can fire the routine: alerting tools, deploy pipelines, internal admin UIs.
- **GitHub** — runs in response to repository events (`pull_request.*`, `release.*`) on repositories where the Claude GitHub App is installed, with rich filter expressions on author, title, body, branches, labels, draft state, and merge state.

Combining triggers is where routines become genuinely useful: a PR review routine can run nightly to catch backlog, fire on every new PR for real-time review, and accept ad-hoc API calls when a developer asks for a re-review after pushing fixes — all from one config.[1]

***

## Where routines live

Routines run on **Anthropic-managed cloud infrastructure**, separate from your local machine. Each run is a fresh Claude Code cloud session that:[1]

- Clones each repository in the configuration from its default branch
- Loads the routine's **cloud environment** (network access tier, environment variables, setup script — the script is cached, so it doesn't re-run on every session)
- Executes the routine's prompt with the selected model
- Has access to every MCP connector you included
- Pushes any changes to `claude/`-prefixed branches by default (or to existing branches if "Allow unrestricted branch pushes" is enabled per-repo)
- Records the run as a new session in your session list, where you can review changes and open PRs

There is **no permission-mode picker and no approval prompts** during a run. Safety comes from scoping: which repos you select, how locked-down the environment is, which connectors you include, and the branch-push policy.[1]

Routines belong to your individual claude.ai account and are not shared with teammates. Anything a routine does through your connected GitHub identity or connectors appears as you: commits and pull requests carry your GitHub user, and Slack messages, Linear tickets, or other connector actions use your linked accounts.[1]

***

## Where to create them

Three surfaces, all writing to the same cloud account:[1]

- **Web** — `claude.ai/code/routines` → "New routine"
- **Desktop app** — Routines sidebar → "New routine" → "Remote" (choosing "Local" creates a Desktop scheduled task instead, which is a separate feature that runs on your machine)
- **CLI** — `/schedule` in any session, optionally with a description (e.g. `/schedule daily PR review at 9am`, or `/schedule in 2 weeks, open a cleanup PR that removes the feature flag`)

The CLI's `/schedule` creates **scheduled** routines only. To add an API or GitHub trigger to an existing routine, edit it on the web. The CLI also supports `/schedule list`, `/schedule update`, and `/schedule run` for managing existing routines.[1]

***

## Plan availability

Routines are available on **Pro, Max, Team, and Enterprise** plans, with [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web) enabled.[1]

***

## How runs are billed

Routines draw down subscription usage like interactive sessions, but with one extra control:[1]

- A **per-account daily cap** limits how many routine runs can start.
- **One-off runs are exempt** from the daily cap. They count as regular subscription usage, but do not eat into the routine-run allowance.
- When the daily cap or subscription usage limit is hit, organizations with **extra usage** enabled keep running on metered overage. Without it, additional runs are rejected until the window resets.

Current consumption and remaining daily allowance are visible at `claude.ai/code/routines` and `claude.ai/settings/usage`.[1]

***

## Use cases the docs call out

The official docs pair each trigger type with a representative use case:[1]

- **Backlog maintenance** (schedule) — nightly issue-tracker grooming, posting a summary to Slack.
- **Alert triage** (API) — monitoring tool calls the routine when an error threshold trips; routine correlates the stack trace with recent commits and opens a draft PR with a fix.
- **Bespoke code review** (GitHub `pull_request.opened`) — applies a team's own review checklist with inline comments.
- **Deploy verification** (API) — CD pipeline fires the routine after each prod deploy; it runs smoke checks and posts go/no-go.
- **Docs drift** (schedule) — weekly scan of merged PRs, opens update PRs against the docs repo for changed APIs.
- **Library port** (GitHub `pull_request.closed` filtered to merged) — ports a change from one SDK repo to a parallel SDK in another language.

These illustrate the common thread: **unattended, repeatable, tied to a clear outcome**.[1]

***

## Limits worth knowing up front

- **Minimum schedule interval is one hour.** Cron expressions that run more frequently are rejected.[1]
- **Custom cron is CLI-only** — pick the closest preset on the web form, then `/schedule update` to refine.[1]
- **Runs may start a few minutes late** due to per-routine stagger. The offset is consistent for each routine.[1]
- **GitHub webhooks are subject to per-routine and per-account hourly caps** during the research preview. Events beyond the cap are dropped until the window resets.[1]
- **The API token is shown once at generation.** If you don't store it, regenerate (which invalidates the old one).[1]
- **Each matching trigger creates a new session** — no session reuse for GitHub-triggered routines. Two PR updates produce two independent sessions.[1]
- **The `/fire` endpoint is claude.ai-only** — not part of the Claude Platform API surface.[1]

***

## Related features

The routines page explicitly links to several adjacent features:[1]

- **`/loop` and in-session scheduling** — for tasks that need to continue inside an open CLI session.
- **Desktop scheduled tasks** — local cron-style tasks that run on your machine and have access to local files.
- **Cloud environment** — the runtime environment used by routines and other cloud sessions.
- **MCP connectors** — what gives routines reach into Slack, Linear, Google Drive, etc.
- **GitHub Actions** — Claude in your CI pipeline rather than as a routine.

These are not interchangeable; the [deep-dive companion](routines-deep-dive.md) compares them on the dimensions that actually matter (where it runs, what triggers it, who owns the runtime).

***

## References

- [Automate work with routines](https://code.claude.com/docs/en/routines.md) — the official Claude Code docs page (April 2026), source for every claim above.[1]
