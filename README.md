# GPS Meeting Room Locator

**A privacy-first indoor room finder that runs entirely in your phone's browser.**
Your location never leaves your device: there is no server, no account, no tracking.

Search a room, turn on live location, and follow a walking route on a map that turns with you, like car navigation but indoors. It was built for a real office floor, and it's designed so **anyone can make one for their own building** from a floor plan and a few GPS measurements.

> The screenshots and the `index.html` in this repository use a **made-up demo building** ("Demo Office, Floor 2"). The real deployment's floor plan is intentionally not published.

<p align="center">
  <img src="docs/screenshots/navigate-heading-up.png" width="240" alt="Heading-up navigation to a meeting room (demo building)">
  <img src="docs/screenshots/whole-floor-route.png" width="240" alt="Whole-floor view with walking route (demo building)">
  <img src="docs/screenshots/search.png" width="240" alt="Room search (demo building)">
</p>

### Try it

| | Link | What it is |
|---|---|---|
| 📍 **Geo-capture tool** | **<https://geo-capture-tool.netlify.app/>** | Measure precise GPS points (30 s lock) and pin them to *your* building's floor plan |
| 🧭 **Room finder demo** | [`index.html`](index.html) | Host it, or open `index.html#demo` on a computer and click the plan to simulate walking |

---

## 🔒 Privacy by design

Most indoor-navigation products work by sending your position to a central service. This one doesn't. Your position is computed and used on your own device, and that's where it ends.

| | Typical indoor navigation | GPS Room Locator |
|---|---|---|
| Where your position is computed | Vendor's server / beacons backend | **In your browser** |
| Position sent over the network | Yes, continuously | **Never** |
| Location history stored | Often, server-side | **No**. Nothing kept after you turn it off |
| Account / sign-in required | Usually | **No** (Outlook is optional) |
| Analytics, ads, trackers | Common | **None** |
| Backend to trust or breach | Yes | **None**: static files only |

**How you can check this yourself**
- The whole app is **one HTML file** ([`index.html`](index.html)) with no build step and no dependencies. Read it.
- Open your browser's developer tools → *Network* while using the app. With location on, you'll see **zero requests**.
- The only network calls in the code are to Microsoft (`login.microsoftonline.com`, `graph.microsoft.com`) for the **optional, off-by-default** Outlook calendar feature. They fetch your meetings and never send a location.

See [PRIVACY.md](PRIVACY.md) for the full data-flow description.

---

## Features

- **Room search** for meeting rooms, focus rooms and facilities (kitchen, exits, first aid).
- **Live location** that fuses three sensors on-device:
  - **GPS:** accuracy-weighted, with jumpy fixes rejected.
  - **Step counting:** from the accelerometer (pedestrian dead reckoning).
  - **Compass heading:** smoothed.

  A small Kalman-style filter combines them, so the dot moves smoothly when you walk and doesn't drift while you stand still.
- **Heading-up navigation (➤):** the map rotates with you, keeps you in the lower part of the screen, and auto-zooms so the destination stays visible. Pinch to zoom manually.
- **Walking routes** that go *around* voids (atria, lightwells), stair/lift cores and closed rooms (grid-based Dijkstra with path smoothing).
- **Map constraints:** the position can't end up inside a void or a core.
- **Optional Outlook integration:** routes you to your next meeting's room. Off until an Entra app ID is configured. Try it without sign-in: `#calendar-demo`.
- No network needed once the page has loaded. No install, no app store.

## Use it

Host the files on any static HTTPS host. Location and motion sensors require HTTPS.

- **Netlify Drop:** drag the folder onto <https://app.netlify.com/drop>.
- **GitHub Pages:** Settings → Pages → deploy from the `main` branch.

Open the page on your phone, tap **Turn live location on**, search a room, and hold the phone flat in front of you.

Handy links (append to your hosted URL):
- `#meet-c`, `#focus-2`: open with a room selected.
- `#demo`: click the plan to simulate a position (desktop testing).
- `#calendar-demo`: made-up meetings to try the calendar flow.
- `#privacy`: the in-app privacy note.

> With the demo building, real GPS will place you "outside the building", because its location is a placeholder. Use `#demo`, or calibrate it for your own building (below).

## Make one for your building

You need a floor-plan image and 10 minutes on site.

1. Open the **geo-capture tool** on your phone: **<https://geo-capture-tool.netlify.app/>** (or host [`tools/geo-capture.html`](tools/geo-capture.html) yourself over HTTPS). Then load a photo or screenshot of the floor plan.
2. Stand at **3–4 outer corners** of the building. At each one, tap **Lock position (30 s)**, then tap the same spot on the plan. The tool averages the GPS fixes and rejects outliers, and fits the plan's **scale and rotation** with an error estimate.
3. Export the **package (.json)**, the **annotated plan (.png)** and the **agent prompt**, and hand them to an AI coding agent. They contain everything needed to trace the plan and georeference it.

<p align="center"><img src="docs/screenshots/geo-capture.png" width="300" alt="Geo-capture tool with the demo floor plan"></p>

The geo-capture tool is also privacy-first. It makes **no network requests**, keeps points only in memory, and creates the export files directly on your device.

Full guide: [docs/ADAPT-TO-YOUR-BUILDING.md](docs/ADAPT-TO-YOUR-BUILDING.md).

## Repository layout

```
index.html                 the room finder (single self-contained file, demo building)
tools/geo-capture.html     30 s GPS lock + floor-plan pinning + agent export (hosted: geo-capture-tool.netlify.app)
docs/                      guide, demo floor plan, screenshots
PRIVACY.md                 what data exists, where it lives, who can see it
```

## Limitations (honest ones)

- Phone GPS indoors is typically accurate to **±5–20 m** and can't tell floors apart. Step counting and map constraints help, but this is not centimetre-level positioning.
- Step direction assumes the phone is held flat, pointing the way you walk.
- Room shapes are traced from a floor-plan image, so they're approximate. Door positions are inferred, and routes enter a room from its nearest wall.

## License

[MIT](LICENSE)
