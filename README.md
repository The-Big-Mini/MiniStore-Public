# MiniStore

> A self-hosted iOS app store — no jailbreak, no AltServer, just your Apple ID.

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
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

MiniStore is a self-hosted iOS app store that lets you install and refresh sideloaded apps using only your Apple ID. It is a fork of [SideStore](https://github.com/SideStore/SideStore), built on [minimuxer](https://github.com/jkcoxson/minimuxer) — a lightweight lockdown muxer that runs inside iOS's sandbox — and [LocalDevVPN](https://github.com/jkcoxson/LocalDevVPN), which provides the loopback tunnel minimuxer needs to communicate with the device.

A computer is needed once to sideload the MiniStore IPA itself — [ILoader](https://github.com/nab138/iloader) is the easiest way to do this via its Import IPA feature. After the initial install, MiniStore handles app installs and refreshes entirely on-device. During refresh, MiniStore will prompt you to connect to the VPN if it isn't already active.

Apps are resigned with your personal development certificate and refreshed in the background before the standard 7-day limit expires.

---

## What makes MiniStore different

| | MiniStore | SideStore |
|---|---|---|
| On-device lockdown muxer | minimuxer (bundled) | minimuxer |
| LocalDevVPN required | Yes (for refresh) | Yes |
| Built-in NetworkExtension target | **No** | Yes |
| VPN tab in the UI | **No** | Yes |
| Built-in curated source | Mini's Repo | — |
| Nightly + Stable update channels | **Yes** | Yes |
| OLED dark mode | **Yes** | — |
| Alternate app icons | **Yes** | — |
| JIT support (SideJITServer + built-in) | **Yes** | Yes |
| App backup & restore | **Yes** | Yes |

---

## Requirements

- iPhone or iPad running **iOS 14** or later
- A computer for the initial sideload — use [ILoader](https://github.com/nab138/iloader) with its Import IPA feature for the easiest experience
- [LocalDevVPN](https://github.com/jkcoxson/LocalDevVPN) installed on the same device for app refreshes

---

## How it works

MiniStore is a standard sandboxed iOS application. When you install or refresh an app, it runs through a pipeline of steps entirely on-device:

1. **Download** — fetches the IPA and any declared dependencies
2. **Verify** — checks the SHA-256 hash, version, and declared permissions against the source
3. **Authenticate** — signs in to Apple's developer portal with your Apple ID
4. **Fetch Provisioning Profiles** — registers App IDs and downloads provisioning profiles
5. **Resign** — re-signs the IPA with your personal development certificate
6. **Send** — transfers the resigned IPA to the device via the AFC protocol
7. **Install** — triggers installation through the lockdown service

Refreshing follows the same steps, skipping Download and Verify. Background refresh runs automatically before the standard 7-day certificate expiry.

---

## Sources

MiniStore ships with two built-in sources:

| Source | What it contains |
|---|---|
| **Mini's Repo** (`OofMini.github.io`) | Curated collection of tweaked and open-source apps |
| **MiniStore Updates** | MiniStore itself — stable and nightly channels |

You can add any AltStore-compatible source from the Browse tab. Users can opt in to nightly builds via **Enable Beta Updates** in Settings.

---

## Acknowledgements

MiniStore builds on the work of several open-source projects:

- [AltStore](https://github.com/rileytestut/AltStore) by Riley Testut — the original self-hosted app store
- [SideStore](https://github.com/SideStore/SideStore) — the community fork this project is based on
- [minimuxer](https://github.com/jkcoxson/minimuxer) by jkcoxson — the on-device lockdown muxer
- [LocalDevVPN](https://github.com/jkcoxson/LocalDevVPN) by jkcoxson — the loopback VPN tunnel
- [Roxas](https://github.com/rileytestut/Roxas) by Riley Testut — shared iOS utility framework
- [AltSign](https://github.com/rileytestut/AltSign) by Riley Testut — Apple Developer portal and code signing
- [ILoader](https://github.com/nab138/iloader) by nab138 — user-friendly sideloader for the initial install

---

## License

MiniStore is licensed under the **GNU Affero General Public License v3.0**.  
See [LICENSE](LICENSE) for the full text.
