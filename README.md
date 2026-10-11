# repo-maintenance

Maintenance bots for Thanaen's repositories, ported from IS Decisions.

## Workflows

- **Skills update** (`.github/workflows/skills-update.yml`) runs on the 1st of each month. It lists every repo the bot App is installed on, keeps the ones with a `skills-lock.json`, runs `skills update` in copy mode, commits through the API so GitHub signs the commit, opens a `chore/skills-update` PR and merges it once Pullfrog approves. Run it by hand from the Actions tab; `dry-run` creates the commit without moving a branch.
- **Dependabot merge** (`.github/workflows/dependabot-merge.yml`) runs every 30 minutes. In every repo the bot App is installed on, it merges the open Dependabot PRs whose head commit Pullfrog approved and whose checks all passed (`mergeable_state` is `clean`). Nothing to add in the repos themselves. Run it by hand from the Actions tab; `dry-run` lists what it would merge.
- **Dependabot bun dedupe** (`.github/workflows/dependabot-bun-dedupe.yml`) is reusable. On a Dependabot PR that touches `bun.lock`, it runs `bun dedupe --lockfile-only` and commits the result to the PR branch as the bot App, so CI runs on it.

The `skills` CLI version is pinned in `skills-cli/package-lock.json`; Dependabot bumps it.

## Setup

1. A GitHub App owned by Thanaen, with repository permissions **Contents: read and write** and **Pull requests: read and write**, installed on the repos to maintain.
2. Actions secrets on this repo: `BOT_APP_CLIENT_ID` and `BOT_APP_PRIVATE_KEY`.
3. In each repo that calls the dedupe workflow, the same two names as **Dependabot** secrets.

## Dedupe caller

Add this as `.github/workflows/dependabot-bun-dedupe.yml` in a repo whose Dependabot config updates `bun`:

```yaml
name: Dependabot bun dedupe

on:
  pull_request:
    paths: [bun.lock]

permissions:
  contents: read

jobs:
  dedupe:
    if: github.actor == 'dependabot[bot]' && github.event.pull_request.user.login == 'dependabot[bot]'
    uses: Thanaen/repo-maintenance/.github/workflows/dependabot-bun-dedupe.yml@main
    secrets:
      app_client_id: ${{ secrets.BOT_APP_CLIENT_ID }}
      app_private_key: ${{ secrets.BOT_APP_PRIVATE_KEY }}
```
