# rmount-ctx

<p align="center">
  <b>Remote mount + live context injection for Claude Code</b><br>
  <sub>Edit locally. Execute remotely. Stay in context.</sub>
</p>

---

## Overview

`rmount-ctx` mounts remote project directories on macOS and keeps Claude Code synced with the live remote environment.

It gives you:

- a local mount at `~/remote_projects/<name>`
- automatic context injection into `CLAUDE.md` and `AGENTS.md`
- live remote metadata such as CPU, RAM, GPU, disk, and `tmux`
- a simple rule: files stay local, heavy commands run over `ssh`

No FUSE. No macFUSE. No kernel extensions. Just `ssh`, `rclone`, and native macOS `mount_webdav`.

---

## How It Works

```text
Local macOS                  Remote host
─────────────                ─────────────────────────
~/remote_projects/<name>  ←  mounted project directory
        │
        ├─ mount_webdav      ← native macOS mount
        │
        ├─ rclone serve webdav
        │
        └─ ssh tunnel        →  remote SSH + SFTP source
```

The mount is backed by three layers:

1. **SSH tunnel** — local ports forward to the remote host
2. **rclone WebDAV bridge** — SFTP is exposed as WebDAV
3. **`mount_webdav`** — macOS handles the mount natively

---

## Requirements

| Item | Why it matters |
|:---|:---|
| macOS | Uses the native WebDAV mount stack |
| `rclone` | Bridges SFTP to WebDAV |
| SSH config | Lets you mount hosts by alias |
| Remote tools (optional) | `lscpu`, `free`, `df`, `tmux`, and optionally `nvidia-smi` enrich injected context |

Quick checks:

```bash
sw_vers
rclone version
ssh <host> echo ok
```

Install `rclone` with Homebrew if needed:

```bash
brew install rclone
```

---

## Quick Start

### With Claude Code

Open this directory in Claude Code and follow the bundled `CLAUDE.md`. It describes the install flow and the hooks that keep context fresh.

### Manual install

#### 1) Install the CLI

```bash
mkdir -p ~/bin
cp rmount-ctx ~/bin/rmount-ctx
chmod +x ~/bin/rmount-ctx
```

Make sure `~/bin` is on your `PATH`.

#### 2) Register the skill

```bash
mkdir -p ~/.claude/skills/rmount-ctx
cp SKILL.md ~/.claude/skills/rmount-ctx/SKILL.md
```

#### 3) Add the SessionStart hook

Merge the contents of [`hooks/settings.json.hook`](hooks/settings.json.hook) into `~/.claude/settings.json` under `hooks.SessionStart`.

This refreshes remote context automatically when Claude Code opens a session inside `~/remote_projects/`.

#### 4) Add the Zsh hook

Append the contents of [`hooks/zshrc.hook`](hooks/zshrc.hook) to the end of `~/.zshrc`, then reload your shell:

```bash
source ~/.zshrc
```

This refreshes remote context whenever you `cd` into a mounted remote project.

#### 5) Verify

```bash
~/bin/rmount-ctx list
```

---

## SSH Host Alias

Define remote hosts in `~/.ssh/config` and use the alias in every command.

```sshconfig
Host myserver
    HostName 192.168.x.x
    User your-username
    IdentityFile ~/.ssh/id_rsa
```

Then mount by alias:

```bash
rmount-ctx mount myserver /path/to/project
```

---

## Usage

### Mount a remote project

```bash
rmount-ctx mount myserver /data/projects/myapp
```

### Use a custom mount name

```bash
rmount-ctx mount 16 /data/experiments training
```

### Refresh remote context

```bash
rmount-ctx refresh training
```

### See active mounts

```bash
rmount-ctx list
```

### Unmount

```bash
rmount-ctx unmount training
```

After mounting, open `~/remote_projects/<name>` in your editor and keep execution commands on the remote host.

If you omit the optional mount name, the script uses `host_basename`, such as `myserver_myapp`.

---

## What Gets Injected

Every mount writes a marked block into both `CLAUDE.md` and `AGENTS.md` on the mounted project.

| Section | Included data |
|:---|:---|
| Path translation | Local path, remote path, and an `ssh` execution template |
| Hardware | CPU model and RAM totals |
| GPU | Per-GPU utilization and VRAM usage, when available |
| Disk | Remote filesystem usage for the mounted path |
| tmux | Active sessions and their foreground commands |
| Rules | Clear guidance to keep execution on the remote host |

The block is wrapped in these markers:

```text
<!-- BEGIN REMOTE-MOUNT CONTEXT (auto-generated) -->
...
<!-- END REMOTE-MOUNT CONTEXT -->
```

That keeps the generated section separate from any hand-written notes.

---

## Repository Files

- `rmount-ctx` — the main CLI entry point
- `CLAUDE.md` — install and usage guidance for Claude Code
- `SKILL.md` — skill packaging for Claude Code
- `hooks/settings.json.hook` — SessionStart hook snippet
- `hooks/zshrc.hook` — shell hook snippet

---

## Troubleshooting

<details>
<summary><b>SSH tunnel failed</b></summary>

```bash
ssh <host> echo ok
which rclone
```

Check that the SSH alias exists and that `rclone` is installed on the local machine.
</details>

<details>
<summary><b>Permission denied on write</b></summary>

WebDAV writes go through the remote SSH user. If the mount is writable locally but still errors on the remote side, check the permissions on the target directory.
</details>

<details>
<summary><b>Port collision</b></summary>

Each mount reserves two local ports. If a stale mount remains, clean up the cache entry:

```bash
rm ~/.cache/rmount-ctx/<name>.json
```
</details>

<details>
<summary><b>Zombie processes after disconnect</b></summary>

```bash
pkill -f "rclone.*serve webdav"
```
</details>

---

## License

MIT
