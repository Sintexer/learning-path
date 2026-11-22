> AKA **Teletypewriter**

Teletypewriters (**TTYs**) were originally used as an automated and efficient form of telegraphy. Instead of sending messages in Morse code and having the receiving party manually transcribe them, TTYs sent and received typed text messages automatically over telegraph or telephone lines, encoding each character as a series of electrical signals.

When computers came into widespread use, TTYs were repurposed so operators could input commands and receive output from computers—input was typed on a keyboard and output was printed on paper.

As technology advanced, TTYs were paired with screens to create **computer terminals**. Terminals were not independent computers, but input/output devices connected to a mainframe computer. Today, while physical TTYs and dumb terminals are rare, **terminal emulators** mimic a TTY inside a graphical interface ([[Linux Terminal]]).

## TTY vs PTY in Modern Linux

- **Virtual consoles (`/dev/tty1`–`/dev/tty6`)**: Kernel-managed text consoles accessible directly on the host.
- **Controlling terminal (`/dev/tty`)**: Abstract alias pointing to the current process's controlling terminal ([[Linux devices]]).
- **Pseudoterminal — PTY (`/dev/ptmx` master + `/dev/pts/N` slave)**: A software pair emulating a hardware TTY. Used by SSH, `tmux`, GUI terminal emulators, and interactive containers (`docker run -it` allocates a PTY and keeps stdin open).

What is the difference between `/dev/tty` and `/dev/pts/N` in Linux?::`/dev/tty` is a synonym for the current process's controlling terminal, whereas `/dev/pts/N` is a specific pseudoterminal (PTY) slave allocated for a terminal emulator window, SSH session, or interactive container.

> [!tip] Production reality
> **The stdio buffering trap**: Many runtimes (C `stdio`, Python, etc.) check `isatty(stdout)`. When attached to a TTY, `stdout` is **line-buffered** (flushed on every newline); when redirected to a pipe or file (such as [[Container]] log collectors), it switches to **block-buffered** (e.g., 4–8 KB), causing logs to appear delayed or vanish on a crash unless you explicitly flush or run unbuffered (e.g., `PYTHONUNBUFFERED=1`, `stdbuf -oL`).

When a process's `stdout` is redirected from a TTY to a pipe or file, libc switches from ==line buffering== to ==full block buffering==, which can delay or lose logs on crash.

## Why do you pass `-it` to `docker run -it <image> bash`? #card

- `-i` (`--interactive`) keeps `stdin` open so you can send input to the container process.
- `-t` (`--tty`) allocates a **pseudoterminal (PTY)** so interactive shells render prompts, handle Ctrl+C signals, line editing, and colors properly.

## Related

- [[Linux devices]]
- [[Linux Terminal]]
- [[Container]]
