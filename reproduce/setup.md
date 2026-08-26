# How my-vm was built (2026-08-26)

**Prompt.** After evaluating the alignment-hive plugins, I asked which cloud option fits
"run things in the cloud, no GPU", pointing at Matt Pocock's tweets ("£20/month for a VM",
"a lil box of my own", benefits: collaboration, no worktrees, scale, factory setups). He
published no repo about it. Chosen: Hetzner VPS. Ask: "set it up and save it under a cute
short name in the tools folder; take what's useful from the remote-kernels skill".

**Taken from remote-kernels** (alignment-hive plugin): the brain/hands framing (Claude and
transcripts stay local, the box is disposable), gitignore-aware sync, credentials declared
but not stored, and "make cleanup explicit" → `my-vm down` requires typing the server name.
Not taken: MCP server, Jupyter kernels, per-session budgets (a VPS bills flat monthly).

**Recipe** ([Rasha Hantash's writeup](https://www.rasha.me/blog/claude-code-on-a-vps) was
the reference): Hetzner + Tailscale + tmux + Claude's browser login over SSH.

**Pieces.**
- `my-vm` — bash CLI. `hcloud` for the API, secrets via `secretspec run` (1Password), server IP
  cached in `.my-vm-ip` so day-to-day commands need no token.
- `cloud-init/user-data.yaml` — user with sudo, key-only SSH, ufw (22 + mosh UDP), fail2ban,
  Tailscale (joins when `TS_AUTHKEY` given), then a per-user script installs uv, fnm/node,
  Claude Code, Codex, clones dotfiles and runs its `bin/install.sh`. `~/.my-vm-ready` marks done.
- `secretspec.toml` — `HCLOUD_TOKEN` required, `TS_AUTHKEY` optional.
- Dotfiles skill `claude/skills/my-vm` — when/how Claude delegates to the box.

**Checks run.** `bash -n my-vm`; `my-vm help`; `my-vm target` with a fake `.my-vm-ip`; cloud-init
YAML parsed with PyYAML; the `my-vm run` remote string printed and inspected for quoting.
Not yet run against a real server — needs `HCLOUD_TOKEN` (see README).

**Codex red-team** (`codex-companion adversarial-review`, 2026-08-26) found four real issues,
all fixed: `readlink -f` is not portable to older macOS (now a symlink-resolving loop);
cloud-init touched `.my-vm-ready` even after failed installs (now `set -e`, verifies each tool,
writes `.my-vm-failed` on error and `my-vm wait` reports it); `my-vm run` joined argv with spaces
(now exactly one quoted command string); `with_secrets` used `HCLOUD_TOKEN` as a proxy for
"secrets loaded", skipping `TS_AUTHKEY` (now a `MYVM_SECRETS` sentinel).
