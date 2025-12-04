# Trivox CI

Centralized reusable GitHub Actions workflows for Trivox projects.

This repo exposes workflows that other repos call via `workflow_call`, so CI logic lives in one place and game/tool repos stay thin.

---

## Usage (TL;DR)

In any project repo (e.g. `mini-arcade-core`), create:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches:
      - main
      - develop
      - "feature/**"
      - "release/**"
      - "hotfix/**"
  pull_request:
    branches:
      - main
      - develop

jobs:
  python-ci:
    uses: trivox-io/trivox-ci/.github/workflows/python-ci.yml@main
    with:
      project-name: "mini-arcade-core"
      lint-path: "src/mini_arcade_core"
    secrets:
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      SLACK_CHANNEL_ID: ${{ secrets.SLACK_CHANNEL_ID }}
```

Push a commit → CI runs using the shared workflow.

---

## Workflows

### `python-ci.yml`

Path: `.github/workflows/python-ci.yml`  

Standard Python CI pipeline:

- Matrix: Python `3.9`, `3.10`, `3.11`
- `pip install .[dev]`
- `isort --check-only .`
- `black --check .`
- `pylint` on a configurable path
- `pytest`
- Optional Slack notification with status + duration

#### Inputs

Exposed via `workflow_call`:

- `project-name` (string, required)  
  Logical name of the project using the workflow.

- `lint-path` (string, optional, default: `src/mini_arcade_core`)  
  Path passed to `pylint` (e.g. `src/pypong`, `src/space_invaders`).

#### Secrets (optional)

If you want Slack notifications, set these **in the calling repo**:

- `SLACK_BOT_TOKEN`
- `SLACK_CHANNEL_ID`

If they’re not provided, the Slack step is skipped.

---

## Adding a New Repo

1. Create `.github/workflows/ci.yml` in the repo.
2. Point a job to the shared workflow:

   ```yaml
   jobs:
     python-ci:
       uses: trivox-io/trivox-ci/.github/workflows/python-ci.yml@main
       with:
         project-name: "my-repo-name"
         lint-path: "src/my_repo_name"
       secrets:
         SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
         SLACK_CHANNEL_ID: ${{ secrets.SLACK_CHANNEL_ID }}
   ```

3. Push to `develop` or open a PR to confirm it runs.

---

_As more workflows are added (C++ builds, PyPI / itch.io publishing, etc.), they’ll follow the same pattern: defined here, consumed via `uses:` in each project._
