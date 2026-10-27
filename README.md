# uison-ssh-player

Run a [uison](https://github.com/uisonlab/uison) program on a remote server over SSH and use its UI in a local window.

The server needs `sshd` and a POSIX shell, nothing else: no agent, no SFTP, no X forwarding. The program runs in uison's
console mode (`--console`): it writes frames as JSON lines to stdout and reads events from stdin. The player draws those
frames in its own window and sends the mouse and keys back, so the program itself knows nothing about SSH or the network.

```text
 uison-ssh-player                           server: any sshd, nothing installed
┌──────────────────────────┐      SSH       ┌────────────────────────────────────┐
│ 1. connect, check host   │ ─────────────▶ │ key or password                    │
│ 2. upload (optional)     │ ──── exec ───▶ │ cat > ~/uison/prog && chmod +x     │
│ 3. run                   │ ──── exec ───▶ │ ~/uison/prog args --console        │
│ App / JSON tabs          │ ◀─── stdout ── │ frames                             │
│                          │ ──── stdin ──▶ │ events, actions                    │
└──────────────────────────┘                └────────────────────────────────────┘
```

## Usage

```sh
cargo install uison-ssh-player --git https://github.com/uisonlab/uison-ssh-player
uison-ssh-player [[user@]host[:port]] [-i key]
```

or from a clone:

```sh
cargo run --release -- me@server.example -i ~/.ssh/id_ed25519
```

The start screen has three steps:

1. **Connect:** host, port, user; a key file (with its passphrase, if any) or a password. On a phone there is no
   `~/.ssh`: **Choose key…** opens the system file chooser and keeps a private copy of the key in the app's own folder;
   **New key** makes an ed25519 key on the device and shows the line to add to `~/.ssh/authorized_keys`.
2. **Upload** (optional): a file on this computer goes to a path on the server, `~/uison/<name>` by default. The folder
   is created and the file made executable (`mkdir -p … && cat > path && chmod +x path`).
3. **Run:** the path on the server and its arguments; `--console` is added unless you untick it.

The connection stays open between runs; **Stop** ends the program and keeps the connection. The **App** tab shows the
program, the **JSON** tab every message in both directions with stderr, filters and a field for sending your own line.

The program must be built for the server (for a Linux server from a Mac, for example,
`--target x86_64-unknown-linux-gnu`). Any uison program works, e.g. Disk Usage from the uison repository:
`disk-usage --console /path`.

## Host keys

The server's key is checked against `~/.ssh/known_hosts` and the player's own list, `~/.config/uison/known_hosts`
(override with `UISON_KNOWN_HOSTS`). A new host is remembered and its fingerprint shown; a changed key is refused.

## Platforms

The player is an ordinary uison program: macOS, Windows, Linux (Wayland and X11), and Android (checked on Android 16;
a desktop-sized program is drawn scaled down). An APK is built with `tools/android.sh` from the uison repository, which
packages a crate's binary without Gradle.

## How it is built

The player is the [`uison-player`](https://crates.io/crates/uison-player) library with an SSH link: `uison-player`
provides the window, the App and JSON tabs and the `Link` trait; this crate implements `Link` over an SSH channel
([russh](https://crates.io/crates/russh) with `ring`, tokio on one thread) and replaces the start screen
(`launcher.json`). These are the only external dependencies: uison itself depends on `std` only.

A copy of this crate also lives in the uison repository (`crates/uison-ssh-player`) as an example of a player with a
custom link.

## License

MIT
