---
name: rmount-ctx
description: Manage WebDAV-over-SSH remote mounts with automatic remote-environment context injection. Use when user asks to mount/unmount remote projects, check remote GPU/RAM/tmux status, or mentions "remote mount", "挂载", "卸载", "远程状态". Also triggers implicitly before GPU-dependent training decisions.
---

# rmount-ctx: Remote Mount Context Injection

## Overview

Mount remote directories through `rclone serve webdav` over an SSH tunnel, then inject remote environment context (hardware, tmux sessions, execution rules) into CLAUDE.md and AGENTS.md so AI agents know: this is a remote mount, edit locally but execute remotely via ssh.

**Core principle:** Files are local through native macOS WebDAV, execution is remote (SSH). Context must be fresh before making GPU/tmux-dependent decisions.

**Announce at start:** "I'm using the rmount-ctx skill to manage remote mount context."

## Intent Categories

### 1. Explicit Lifecycle Commands

Recognize these user intents and map to the correct `rmount-ctx` subcommand:

| User says | Action |
|-----------|--------|
| "mount remote", "挂载", "mount X to Y" | `rmount-ctx mount <host> <remote-path> [name]` |
| "unmount X", "卸载 X", "remove mount X" | `rmount-ctx unmount <name>` |
| "list mounts", "查看挂载", "what's mounted" | `rmount-ctx list` |

### 2. Explicit Refresh

When user asks about current remote state:

| User says | Action |
|-----------|--------|
| "refresh remote", "刷新远程状态" | `rmount-ctx refresh <name>` |
| "GPU 情况怎么样", "how are GPUs looking" | `rmount-ctx refresh <name>` then read CLAUDE.md |
| "看看远程资源", "check remote resources" | `rmount-ctx refresh <name>` then read CLAUDE.md |

### 3. Implicit Refresh (Proactive — BLOCKING)

**Rule:** If you are about to make a decision that depends on current remote state, you MUST call `rmount-ctx refresh <name>` first, then re-read the updated CLAUDE.md before proceeding.

Triggers include:
- Preparing training scripts or launching training jobs
- Assigning GPU devices (`CUDA_VISIBLE_DEVICES`)
- Checking if a GPU is free before scheduling work
- Inspecting tmux sessions to decide whether to create new or attach existing
- Diagnosing "why is training slow" or "is my job still running"

This is a **blocking foreground refresh** — you need fresh data to make the right decision.

## Step-by-Step: Mount

1. Parse the user's intent: `<host>`, `<remote-path>`, optional `<name>`
2. Run: `rmount-ctx mount <host> <remote-path> [name]`
3. Report: mount point, hardware summary, tmux sessions
4. The script handles: SSH tunnel, rclone WebDAV service, native macOS WebDAV mount, remote info collection, CLAUDE.md/AGENTS.md injection, git exclude

## Step-by-Step: Unmount

1. Identify which mount to remove (by name)
2. Ask user to confirm if not explicitly stated
3. Run: `rmount-ctx unmount <name>`
4. Report: cleanup result

## Step-by-Step: Refresh

1. Determine which mount (if cwd is under `~/remote_projects/`, use that name)
2. Run: `rmount-ctx refresh <name>`
3. Read the updated CLAUDE.md to get fresh hardware/tmux state
4. Use that information for the current task

## Host Nickname Resolution

The user has SSH hosts configured in `~/.ssh/config` with nicknames like `19`, `12`, etc. Pass these nicknames directly to `rmount-ctx`. No IP resolution needed — SSH config handles it.

## Common Patterns

**Starting new work on a remote project:**
```
User: "mount 19 to ~/remote_projects/myproject"
→ rmount-ctx mount 19 /home/mengfei/projects/myproject
```

**Before launching training:**
```
→ rmount-ctx refresh myproject  (blocking — must complete first)
→ Read CLAUDE.md for GPU availability
→ Choose free GPU, set CUDA_VISIBLE_DEVICES
→ ssh 19 'cd /home/mengfei/projects/myproject && python train.py'
```

**Checking if a job finished:**
```
→ rmount-ctx refresh myproject
→ Read tmux section of updated CLAUDE.md
→ Report to user
```

## Red Flags

- Never run `rmount-ctx` in a pipe or background when refresh is implicit — it's blocking
- Don't assume GPU/tmux state from stale CLAUDE.md — always refresh before decisions
- If `rmount-ctx mount` reports SSH timeout, tell the user — mount may succeed but context is incomplete
- If the user asks to unmount and there are unsaved changes, warn them
