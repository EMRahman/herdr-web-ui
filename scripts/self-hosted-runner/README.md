# Self-hosted CI runner

An ephemeral Docker GitHub Actions runner for this repository's Linux CI/CD jobs. Each container registers, runs exactly one job and exits; the launcher then starts a fresh one. The image has Chromium system libraries, ffmpeg, git, curl, jq, unzip and xz-utils preinstalled and no sudo. Jobs detect it with `RUNNER_ENVIRONMENT=self-hosted`.

## Prerequisites

- Docker on a Linux x64 host.
- `gh` logged in as a repository admin (needed to create registration tokens and delete runners).

## Run

```sh
scripts/self-hosted-runner/run.sh                         # 1 slot, repo from the current checkout
scripts/self-hosted-runner/run.sh --count 3 --cpus 8 --memory 12g
scripts/self-hosted-runner/run.sh --repo OWNER/NAME --labels herdr-ci --build
scripts/self-hosted-runner/run.sh --once                  # one container per slot, then exit
```

The image `herdr-ci-runner:local` is built when missing or with `--build`. Ctrl+C stops the containers and deletes any runner of this host (`herdr-ci-<host>-*`) still registered.

## Routing jobs to it

```sh
gh variable set CI_SELF_HOSTED_LABELS --body '["self-hosted","linux","x64","herdr-ci"]'
gh variable delete CI_SELF_HOSTED_LABELS   # back to GitHub-hosted ubuntu-24.04
```

A manual run can override the variable with the `self_hosted_labels` workflow_dispatch input. Fork PRs always use `ubuntu-24.04`; Windows/macOS jobs, `remote-bundles.yml` and `windows-install.yml` stay GitHub-hosted.

## Security model

This is a public repository. Fork PR workflow runs need maintainer approval, and a fork PR can edit `runs-on`, so reading the workflow changes before approving is the real boundary. Containers are ephemeral (`--ephemeral`, `--rm`), unprivileged (`--cap-drop ALL`, `no-new-privileges`, no `--privileged`), have no host mounts and no Docker socket, and the image has no sudo. The registration token reaches the container on stdin, never through its environment, so a job cannot read it from `/proc/1/environ`.
