# GitHub Actions Failure Alerts To Your Phone

Get a phone alert when CI, deploys, smoke tests, or release jobs fail. GitHub
Actions gets a send-only encrypted NerveOps secret, not chat history or command
power.

[Website](https://nerve.ink) · [App Store](https://apps.apple.com/us/app/nerveops/id6778026992) · [Google Play](https://play.google.com/store/apps/details?id=ink.nerve.app&pcampaignid=web_share) · [CLI](https://github.com/nerve-ink/nerve-cli)

Use this when:

- a deploy failed and nobody saw the workflow notification;
- a smoke test caught production drift after a green build;
- a cron, backup, or release job needs to page your phone;
- you do not want a Slack, Telegram, or generic bot token in CI.

This action wraps [`nerve send`](https://github.com/nerve-ink/nerve-cli). It
installs the public Nerve CLI, encrypts the alert locally with your sender DSN,
and sends ciphertext to the Nerve relay.

## Alert Me When CI Fails

1. Install NerveOps from the
   [App Store](https://apps.apple.com/us/app/nerveops/id6778026992) or
   [Google Play](https://play.google.com/store/apps/details?id=ink.nerve.app&pcampaignid=web_share).
2. Create a pipe in the NerveOps app.
3. Open pipe setup.
4. Copy the sender DSN.
5. Store it as a repository or organization secret named `NERVE_DSN`.
6. Add this step after your build, test, deploy, or smoke-test steps:

```yaml
- name: Alert phone on CI failure
  if: failure()
  uses: nerve-ink/nerveops-action@v1
  with:
    dsn: ${{ secrets.NERVE_DSN }}
    title: CI failed
    severity: critical
    message: |
      FAILED: ${{ github.repository }}
      ref: ${{ github.ref_name }}
      sha: ${{ github.sha }}
      run: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

## Alert Me When Deploy Fails

```yaml
- name: Alert phone on deploy failure
  if: failure()
  uses: nerve-ink/nerveops-action@v1
  with:
    dsn: ${{ secrets.NERVE_DSN }}
    title: Deploy failed
    severity: critical
    message: |
      DEPLOY FAILED: production
      repo: ${{ github.repository }}
      sha: ${{ github.sha }}
      run: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

## Alert Me When Smoke Tests Fail

```yaml
- name: Run production smoke tests
  run: ./scripts/smoke-prod.sh

- name: Alert phone on smoke-test failure
  if: failure()
  uses: nerve-ink/nerveops-action@v1
  with:
    dsn: ${{ secrets.NERVE_DSN }}
    title: Smoke tests failed
    severity: critical
    message: |
      SMOKE FAILED: production
      repo: ${{ github.repository }}
      ref: ${{ github.ref_name }}
      run: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `dsn` | yes | | Send-only sender DSN copied from the NerveOps app. Use a GitHub secret. |
| `message` | no | GitHub Actions summary | Alert body. Keep it short. |
| `title` | no | `GitHub Actions` | Short alert title shown in NerveOps clients. |
| `severity` | no | `standard` | `standard`, `alert`, or `critical`. |
| `kind` | no | `alert` | Signal kind metadata. |
| `install-url` | no | `https://nerve.ink/install.sh` | Nerve CLI installer URL. |

## Security boundary

Use a sender DSN for GitHub Actions. A sender DSN can send into one pipe, but it
cannot read history, decrypt content, connect as an agent, or execute commands.

Do not send full logs or secrets. Send short operational context: repository,
branch/environment, status, commit SHA, and run URL.

## When not to use this action

Do not use this action as a monitoring platform, chat system, or remote command
runner. It is for one-way encrypted operational signals.

For signed actions on a machine you control, use
[`nerve-agent`](https://github.com/nerve-ink/nerve-agent) explicitly.
