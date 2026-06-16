# NerveOps GitHub Action

Send encrypted CI/CD signals from GitHub Actions to NerveOps.

This action is a thin wrapper around [`nerve send`](https://github.com/nerve-ink/nerve-cli).
It installs the public Nerve CLI, encrypts the message locally with your sender
DSN, and sends ciphertext to the Nerve relay.

## Start with failure alerts

1. Create a pipe in the NerveOps app.
2. Open pipe setup.
3. Copy the sender DSN.
4. Store it as a repository or organization secret named `NERVE_DSN`.
5. Add this step after your build/test/deploy steps:

```yaml
- name: Notify NerveOps on failure
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

## Deploy notification

```yaml
- name: Notify NerveOps
  uses: nerve-ink/nerveops-action@v1
  with:
    dsn: ${{ secrets.NERVE_DSN }}
    title: Backend Deploy
    severity: standard
    message: |
      deploy ${{ job.status }}
      repo: ${{ github.repository }}
      sha: ${{ github.sha }}
      run: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `dsn` | yes | | Sender DSN copied from the NerveOps app. Use a GitHub secret. |
| `message` | no | GitHub Actions summary | Signal body. Keep it short. |
| `title` | no | `GitHub Actions` | Short title shown in NerveOps clients. |
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
