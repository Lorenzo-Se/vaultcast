# Vaultcast

**Vaultcast** is a [Raycast](https://raycast.com) extension to search, copy and paste credentials from any CLI-based password manager.

The extension is built around a pluggable **adapter system**: built-in adapters ship with the extension, and third-party adapters can be added at runtime through a simple stdio protocol — no fork or rebuild required.

![Platforms](https://img.shields.io/badge/platform-macOS%20%7C%20Windows-lightgrey)
![License](https://img.shields.io/badge/license-MIT-blue)
![Raycast](https://img.shields.io/badge/Raycast-extension-FF6363)

## Features

- **Fast vault search** across one or all configured password managers at once
- **Copy or paste** passwords, usernames, emails and TOTP codes into the frontmost app
- **Open in browser** or jump straight to the item in the password manager
- **Multi-manager switcher** with available, locked and unavailable managers grouped
- **Pluggable adapters** — add any password manager via an external script and a JSON manifest
- **Extension session** — unlock once, then silently re-unlock managers until the session times out
- **Touch ID / Apple Watch / device passcode** to resume a locked session (macOS)
- **PIN unlock** for Proton Pass (6-digit session PIN)
- **Optional TOTP auto-copy** a few seconds after copying a password

### Supported password managers

| Manager                                    | Type     | Notes                                       |
| ------------------------------------------ | -------- | ------------------------------------------- |
| [Proton Pass](https://proton.me/pass)      | Built-in | Uses `pass-cli`, supports session PIN       |
| _Your manager_                             | External | Write an adapter — see [Adapters](#adapters) |

## Requirements

- [Raycast](https://raycast.com) on macOS or Windows
- Node.js 22+ and npm
- The CLI of your password manager (e.g. `pass-cli` for Proton Pass)
- macOS only, for Touch ID: Xcode Command Line Tools (`xcode-select --install`) — the full Xcode app is **not** required

## Installation

The extension is not published in the Raycast Store. Install it locally:

```bash
git clone https://github.com/Lorenzo-Se/raycast_pwm.git
cd raycast_pwm
npm install
npm run dev
```

Then open Raycast and run **Search Passwords**.

To install a persistent local build instead of running dev mode:

```bash
npm run build
```

## Usage

| Action           | macOS     | Windows     |
| ---------------- | --------- | ----------- |
| Paste password   | `⌘ ⇧ ↩`   | `Ctrl ⇧ ↵`  |
| Copy password    | `⌘ ⇧ C`   | `Ctrl ⇧ C`  |
| Copy email       | `⌘ C`     | `Ctrl C`    |
| Copy TOTP        | `⌘ T`     | `Ctrl T`    |

All other actions (open in browser, paste email, copy username, open in password manager, lock session, reload items) are available from the action panel (`⌘ K`).

Use the dropdown in the search bar to switch between managers or to search **all managers** at once.

## Configuration

| Preference                           | Description                                                                                                                         |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| Default Manager                      | Built-in adapter to use by default, or **Auto** for the first available one                                                         |
| Default Manager Override             | Adapter ID to use as default (e.g. for external adapters). Takes precedence over the dropdown                                       |
| External Adapters Directory          | Folder containing third-party adapters, e.g. `~/.config/raycast-pwm/adapters`                                                       |
| CLI Path Overrides (JSON)            | macOS: `{"1password": "/opt/homebrew/bin/op"}` · Windows: `{"protonpass": "%LOCALAPPDATA%\\Programs\\pass-cli\\pass-cli.exe"}`      |
| Close window after copying           | Close Raycast after copying a credential (default: on)                                                                              |
| Auto-copy TOTP after password action | Copy the TOTP code 5 seconds after copying or pasting a password (default: off)                                                     |
| Enable extension session             | Keep master password / PIN in the session helper's memory and re-unlock managers silently until timeout or Raycast quits (default: on) |
| Session timeout (minutes)            | Lock after 1–1440 minutes of inactivity (default: 15). With an extension session, resume via Touch ID; otherwise unlock the manager again |
| Remember in Keychain                 | macOS only. Store credentials in the Keychain so Touch ID works after a Raycast restart (default: off)                               |

## Security

- **No plaintext on disk.** Credentials for the extension session are held only in the memory of a detached session helper (`assets/session-helper.mjs`). Nothing is written to `/tmp`.
- **Bound to Raycast.** The session helper survives extension remounts but exits automatically when Raycast quits.
- **Idle lock.** Sessions lock after the configured timeout; resuming requires biometric or device authentication.
- **Keychain is opt-in.** Persisting credentials in the macOS Keychain is disabled by default.
- **Biometrics via a native helper.** Touch ID is handled by a small Swift CLI (`swift/BiometricAuth/main.swift`) compiled with `swiftc` during `npm run dev` / `npm run build`.

> [!NOTE]
> The Raycast Store does not allow Keychain access, so Touch ID re-authentication is intended for local builds only.

## Adapters

### Built-in adapters

1. Create `src/adapters/<manager>.ts` implementing `PasswordManagerAdapter`
2. Register it in `src/adapters/index.ts`
3. Optionally set a CLI path override in the extension preferences

### External adapters

External adapters are standalone scripts that talk to the extension over stdio. Each adapter lives in its own folder with a `pwm-adapter.json` manifest and a script.

```bash
mkdir -p ~/.config/raycast-pwm/adapters
cp -R examples/external-adapter/protonpass ~/.config/raycast-pwm/adapters/protonpass
```

Then set **External Adapters Directory** to `~/.config/raycast-pwm/adapters` in the extension preferences.

- Protocol and authentication: [src/adapters/README.md](src/adapters/README.md)
- Reference adapter: [Proton Pass](examples/external-adapter/protonpass/README.md)

## Project structure

```
src/
├── search-passwords.tsx         # Main command (UI)
├── unlock-form.tsx              # Master password / PIN + Touch ID
├── types.ts                     # Adapter interface
├── registry.ts                  # Adapter registry
├── registry/                    # External adapter loading
├── adapters/                    # Built-in adapters + external protocol
├── session-helper/              # Session helper IPC protocol
├── hooks/                       # React hooks
└── utils/                       # CLI, biometrics, credential vault, helpers

assets/session-helper.mjs        # Session helper (RAM-only, IPC)
swift/BiometricAuth/             # Touch ID + Keychain CLI (macOS)
examples/external-adapter/       # Reference adapter (Proton Pass)
scripts/                         # Build scripts
```

## Development

```bash
npm run dev        # Start Raycast dev mode
npm run build      # Build into ~/.config/raycast/extensions/raycast-pwm
npm run lint       # Lint
npm run fix-lint   # Lint and auto-fix
```

`npm run dev` and `npm run build` compile the biometric helper and the session helper first.

## Contributing

Issues and pull requests are welcome. New password manager support is best added as an external adapter first — see [src/adapters/README.md](src/adapters/README.md).

## License

Released under the [MIT License](LICENSE).
