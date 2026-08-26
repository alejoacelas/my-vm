# my-vm

Can a €5/month box run my agents so my laptop doesn't have to?

`my-vm` is my Hetzner VM — Ubuntu with Claude Code, Codex, gh, uv, node, tmux and mosh preinstalled — plus a CLI to reach it and hand it work.

```sh
my-vm up && my-vm wait      # provision (~5 min), then `my-vm login` to sign the agents in
my-vm mosh                # roaming shell, attached to tmux
my-vm run build "cd best && uv run scripts/train.py"   # detached job → ~/jobs/build.log
my-vm log build -f        # follow it
my-vm push ./project      # rsync, honouring .gitignore
my-vm down                # delete the box (asks you to type its name)
```

Why Hetzner over RunPod/Codespaces: I don't need GPUs; a persistent cx33 (4 vCPU, 8 GB) costs about €5/month flat, versus hourly billing and ephemeral disks elsewhere.

## Setup

1. Hetzner: create a project, then an API token (Security → API tokens, Read & Write). Store it as `HCLOUD_TOKEN` in the 1Password `Developer-Credentials` vault; `secretspec.toml` declares it.
2. Optional Tailscale: a reusable auth key as `TS_AUTHKEY` in the same vault. The box then joins your tailnet with Tailscale SSH and `my-vm` addresses it by name; without it, `my-vm` uses the public IP with your `~/.ssh/id_ed25519` key.
3. `brew install hcloud mosh`, then `ln -s "$PWD/my-vm" ~/.local/bin/my-vm`.

Settings live in `my-vm.conf`; override in a gitignored `my-vm.local.conf`. `cloud-init/user-data.yaml` is what the box runs at first boot; it clones my public [dotfiles](https://github.com/alejoacelas/dotfiles) so Claude on the box has the same instructions and skills as Claude here.

## Delegating from Claude

The `my-vm` skill in dotfiles tells Claude when and how to push a long job to the box (`my-vm run`), poll it (`my-vm log`), and pull results back. Secrets never go to the box unless I `my-vm ssh` and log in myself.

See [reproduce/](reproduce/) for how this was built.
