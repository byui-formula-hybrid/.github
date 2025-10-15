# Workflow templates

This folder contains example caller workflows and documentation showing how to call the notification workflows in `.github/workflows/`.

Principles

- Sensitive values (webhooks, role IDs if you consider them sensitive) must be passed as repository secrets. When calling a reusable workflow across repositories, map secrets using the `secrets:` mapping in the caller.
- Non-sensitive options (channel names, template keys) may be passed via `with:` inputs and should be declared in the called workflow as `workflow_call.inputs`.
- Organization-level secrets must be enabled (scoped) for the caller repository by an org admin.

Files

- `call-discord-merge.yml` — example workflow that calls the merge notification workflow in the `.github` repo and maps secrets.
- `call-discord-pr.yml` — example workflow that calls the PR notification workflow in the `.github` repo and maps secrets.

Examples

Call the merge notification workflow (cross-repo example):

```yaml
name: Call Discord Merge Notification

on:
  push:
    branches: [ main ]

jobs:
  notify:
    uses: byui-formula-hybrid/.github/.github/workflows/discord-merge-notification.yml@main
    # Pass any non-sensitive inputs using `with:` (only if the called workflow declares inputs)
    # with:
    #   channel: ops
    secrets:
      # These secrets must exist in the caller repository (they can be org-scoped and enabled for this repo)
      DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}
      DISCORD_ROLE_ID: ${{ secrets.DISCORD_ROLE_ID }}
```

Call the PR notification workflow (cross-repo example):

```yaml
name: Call Discord PR Notification

on:
  pull_request:
    types: [opened, ready_for_review]

jobs:
  notify:
    uses: byui-formula-hybrid/.github/.github/workflows/discord-pr-notification.yml@main
    secrets:
      DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}
      DISCORD_ROLE_ID: ${{ secrets.DISCORD_ROLE_ID }}
```

Same-repo usage

If you keep these notification workflows in the same repository you run (no cross-repo `uses:`), you only need to add `DISCORD_WEBHOOK` and (optionally) `DISCORD_ROLE_ID` to the repository's secrets — no `secrets:` mapping required.

Security notes

- Never put webhooks directly in `with:` inputs or in the repo; always store them as secrets.
- GitHub requires explicit `secrets:` mapping for cross-repo reusable workflow calls to prevent accidental secret leakage.