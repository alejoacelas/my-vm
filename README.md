# devbox

Can a €5/month box run my agents so my laptop doesn't have to?

`devbox` is my Hetzner VM — Ubuntu with Claude Code, Codex, gh, uv, node, tmux and mosh preinstalled — plus a CLI to reach it and hand it work.

```sh
devbox up && devbox wait      # provision (~5 min), then `devbox login` to sign the agents in
devbox mosh                # roaming shell, attached to tmux
devbox run build "cd best && uv run scripts/train.py"   # detached job → ~/jobs/build.log
devbox log build -f        # follow it
devbox push ./project      # rsync, honouring .gitignore
devbox down                # delete the box (asks you to type its name)
```

Why Hetzner over RunPod/Codespaces: I don't need GPUs; a persistent cx33 (4 vCPU, 8 GB) costs about €5/month flat, versus hourly billing and ephemeral disks elsewhere.

## Setup

1. Hetzner: create a project, then an API token (Security → API tokens, Read & Write). Store it as `HCLOUD_TOKEN` in the 1Password `Developer-Credentials` vault; `secretspec.toml` declares it.
2. Optional Tailscale: a reusable auth key as `TS_AUTHKEY` in the same vault. The box then joins your tailnet with Tailscale SSH and `devbox` addresses it by name; without it, `devbox` uses the public IP with your `~/.ssh/id_ed25519` key.
3. `brew install hcloud mosh`, then `ln -s "$PWD/devbox" ~/.local/bin/devbox`.

Settings live in `devbox.conf`; override in a gitignored `devbox.local.conf`. `cloud-init/user-data.yaml` is what the box runs at first boot; it clones my public [dotfiles](https://github.com/alejoacelas/dotfiles) so Claude on the box has the same instructions and skills as Claude here.

## Delegating from Claude

The `devbox` skill in dotfiles tells Claude when and how to push a long job to the box (`devbox run`), poll it (`devbox log`), and pull results back. Secrets never go to the box unless I `devbox ssh` and log in myself.

See [reproduce/](reproduce/) for how this was built.
