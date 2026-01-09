# Building a Watch Party with Shaka Player

This tutorial shows how to build a "watch party" using Shaka Player and the `@pubnub/shaka-player` package for real-time synchronization.

**The concept:** One person (the "host") controls playback — everyone else stays in sync automatically.

## Prerequisites

- **Node.js 18+**
- **A PubNub keyset** — [Sign up here](https://admin.pubnub.com/)

## How It Works

The `@pubnub/shaka-player` package handles real-time sync (play, pause, seek, drift correction) while Shaka Player streams video from your CDN. Video bytes come from your media server; only small sync commands flow through PubNub.

```
┌─────────────────────────────────────────────────────┐
│              PubNub (Sync Commands)                 │
└─────────────────────────────────────────────────────┘
         ▲              │              │
    publish         subscribe      subscribe
         │              ▼              ▼
    ┌────────┐     ┌────────┐    ┌────────┐
    │  HOST  │     │ VIEWER │    │ VIEWER │
    └────┬───┘     └────┬───┘    └────┬───┘
         │              │              │
         ▼              ▼              ▼
┌─────────────────────────────────────────────────────┐
│           CDN / Media Server (HLS/DASH)             │
└─────────────────────────────────────────────────────┘
```

## Setup

```bash
npm create vite@latest shaka-watch-party -- --template vanilla-ts
cd shaka-watch-party
npm install @pubnub/shaka-player
```

Add your PubNub keys to `.env.local`:

```bash
VITE_PUBNUB_PUBLISH_KEY=pub-c-...
VITE_PUBNUB_SUBSCRIBE_KEY=sub-c-...
VITE_PUBNUB_USER_ID=your-user-id
```

## Implementation

### HTML (`index.html`)

```html
<body>
  <video id="video" controls autoplay style="width: 100%; max-width: 900px;"></video>
  <script type="module" src="/src/main.ts"></script>
</body>
```

### TypeScript (`src/main.ts`)

```typescript
import "@pubnub/shaka-player";

const videoEl = document.querySelector<HTMLVideoElement>("#video")!;
const manifestUrl = "https://storage.googleapis.com/shaka-demo-assets/bbb-dark-truths/dash.mpd";

function getRoomConfig() {
  const params = new URLSearchParams(window.location.search);
  return {
    roomId: params.get("room") ?? "demo",
    role: (params.get("role") ?? "viewer") as "host" | "viewer",
  };
}

async function main() {
  shaka.polyfill.installAll();
  if (!shaka.Player.isBrowserSupported()) {
    throw new Error("Browser not supported");
  }

  const player = new shaka.Player();
  await player.attach(videoEl);
  await player.load(manifestUrl);

  const { roomId, role } = getRoomConfig();

  // Create the SyncManager
  const syncManager = new shaka.sync.SyncManager(player, {
    publishKey: import.meta.env.VITE_PUBNUB_PUBLISH_KEY,
    subscribeKey: import.meta.env.VITE_PUBNUB_SUBSCRIBE_KEY,
    userId: import.meta.env.VITE_PUBNUB_USER_ID,
    maxDriftThreshold: 0.5, // seconds
    syncIntervalMs: 5000,   // ms
  });

  syncManager.connect(roomId);

  if (role === "host") syncManager.becomeMaster();
  else syncManager.becomeFollower();

  // Optional: listen for host changes
  syncManager.addEventListener("masterchanged", (e: any) => {
    console.log("New host:", e.newMasterId);
  });
}

main().catch(console.error);
```

## Running

```bash
npm run dev
```

Open two browser tabs:
- **Host:** `http://localhost:5173/?room=demo&role=host`
- **Viewer:** `http://localhost:5173/?room=demo&role=viewer`

Play, pause, or seek on the host — viewers follow automatically.

## Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `maxDriftThreshold` | `0.5` | Seconds of drift before correction |
| `syncIntervalMs` | `5000` | How often to send sync pulses |
| `autoElectMaster` | `false` | Auto-elect new host if current leaves |

### Use Case Tuning

| Use Case | Threshold | Interval | Notes |
|----------|-----------|----------|-------|
| Sports/Live | 0.3s | 3000ms | Tighter sync |
| Movie Night | 0.5s | 5000ms | Good balance |
| Background | 1.0s | 10000ms | Lower bandwidth |

## Additional Features

### Presence (Who's Watching)

```typescript
syncManager.addEventListener('userjoined', (e) => console.log('Joined:', e.userId));
syncManager.addEventListener('userleft', (e) => console.log('Left:', e.userId));
const viewers = syncManager.getViewers();
```

### Host Transfer

```typescript
// Transfer host role to another user
syncManager.transferMaster(newHostId);
```

### Handle Reconnection

```typescript
syncManager.addEventListener('disconnected', () => showNotification('Reconnecting...'));
syncManager.addEventListener('reconnected', () => showNotification('Reconnected!'));
```

## Mobile Notes

Mobile browsers require user interaction before playing video. Wrap your setup in a click handler:

```typescript
document.getElementById('join-btn').addEventListener('click', async () => {
  await videoElement.play();
  syncManager.connect(roomId);
});
```

## Production Security

For production, configure [PubNub Access Manager](https://www.pubnub.com/docs/general/security/access-control) to:
- Restrict publish access to hosts only
- Use time-limited tokens
- Create private rooms requiring authentication

## Resources

- **[@pubnub/shaka-player](https://www.npmjs.com/package/@pubnub/shaka-player)** — The sync package
- **[Shaka Player Docs](https://shaka-player-demo.appspot.com/docs/api/index.html)** — Full API reference
- **[Shaka Player Demo](https://shaka-player-demo.appspot.com/demo/)** — Try features live

*Happy syncing! 🎬🍿*
