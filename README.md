# Always Close

A private, real-time distance tracker for two people. Built as a PWA — no app store, no install, just a URL. Open it on both phones, pick your side, and it shows how far apart you are at any moment.

---

## Project Files

| File | Purpose |
|---|---|
| `index.html` | The entire app — HTML, CSS, JS in one file |
| `manifest.json` | PWA manifest — enables "Add to Home Screen" with icon |
| `icon.svg` | Home screen icon (dark background, terracotta heart) |

---

## Features

### V1 — Distance
- Real-time distance between two phones via browser geolocation
- Haversine formula — accurate great-circle distance
- Auto-scales: shows metres, km, or ♡ (when together)
- Partner online status + "Updated Xs ago" freshness indicator

### V2 — Next Meeting Countdown
- Either person sets a date/time via datetime picker
- Both phones instantly see a live countdown (days · hrs · min)
- Synced via Firebase Realtime Database
- Auto-resets when meeting has passed with prompt to set next one

### V3 — Heat + Heartbeat + Shared Note
- **Background heat** — background shifts from cream → warm blush as distance decreases; pulse rings accelerate; distance number glows
- **Send a pulse** — tap the EKG line button; partner's screen flashes a warm radial burst. Vibrates on receipt (Android). Browser notification fires if tab is in background
- **Shared note** — one persistent line of text, either person can update. Shows who wrote it. Saves on blur or Enter. Partner notified via vibration + browser notification on update
- Meeting changes also trigger vibration + notification

---

## Tech Stack

- **Pure HTML/CSS/JS** — no framework, no build step
- **Firebase Realtime Database** — live sync for location, meeting time, note, heartbeat
- **Web Geolocation API** — `watchPosition` for continuous GPS updates
- **Web Vibration API** — haptic feedback on events (Android Chrome)
- **Web Notifications API** — browser notifications when tab is backgrounded
- **PWA** — installable via "Add to Home Screen" on Android and iOS

---

## Firebase Database Structure

```
alwaysclose_hubby_wifey/
  me/
    lat: number
    lng: number
    ts:  timestamp
  her/
    lat: number
    lng: number
    ts:  timestamp
  nextMeeting: timestamp
  note/
    text: string
    from: "me" | "her"
    ts:   timestamp
  heartbeat/
    from: "me" | "her"
    ts:   timestamp
```

---

## Setup & Deployment

### Prerequisites
- Node.js (v18+)
- Firebase CLI: `npm install -g firebase-tools`

### First-time Setup
```bash
firebase login
mkdir always-close && cd always-close
# drop index.html, manifest.json, icon.svg into this folder
firebase init hosting
# public dir: .  |  single-page app: N  |  overwrite index.html: N
firebase deploy
```

### Subsequent Deploys
```bash
firebase deploy
```

### Firebase Console Setup
1. Create project at console.firebase.google.com
2. Click **</>** → Register web app → copy config into `index.html`
3. Build → **Realtime Database** → Create → test mode
4. Project Settings → **Cloud Messaging** → Generate VAPID key *(for future FCM)*

---

## How It Works

Both people open the same URL and pick **Hubby** or **Wifey**. Each phone:
1. Requests geolocation permission and starts `watchPosition`
2. Writes `{lat, lng, ts}` to its Firebase node on every position update
3. Listens to the partner's node in real-time
4. Calculates Haversine distance and updates the display

All events (note, meeting, heartbeat) are written to Firebase and both clients listen — so updates appear instantly on both sides.

---

## Security Notes

- The Firebase config in the HTML is **not secret** — it's a client identifier, not a credential
- Currently running in **test mode** — anyone with the project ID can read/write
- Recommended Firebase rules (set in Console → Realtime Database → Rules):

```json
{
  "rules": {
    "alwaysclose_hubby_wifey": {
      ".read": true,
      ".write": true
    },
    "$other": {
      ".read": false,
      ".write": false
    }
  }
}
```

- Test mode **expires in 30 days** — apply the rules above before then
- No authentication — the app is secured by obscurity of the URL and room ID

---

## Known Limitations

- **Background GPS** — browser geolocation is throttled when the screen is off or the tab is minimised. Best results with screen on or tab pinned in Chrome
- **Notifications** — Web Notifications only fire if the browser tab is open (even if backgrounded). True lock-screen push requires FCM (planned V3.1)
- **iOS** — Vibration API not supported on iOS. Notifications require iOS 16.4+ with PWA installed to home screen

---

## Roadmap

| Version | Feature |
|---|---|
| ✅ V1 | Live distance |
| ✅ V2 | Next meeting countdown |
| ✅ V3 | Background heat, shared note, pulse button, vibration, browser notifications |
| 🔜 V3.1 | FCM push notifications (true lock-screen push via Firebase Cloud Functions + Service Worker) |

---

## Live URL

[https://alwaysclose-305fb.web.app](https://alwaysclose-305fb.web.app)
