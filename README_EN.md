<p align="right">
  <a href="README.md">Русский</a> | <strong>English</strong>
</p>

<div align="center">

<h1>Ser-Net for Android</h1>

<p><b>An Android VPN client built on Xray-core: subscriptions, TUN tunnel, per-app routing and a Quick Settings tile.</b></p>

<p>
  <a href="https://github.com/bgkv44/Ser-Net-android/releases"><img alt="Release" src="https://img.shields.io/github/v/release/bgkv44/Ser-Net-android?include_prereleases&amp;label=release&amp;color=7ee0ad"></a>
  <a href="https://github.com/bgkv44/Ser-Net-android/releases"><img alt="Downloads" src="https://img.shields.io/github/downloads/bgkv44/Ser-Net-android/total?label=downloads&amp;color=82b8ff"></a>
  <img alt="Platform" src="https://img.shields.io/badge/platform-Android%207.0%2B-3DDC84?logo=android&amp;logoColor=white">
  <img alt="Core" src="https://img.shields.io/badge/core-Xray--core-6f42c1">
  <img alt="Status" src="https://img.shields.io/badge/status-beta-ffd27d">
</p>

<p>
  <a href="https://github.com/bgkv44/Ser-Net-android/releases">Download</a> ·
  <a href="https://github.com/bgkv44/Ser-Net">Windows version</a> ·
  <a href="https://t.me/secureNet_Support_new">Support</a> ·
  <a href="README.md">Русский</a>
</p>

</div>

**Contents:** [About](#about) · [Features](#features) · [Protocols](#supported-protocols) · [Tunnel](#how-the-tunnel-works) · [Requirements](#requirements) · [Installation](#installation) · [First connection](#first-connection) · [Tile and notification](#tile-and-notification) · [Routing](#routing) · [Settings](#settings) · [Updates](#updates) · [Troubleshooting](#troubleshooting) · [Support](#support)

## About

Ser-Net for Android is a mobile VPN client built on [Xray-core](https://github.com/XTLS/Xray-core). It reads servers from a standard provider subscription link, brings up a tunnel through the system `VpnService`, and lets you decide where traffic goes: keep Russian sites direct, send everything through the VPN, or tunnel only the apps you pick.

It shares the idea of the [Windows version](https://github.com/bgkv44/Ser-Net) but is implemented separately for Android: the tunnel is held by `VpnService`, the Xray core runs from Kotlin through LibXray, and the app is controlled from the notification, the Quick Settings tile and its own UI.

> [!NOTE]
> Ser-Net does not provide VPN servers, subscriptions or infrastructure access. You need your own compatible link or subscription from a provider.

## Features

| Feature | Details |
|---|---|
| **Subscriptions** | Several links at once, cards with remaining traffic and expiry date, per-subscription refresh without switching the others. |
| **Server selection** | Search by name or country, latency for every node, and “not checked” labels instead of invented numbers. |
| **Latency checks** | Five methods: `Ser-Net` (a real HTTP request through the server), `GET`/`HEAD` through the server, `TCP`, `ICMP`. Method and timeout are configurable. |
| **TUN tunnel** | A dedicated `VpnService`: the virtual interface covers the whole device and core sockets are protected with `protect()`. |
| **Per-app routing** | Tunnel only selected apps, or every app except the excluded ones. |
| **Routing modes** | Smart, All traffic through VPN, and Selected only, with ready-made service presets and custom domains. |
| **Quick Settings tile** | Connect and disconnect with one tap; state and server name are shown on the tile. |
| **Notification** | Tunnel status, speed and Pause / Start / Cancel buttons without opening the app. |
| **DPI evasion** | TLS fragmentation for TLS/REALITY profiles when connections are blocked by SNI. |
| **Ad blocking** | Ad-domain filtering inside the core. |
| **Protect at startup** | On launch the app checks servers and connects to the fastest one. |
| **Subscription input** | Paste a link from the clipboard or scan a QR code with the camera. |
| **Appearance** | Light, dark or system theme; the layout respects cutouts and the keyboard. |
| **Diagnostics** | App, core and service versions, HWID and device model under About. |

## Supported protocols

| Protocol | Support |
|---|---|
| VLESS | ✅ |
| VMess | ✅ |
| Trojan | ✅ |
| Shadowsocks | ✅ |

Hysteria2 is not supported: Xray-core does not work with it.

## Supported transports

TCP · WebSocket (WS) · gRPC · XHTTP · HTTP

The availability of a specific transport depends on the server configuration and its compatibility with Xray-core. `geoip:`/`geosite:` rules work: geodata ships with the app and is unpacked into internal storage on first launch.

## How the tunnel works

1. You press the connect button; the app asks for VPN permission once.
2. `VpnService` creates the `tun0` interface and hands its descriptor to the Xray core.
3. The core dials the selected server; every core socket goes through `VpnService.protect()`, so traffic never loops back.
4. DNS inside the tunnel is served by a Go resolver bound to a protected socket.
5. On disconnect the service stops the core and closes the interface; the app keeps running, settings and the selected server are preserved.

Android has a single tunnel type — TUN. There is no “system proxy” mode like on Windows: a mobile system needs the virtual interface.

## Requirements

- Android 7.0 (API 24) or newer. Built and tested on Android 14.
- A CPU with `arm64-v8a` (most devices), `armeabi-v7a` (older 32-bit), `x86`/`x86_64` (emulators).
- Your own connection link or subscription.
- The VPN permission Android asks for on the first connection.

## Installation

1. Open the [releases page](https://github.com/bgkv44/Ser-Net-android/releases).
2. Download the APK for your CPU:

| File | ABI | Who it is for |
|---|---|---|
| `Ser-Net_1.0.0-beta.1.6_arm64-v8a.apk` | `arm64-v8a` | almost every phone and tablet since 2017 |
| `Ser-Net_1.0.0-beta.1.6_armeabi-v7a.apk` | `armeabi-v7a` | older 32-bit devices |
| `Ser-Net_1.0.0-beta.1.6_x86.apk` | `x86` | 32-bit emulators |
| `Ser-Net_1.0.0-beta.1.6_x86_64.apk` | `x86_64` | emulators and x86 tablets |

   Not sure about your ABI? Pick `arm64-v8a`.
3. Allow installation from this source when Android asks.
4. Open the file and install the app.

Current builds are marked **beta**: test versions that may change between releases. Every APK is signed with the same key, so a new version installs over the previous one without losing data.

## First connection

1. Open **Subscriptions** in the bottom navigation and press **Add subscription**.
2. Paste a link from the clipboard or scan a QR code, then press **Load**.
3. With several subscriptions, select the active one; its servers appear on the main screen.
4. On the **Connection** screen pick a server and press the round power button.
5. Grant the VPN permission in the system dialog and, if needed, disable battery optimisation for Ser-Net: some firmware closes background work when the screen is off.

## Tile and notification

- The **Ser-Net tile** in the Quick Settings panel connects and disconnects the VPN with one tap and shows the selected server. A long press opens the app itself instead of the system’s App info screen.
- The **notification** keeps the tunnel in foreground mode and lets you pause, resume or cancel the connection without opening the app. On Android 13+ it needs the notification permission.

## Routing

| Mode | Behaviour |
|---|---|
| **Smart** | Russian sites and the local network stay direct; everything else goes through the VPN. |
| **All traffic through VPN** | Everything is tunnelled except local networks and your exceptions. |
| **Selected only** | Only rules in the “Through VPN” list are tunnelled; everything else stays direct. |

Rules come from service presets, custom domains and the app list: tunnel only selected apps, or exclude them from the tunnel. On an active connection, changes are applied with the **Apply** button.

## Settings

- **Connection** — tunnel mode and its options.
- **DPI evasion** — TLS fragmentation for TLS/REALITY profiles.
- **Ads and trackers** — ad-domain filtering.
- **Protect at startup** — connect to the fastest server when the app starts.
- **Route** — routing mode and the app list.
- **Latency check** — method, test URL and timeout.
- **Appearance** — light, dark or system theme.
- **About** — app, core and service versions, HWID and device.

> [!WARNING]
> A subscription URL is access to your account. Do not post it in chats or attach it to issues. The app version and reproduction steps are enough for diagnostics.

## Updates

There is no in-app updater on Android: download the new APK from the [releases page](https://github.com/bgkv44/Ser-Net-android/releases) and install it over the current build. Subscriptions, the selected server and settings are kept; app data is not removed.

## Troubleshooting

- **No servers appeared** — check the subscription link and press “Refresh”. A failed download keeps the previous list.
- **Cannot connect** — try another server and another latency method; make sure the VPN permission is granted.
- **The tunnel drops by itself** — disable battery optimisation for Ser-Net and remove the app from the firmware’s background restrictions (MIUI, ColorOS, OneUI).
- **A site takes the wrong route** — check the routing mode, the rule preset and the app list, then press “Apply”.
- **Need details** — open Settings → About and note the app, core and service versions.

## Support

- Telegram: [@secureNet_Support_new](https://t.me/secureNet_Support_new)
- Bugs and ideas: [GitHub Issues](https://github.com/bgkv44/Ser-Net-android/issues)
- Windows version: [bgkv44/Ser-Net](https://github.com/bgkv44/Ser-Net)

Include the version from About and your device model. Never send your subscription link or passwords.

## Source code

This repository publishes ready-to-use builds and Android documentation. Application source code is not hosted here, so the program cannot be built from this repository — use the files on the [releases page](https://github.com/bgkv44/Ser-Net-android/releases).

## Important

Check the legal requirements in your jurisdiction before using or distributing the program. Do not use or distribute it where it is prohibited or restricted by applicable law.

---

<div align="center">
  <sub>Ser-Net · <a href="#">Back to top</a> · <a href="README.md">Русская версия</a> · <a href="https://github.com/bgkv44/Ser-Net-android/releases">Releases</a></sub>
</div>