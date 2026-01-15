<div align="center">

<!-- HERO BANNER - See description below for what to create -->
<!-- Replace this placeholder with your custom banner image -->
<img src="docs/logos/pubnub-shaka-hero.png" alt="PubNub + Shaka Player - Watch Party Video Streaming" width="100%">

# @pubnub/shaka-player

### Real-Time Synchronized Video Playback for the Web

[![npm version](https://img.shields.io/npm/v/@pubnub/shaka-player?color=E11D48&label=npm&logo=npm)](https://www.npmjs.com/package/@pubnub/shaka-player)
[![PubNub](https://img.shields.io/badge/Powered%20by-PubNub-E11D48?logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJ3aGl0ZSI+PGNpcmNsZSBjeD0iMTIiIGN5PSIxMiIgcj0iMTAiLz48L3N2Zz4=)](https://www.pubnub.com/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

---

**🎬 Watch videos together in perfect sync across the globe**

This is NOT the vanilla Shaka Player. This is a **PubNub-enhanced fork** that adds real-time playback synchronization, enabling "Watch Party" experiences where multiple viewers stay perfectly in sync.

[Get Started](#-quick-start) · [Watch Party Demo](demo/sync/) · [API Reference](#-syncmanager-api) · [PubNub Dashboard](https://admin.pubnub.com)

</div>

---

## 🆚 How is this different from Shaka Player?

| Feature | Shaka Player | @pubnub/shaka-player |
|---------|--------------|---------------------|
| DASH/HLS Streaming | ✅ | ✅ |
| DRM Support | ✅ | ✅ |
| Offline Playback | ✅ | ✅ |
| **Real-Time Sync (Watch Party)** | ❌ | ✅ **NEW** |
| **Master/Follower Control** | ❌ | ✅ **NEW** |
| **Automatic Drift Correction** | ❌ | ✅ **NEW** |
| **Presence Events** | ❌ | ✅ **NEW** |

> **Looking for vanilla Shaka Player?** → [shaka-player on npm](https://www.npmjs.com/package/shaka-player)

---

## ⚡ What Can You Build?

<table>
<tr>
<td width="33%" align="center">

### 🎉 Watch Parties
Friends watching movies/shows together remotely

</td>
<td width="33%" align="center">

### 🏟️ Live Events
Synchronized viewing for sports, concerts, premieres

</td>
<td width="33%" align="center">

### 🎓 E-Learning
Instructors controlling video playback for classes

</td>
</tr>
<tr>
<td width="33%" align="center">

### 💼 Corporate Training
Synchronized training videos across offices

</td>
<td width="33%" align="center">

### 🎮 Gaming Streams
Synced esports viewing experiences

</td>
<td width="33%" align="center">

### 🏥 Telehealth
Doctors reviewing scans with patients in real-time

</td>
</tr>
</table>

---

## 🚀 Quick Start

### 1. Install the package

```bash
npm install @pubnub/shaka-player pubnub
```

### 2. Get your PubNub keys (free)

1. Create an account at **[admin.pubnub.com](https://admin.pubnub.com)**
2. Create a new app → Get your **Publish Key** and **Subscribe Key**

### 3. Start syncing!

```javascript
import shaka from '@pubnub/shaka-player';

// Initialize player
const video = document.getElementById('video');
const player = new shaka.Player();
await player.attach(video);
await player.load('https://example.com/manifest.mpd');

// Create SyncManager with your PubNub keys
const syncManager = new shaka.sync.SyncManager(player, {
  publishKey: 'pub-c-YOUR-PUBLISH-KEY',
  subscribeKey: 'sub-c-YOUR-SUBSCRIBE-KEY',
  userId: 'user-123',              // Optional: auto-generated if omitted
  maxDriftThreshold: 0.5,          // Seconds before force-sync (default: 0.5)
  syncIntervalMs: 5000             // Sync pulse interval (default: 5000)
});

// Join a watch party room
syncManager.connect('friday-movie-night');

// Control playback for everyone (or stay as follower)
syncManager.becomeMaster();
```

---

## 🔌 SyncManager API

| Method | Description |
|--------|-------------|
| `connect(roomId)` | Join a sync room |
| `disconnect()` | Leave the current room |
| `becomeMaster()` | Take control of playback for all viewers |
| `becomeFollower()` | Follow the master's playback |
| `getRole()` | Returns `'master'` or `'follower'` |
| `isConnected()` | Returns connection status |
| `getRoomId()` | Returns current room ID |
| `getUserId()` | Returns this client's user ID |
| `destroy()` | Clean up resources |

### Events

```javascript
syncManager.addEventListener('masterchanged', (event) => {
  console.log('New master:', event.newMasterId);
});
```

---

## 🎯 How Sync Works

```
┌─────────────────────────────────────────────────────────────────┐
│                        PubNub Cloud                             │
│                    (Real-Time Messaging)                        │
└───────────────────────────┬─────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   MASTER 👑   │   │  FOLLOWER 1   │   │  FOLLOWER 2   │
│               │   │               │   │               │
│ Play/Pause →──┼───┼──→ Receives ──┼───┼──→ Receives   │
│ Seek →────────┼───┼──→ Receives ──┼───┼──→ Receives   │
│ Sync Pulse →──┼───┼──→ Adjusts ───┼───┼──→ Adjusts    │
└───────────────┘   └───────────────┘   └───────────────┘
```

1. **Master** controls playback (play, pause, seek)
2. Commands are sent instantly via **PubNub** to all connected clients
3. **Followers** receive and apply commands with latency compensation
4. **Periodic sync pulses** correct any drift between clients

---

## 📦 Included Builds

| Build | File | Use Case |
|-------|------|----------|
| **Full + UI** | `shaka-player.ui.js` | Complete player with UI controls |
| **Full** | `shaka-player.compiled.js` | Player without UI |
| **DASH Only** | `shaka-player.dash.js` | Lightweight DASH-only build |
| **HLS Only** | `shaka-player.hls.js` | Lightweight HLS-only build |

All builds include the **SyncManager** for Watch Party functionality.

---

## 🎬 Full Shaka Player Features

This package includes **all features** from [Shaka Player](https://github.com/shaka-project/shaka-player):

<details>
<summary><b>📺 Streaming Formats</b></summary>

- **DASH** - VOD, Live, In-Progress Recording
- **HLS** - VOD, Live, Event, Low-Latency
- Multi-period content
- Multi-codec/container support

</details>

<details>
<summary><b>🔐 DRM Support</b></summary>

| | Widevine | PlayReady | FairPlay | ClearKey |
|:--:|:--:|:--:|:--:|:--:|
| Chrome | ✅ | - | - | ✅ |
| Firefox | ✅ | - | - | ✅ |
| Edge | ✅ | ✅ | - | ✅ |
| Safari | - | - | ✅ | - |

</details>

<details>
<summary><b>📱 Platform Support</b></summary>

| Platform | Chrome | Firefox | Safari | Edge |
|:---------|:------:|:-------:|:------:|:----:|
| Windows | ✅ | ✅ | - | ✅ |
| macOS | ✅ | ✅ | ✅ | ✅ |
| Linux | ✅ | ✅ | - | ✅ |
| Android | ✅ | ✅ | - | - |
| iOS | Native | Native | Native | - |

</details>

<details>
<summary><b>🎨 Additional Features</b></summary>

- Offline storage & playback
- Subtitles (WebVTT, TTML, CEA-608/708)
- Thumbnails support
- VR/360° video
- Monetization with IMA SDK
- Content Steering

</details>

---

## 📖 Documentation

| Resource | Link |
|----------|------|
| **Watch Party Demo** | [demo/sync/](demo/sync/) |
| **Full API Docs** | [shaka-project.github.io/shaka-player/docs/api](https://shaka-project.github.io/shaka-player/docs/api/index.html) |
| **Tutorials** | [Shaka Tutorials](https://shaka-project.github.io/shaka-player/docs/api/tutorial-welcome.html) |
| **PubNub Dashboard** | [admin.pubnub.com](https://admin.pubnub.com) |
| **PubNub Docs** | [pubnub.com/docs](https://www.pubnub.com/docs) |

---

## 🤝 Contributing

We welcome contributions! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## 📄 License

Apache 2.0 - See [LICENSE](LICENSE)

---

<div align="center">

### Built with ❤️ by

<a href="https://www.pubnub.com">
  <img src="https://www.pubnub.com/wp-content/uploads/2024/12/PubNub_Logo_RGB_Flame.svg" alt="PubNub" height="60">
</a>

**[PubNub](https://www.pubnub.com)** - The Real-Time Communication Platform

<br>

*Based on [Shaka Player](https://github.com/shaka-project/shaka-player) by Google*

</div>
