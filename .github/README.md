# CI

This repository uses GitHub Actions (`.github/workflows/validate.yml`) with two jobs:

- **lint** — runs `ansible-lint` on every push, PR and manual run
- **deploy** — runs the playbook against the servers, only on `main` (push or manual run), after lint passes. Overlapping runs are serialized to avoid conflicts on the target (e.g. apt locks).

## One-time setup

The deploy job connects to the servers over SSH with a dedicated deploy key pair. Its private half lives in GitHub secrets, never in the repo.

### 1. Generate the key pair

On your machine, generate a key dedicated to CI (no passphrase — CI cannot answer prompts):

```sh
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/infra_deploy -N ""
```

### 2. Authorize the public key on the servers

For each host in `inventory.yml`:

```sh
ssh-copy-id -i ~/.ssh/infra_deploy.pub <ansible_user>@<ansible_host>
```

The playbook uses `become: true`, so the user must have passwordless sudo (default on Debian cloud images):

```sh
ssh -i ~/.ssh/infra_deploy <ansible_user>@<ansible_host> sudo -n true
```

### 3. Configure GitHub secrets

In *Settings → Secrets and variables → Actions*, create two repository secrets:

| Secret | Value |
| --- | --- |
| `SSH_PRIVATE_KEY` | Full contents of `~/.ssh/infra_deploy` (the **private** key, including the `-----BEGIN/END-----` lines) |
| `SSH_KNOWN_HOSTS` | The known_hosts entry for each host in `inventory.yml` |

Prefer the entry already trusted on your machine (TOFU) over `ssh-keyscan`:

```sh
ssh-keygen -F sortir.in
```

If the host is not known yet, connect to it once with `ssh` first, or use `ssh-keyscan -H <ansible_host>` (may fail on some servers, e.g. OpenSSH 10 with per-source connection limiting).

Host key checking stays enabled: if a server's host key changes, the deploy fails instead of silently trusting the new key.

### 4. Firewall

GitHub-hosted runners must reach port 22 of every target host. If it is firewalled, use a self-hosted runner instead.

## Optional hardening

- In the servers' `authorized_keys`, prefix the CI key with `no-port-forwarding,no-agent-forwarding,no-X11-forwarding` to limit it to SSH commands.
- Create a `production` [environment](https://docs.github.com/en/actions/deployment/targeting-different-runs/environments) and set `environment: production` on the `deploy` job to require manual approval before each run.

## Manual run

Trigger a deploy without pushing from the *Actions* tab → **CI** → **Run workflow** (on `main`).
