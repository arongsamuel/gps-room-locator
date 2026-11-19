# Privacy

Atlas Room Finder is built so that **your location never leaves your device**. This document describes exactly which data exists, where it lives, and who could ever see it. It's written so you can check every claim against the source code.

## Short version

- Your position (GPS, compass, step counts) is processed **only in your browser**, and is **never sent, uploaded or logged** anywhere.
- No location history is kept. When you turn live location off or close the page, the position is gone.
- No accounts, analytics, ads, cookies for tracking, or third-party scripts.
- The optional Outlook feature talks **directly** to Microsoft and only reads your own meetings for today.

## Data flow

```
 Phone sensors ──► browser (index.html) ──► map on your screen
 GPS · compass · accelerometer      │
                                    └─✗ no network path for position data
```

| Data | Where it comes from | Where it goes | Stored? |
|---|---|---|---|
| GPS position | Browser Geolocation API | Used in memory to draw the dot and route | **No** |
| Compass heading | DeviceOrientation API | Used in memory to rotate the map | **No** |
| Accelerometer (steps) | DeviceMotion API | Used in memory for step counting | **No** |
| Selected room | You | In memory, and in the page address (`#room`) | No |
| Map alignment (building reference point) | Built into the file | `localStorage`, on your device only | Yes, local |
| Outlook sign-in token (optional) | Microsoft, after *you* sign in | `localStorage`, on your device only | Yes, local, revocable |
| Today's meetings (optional) | Microsoft Graph, read directly | In memory, to match a room name | **No** |

## Network requests, completely listed

| Request | When | Purpose | Carries your location? |
|---|---|---|---|
| Loading the page from the host | When you open it | Download the static file | No |
| `login.microsoftonline.com` | Only if Outlook is configured **and** you tap 📅 | Sign in (OAuth 2.0 + PKCE, no secret) | No |
| `graph.microsoft.com/v1.0/me/calendarView` | Only when signed in to Outlook | Read today's meetings | No |
| `res.cdn.office.net` (TeamsJS) | Only when opened **inside Microsoft Teams** | Teams integration (SRI-pinned) | No |

Nothing else. You can confirm this in your browser's developer tools (Network tab), or by searching the source for `fetch(`.

## Who can see what

- **The static host** (e.g. Netlify, GitHub Pages) sees ordinary web-server data when you load the page, such as your IP address and the time, like any website. It never receives your position, because the app never sends it.
- **Your phone's location service** (Google / Apple) works out GPS + Wi-Fi positions the same way it does for every app. That's governed by your phone's settings, not by this app.
- **Microsoft** sees your sign-in and calendar request only if you use the optional Outlook feature. That's the same as opening Outlook.
- **Nobody else.** There is no backend, no database, and no analytics service.

## The geo-capture tool

[`tools/geo-capture.html`](tools/geo-capture.html), used once to calibrate a new building, follows the same rules:
- It makes **no network requests at all**.
- The floor-plan image you load is read locally and never uploaded.
- Calibration points exist only in memory until *you* download the export files.
- The exports contain the coordinates of the calibration spots you measured, usually building corners. Share them as you see fit.

## Permissions and how to revoke them

- **Location / Motion & Orientation:** granted per site in your browser. Revoke it any time in browser or phone settings.
- **Outlook:** remove the app's access in your Microsoft account (*My Apps / Permissions*). You can also clear the site's data in your browser.

## Note on the published coordinates

`index.html` contains one fixed reference point (`DEFAULT_GEO.anchorLL`). It's a spot in the open work area of a public building, used to align the floor plan with real-world coordinates. It's not a person's position.
