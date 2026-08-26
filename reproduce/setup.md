# How hut was built (2026-08-26)

**Prompt.** After evaluating the alignment-hive plugins, I asked which cloud option fits
"run things in the cloud, no GPU", pointing at Matt Pocock's tweets ("£20/month for a VM",
"a lil box of my own", benefits: collaboration, no worktrees, scale, factory setups). He
published no repo about it. Chosen: Hetzner VPS. Ask: "set it up and save it under a cute
short name in the tools folder; take what's useful from the remote-kernels skill".

**Taken from remote-kernels** (alignment-hive plugin): the brain/hands framing (Claude and
transcripts stay local, the box is disposable), gitignore-aware sync, credentials declared
but not stored, and "make cleanup explicit" → `hut down` requires typing the server name.
Not taken: MCP server, Jupyter kernels, per-session budgets (a VPS bills flat monthly).

**Recipe** ([Rasha Hantash's writeup](https://www.rasha.me/blog/claude-code-on-a-vps) was
the reference): Hetzner + Tailscale + tmux + Claude's browser login over SSH.

**Pieces.**
- `hut` — bash CLI. `hcloud` for the API, secrets via `secretspec run` (1Password), server IP
  cached in `.hut-ip` so day-to-day commands need no token.
- `cloud-init/user-data.yaml` — user with sudo, key-only SSH, ufw (22 + mosh UDP), fail2ban,
  Tailscale (joins when `TS_AUTHKEY` given), then a per-user script installs uv, fnm/node,
  Claude Code, Codex, clones dotfiles and runs its `bin/install.sh`. `~/.hut-ready` marks done.
- `secretspec.toml` — `HCLOUD_TOKEN` required, `TS_AUTHKEY` optional.
- Dotfiles skill `claude/skills/hut` — when/how Claude delegates to the box.

**Checks run.** `bash -n hut`; `hut help`; `hut target` with a fake `.hut-ip`; cloud-init
YAML parsed with PyYAML; the `hut run` remote string printed and inspected for quoting.
Not yet run against a real server — needs `HCLOUD_TOKEN` (see README).
