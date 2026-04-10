# MiniStore

> Sideload apps on any non-jailbroken iOS device

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://makeapullrequest.com)
[![Nightly Build](https://github.com/The-Big-Mini/MiniStore/actions/workflows/nightly.yml/badge.svg)](https://github.com/The-Big-Mini/MiniStore/actions/workflows/nightly.yml)
[![Stable Build](https://github.com/The-Big-Mini/MiniStore/actions/workflows/stable.yml/badge.svg)](https://github.com/The-Big-Mini/MiniStore/actions/workflows/stable.yml)

<p align="center">
  <img src="Screenshots/MiniStore-1.PNG" width="230" alt="Browse apps"/>
  &nbsp;
  <img src="Screenshots/MiniStore-2.PNG" width="230" alt="App detail"/>
  &nbsp;
  <img src="Screenshots/MiniStore-3.PNG" width="230" alt="My Apps"/>
</p>

---

MiniStore is a self-hosted iOS app store that lets you install and refresh sideloaded apps using only your Apple ID. It is a fork of [SideStore](https://github.com/SideStore/SideStore), with the VPN layer stripped out and replaced by [minimuxer](https://github.com/jkcoxson/minimuxer) — a lightweight lockdown muxer that runs entirely inside iOS's sandbox. There is no companion desktop app, no USB cable, and no network extension required.

Apps are resigned with your personal development certificate and periodically refreshed in the background before the standard 7-day limit is reached. The whole process happens on-device.

---

## What makes MiniStore different

| | MiniStore | SideStore |
|---|---|---|
| VPN / NetworkExtension | **No** | Yes |
| On-device lockdown muxer | minimuxer (bundled) | minimuxer |
| Desktop companion required | **No** | No |
| Built-in curated source | Mini's Repo | — |
| Nightly + Stable channels | **Yes** | Yes |
| OLED dark mode | **Yes** | — |
| JIT support (SideJITServer + built-in) | **Yes** | Yes |
| App backup & restore | **Yes** | Yes |
| Alternate app icons | **Yes** | — |

---

## Requirements

**To run MiniStore:**
- iPhone or iPad running **iOS 14** or later

**To build from source:**
- **macOS** with Xcode 16 or later (Xcode 26+ for CI)
- **Rustup** (`brew install rustup`) — required to compile minimuxer
- **ldid** (`brew install ldid`) — for fake-signing in local builds

---

## Building

Clone the repository with its submodules:

```bash
git clone --recursive https://github.com/The-Big-Mini/MiniStore.git
cd MiniStore
```

Then pick your workflow:

```bash
make build          # xcodebuild archive → SideStore.xcarchive
make fakesign       # ldid fake-sign all extensions with Release entitlements
make ipa            # package the archive → SideStore.ipa
make build-and-test # build + run unit tests on the simulator
make clean          # remove SideStore.ipa and the build/ directory
```

For local paid-certificate builds, copy `CodeSigning.xcconfig.sample` to `CodeSigning.xcconfig` and fill in your team ID and signing identity. That file is gitignored and never committed.

---

## Project layout

```
AltStore/           Main app target (Swift + ObjC)
  Browse/           Source browser tab — search and add sources
  My Apps/          Installed apps, expiry countdown, refresh
  Managing Apps/    AppManager — install / refresh / revoke pipeline
  Operations/       NSOperation subclasses for each async task
  Components/       Shared UI cells, buttons, reusable views
  Settings/         Preferences, alternate icons, export database
AltStoreCore/       Shared framework — CoreData models, extensions
AltWidget/          WidgetKit home-screen widget
AltBackup/          Companion backup app (bundled inside main IPA)
SideStore/          minimuxer Swift bridge + network interface layer
  MinimuxerWrapper.swift   Rust ↔ Swift FFI bridge
  IfManager.swift          Wi-Fi interface introspection
Shared/             ObjC/C headers shared across targets
xcconfigs/          Per-target xcconfigs (all inherit Build.xcconfig)
Build.xcconfig      Single source of truth for version and bundle IDs
side.json           Self-update source — stable channel
sidenightly.json    Self-update source — nightly channel
scripts/ci/         Python CI helpers (release notes, metadata, deploy)
.github/workflows/  CI pipelines: pr / nightly / stable / deploy
```

---

## Architecture

MiniStore is a standard sandboxed iOS application. The core sideloading pipeline is a chain of `NSOperation` subclasses that run serially inside an `OperationQueue`:

1. **Download** — fetches the IPA and any declared dependencies
2. **Verify** — checks the SHA-256 hash, version, and declared permissions against the source
3. **Authenticate** — signs in to Apple's developer portal and obtains an app-specific password
4. **Fetch Provisioning Profiles** — registers App IDs, updates features, and downloads profiles
5. **Resign** — re-signs the IPA with your personal certificate using AltSign
6. **Send** — transfers the resigned IPA to minimuxer via the AFC protocol
7. **Install** — triggers installation through the lockdown service

Refreshing apps follows the same pipeline, minus the Download step. Background refresh runs via `BGProcessingTask` and respects iOS's background execution budget.

---

## Sources

MiniStore ships with two built-in sources:

| Source | What it contains |
|---|---|
| **Mini's Repo** (`OofMini.github.io`) | Curated collection of tweaked and open-source apps |
| **MiniStore Updates** | MiniStore itself — stable and nightly channels |

You can add any AltStore-compatible source from the Browse tab. The self-update sources are hidden from the Sources list to keep things uncluttered.

---

## CI

| Workflow | Trigger | Output |
|---|---|---|
| `pr.yml` | PR open / push to PR branch | `MiniStore-pr.N+sha.ipa` attached to the PR |
| `nightly.yml` | Every day at 00:00 UTC | Nightly release (if new commits exist) |
| `stable.yml` | Tag push or manual dispatch | Stable release |
| `deploy_public.yml` | Manual dispatch | Mirrors stable + nightly to MiniStore-Public |

---

## Contributing

Pull requests are welcome. For significant changes, please open an issue first to discuss what you'd like to change.

When submitting a PR:
- Keep changes focused — one logical change per PR
- Follow the existing code style (Swift, no force-unwraps at call sites, `Logger` for all logging)
- Do not add VPN-related code or NetworkExtension framework references
- CI builds are automatically attached to every PR for easy testing

---

## Acknowledgements

MiniStore builds on the work of several open-source projects:

- [AltStore](https://github.com/rileytestut/AltStore) by Riley Testut — the original self-hosted app store
- [SideStore](https://github.com/SideStore/SideStore) — the community fork this project is based on
- [minimuxer](https://github.com/jkcoxson/minimuxer) by jkcoxson — the on-device lockdown muxer
- [Roxas](https://github.com/rileytestut/Roxas) by Riley Testut — shared iOS utility framework
- [AltSign](https://github.com/rileytestut/AltSign) by Riley Testut — Apple Developer portal and code signing

---

## License

MiniStore is licensed under the **GNU Affero General Public License v3.0**.  
See [LICENSE](LICENSE) for the full text.
