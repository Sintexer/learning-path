---
flashcards:
  q-hiru: { nid: 1795356717042, hash: 4pnpzh4j, sync: nz2tskzb }
  q-aw94: { nid: 1795356717066, hash: jcn9btu2, sync: yp4fgedu }
---

## Modern CLI Tools

- [`eza`](https://github.com/eza-community/eza) (maintained fork of `exa`) — modern `ls` replacement with git status and tree view
- `lf` — fast Vim-motions terminal file manager
- [`duf`](https://github.com/muesli/duf) — Disk Usage _(better `df`)_
- [`bat`](https://github.com/sharkdp/bat) — Reading files with syntax highlighting _(better `cat`)_
- [`fzf`](https://github.com/junegunn/fzf) — Fuzzy file/history finder _(better `find` / `Ctrl+R`)_
- [`rip`](https://github.com/nivekuil/rip) — Safe deletion tool _(better `rm`)_
- [`ripgrep`](https://github.com/BurntSushi/ripgrep) (`rg`) — Fast recursive text search _(better `grep`)_
- [`bpytop`](https://github.com/aristocratos/bpytop) / `btop` — Resource monitoring _(better `htop`)_
- [`speedtest-cli`](https://github.com/sivel/speedtest-cli) — Command line speed test utility
- [`dua-cli`](https://github.com/Byron/dua-cli) — Disk usage analyzer and monitor _(better `du`)_

## Production Debugging Staples (Standard POSIX / Linux)

When SSHing into production servers or `kubectl exec`ing into minimal [[Container|containers]], modern Rust tools are rarely installed. Know the built-in equivalents:

- **Networking**: `ss -tulpn` (listening sockets/ports), `curl -v`, `dig`, `tcpdump`.
- **Processes & Open Files**: `ps auxf`, `lsof -i :8080`, `lsof +L1` (deleted-but-open files), `strace -p <pid>` (syscalls).
- **Disk & System**: `df -h`, `df -i` (inodes), `du -sh *`, `iostat -xz 1`, `vmstat 1`, `journalctl -u <service> -f`.
- **Safe Bash scripting**: Always start automation scripts with `set -euo pipefail` so unset variables, command failures, and mid-pipeline errors fail fast instead of silently continuing.

> [!tip] Production reality
> Remember the [[TTY]] buffering trap when piping logs through `grep`/`awk`: redirecting stdout to a pipe switches libc from line buffering to block buffering, so live tail pipelines can appear frozen unless you pass `--line-buffered` (e.g., `tail -f app.log | grep --line-buffered ERROR`).

Always start production Bash scripts with `set -euo pipefail` so ==failing commands, unset variables, and broken pipes exit immediately== instead of silently corrupting state.
^q-hiru

Which standard Linux commands show (1) listening TCP/UDP ports with owning PIDs and (2) deleted files still consuming disk space?::(1) `ss -tulpn` (or `lsof -i`) and (2) `lsof +L1`.
^q-aw94

## Related

- [[TTY]]
- [[Linux devices]]
- [[Linux hard drive structure]]
- [[MacOS keyboard actions]]
- [[Container]]
- [[Kubernetes]]
