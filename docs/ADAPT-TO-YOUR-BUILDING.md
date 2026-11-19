# Make a room finder for your building

The room finder has two building-specific parts, and everything else is generic:

1. **`FLOOR`:** the traced floor plan: rooms, open zones, cores, voids and facilities.
2. **`DEFAULT_GEO`:** how plan coordinates map onto real-world GPS coordinates.

You get the second part from a few on-site measurements with the geo-capture tool. The first part you can trace by hand, or hand to an AI coding agent together with the tool's export.

---

## Step 1: Get a floor plan image

Any clear, roughly top-down image works:
- a photo of the evacuation or wayfinding sign (shoot straight on, so it isn't skewed),
- a screenshot of a PDF floor plan,
- an architect's drawing.

Make sure the room names are readable. They're what people will search for.

## Step 2: Measure calibration points on site (≈10 minutes)

1. Open the geo-capture tool on your phone: **<https://geo-capture-tool.netlify.app/>**. Or host `tools/geo-capture.html` yourself; it **must be HTTPS** for GPS. It's one static file and makes no network requests.
2. Fill in the building and floor, and load the floor plan image.
3. Walk to an **outer corner** of the building, on the floor you're mapping and as close to the window/façade as you can get.
4. Tap **Lock position (30 s)** and keep the phone still. The screen stays on. The tool:
   - collects GPS fixes continuously, and asks for a fresh one every 2 s,
   - drops fixes that are much worse than the best one,
   - averages the rest, weighted by their accuracy,
   - reports the best accuracy and the **spread**, i.e. how much the fixes scattered.
5. Tap the plan exactly where you're standing, give the point a name, and **Save**.
6. Repeat at **3–4 corners**, as far apart as possible.

After two points the tool shows the fitted **rotation** (how the plan's "up" is turned from north), **scale** (cm per pixel → building size in metres), and from three points the **fit error** per point. If one point is off by more than ~6 m, redo it.

> **Why corners?** Rotation and scale come from the *distances between* points. Errors of a few metres matter much less over 50 m than over 10 m, so points far apart on the outline beat points in the middle. Outer walls also have the best sky/Wi-Fi visibility.

## Step 3: Export and hand it over

The tool creates three files on your device (nothing is uploaded):

| File | Contents |
|---|---|
| `…-calibration.json` | Points (lat/lng, accuracy, spread, pixel position), the similarity fit, and drop-in `room_finder` values |
| `…-calibration.png` | Your floor plan with numbered points plus a legend |
| `…-agent-prompt.md` | Instructions for an AI coding agent, including a table of the points |

Give all three, **plus the original floor plan image and this repository's `index.html`**, to your agent. The prompt asks it to keep the privacy guarantees and to test against your calibration points.

## Step 4: What the agent (or you) changes in `index.html`

### `FLOOR`

Plan units are arbitrary, but the UI (zoom levels, marker sizes, labels) is tuned for plans **about 900 units across**. The export gives `plan_unit_scale`: multiply image pixels by it to get plan units.

```js
const FLOOR = {
  name: "2nd floor",
  size: [900, 640],                 // = fit.room_finder.FLOOR_size
  shapes: [
    // r: [x, y, width, height] in plan units, origin top-left, y down
    {id:"lobby",  name:"Lobby",      kind:"zone",   fill:"#a9c4e4", r:[0,0,300,200]},
    {id:"void",   name:"Atrium",     kind:"atrium", fill:"#ffffff", r:[350,200,200,200]},  // not walkable
    {id:"core-w", kind:"core",       fill:"#a8413a", r:[300,220,40,160]},                 // stairs/lifts: not walkable
    {id:"r201",   name:"Room 2.01",  kind:"room", meet:true,  fill:"#69983b", r:[600,0,120,90]},
    {id:"f1",     name:"F1",         kind:"room", focus:true, fill:"#f4f4f2", r:[720,0,60,60]},
    // non-rectangular rooms: add poly:[[x,y],…] for drawing; r stays the bounding box used for routing
  ],
  pois: [ {id:"pantry", name:"Pantry", icon:"☕", p:[320,210]} ]
};
```

| `kind` | Drawn | Walkable | Notes |
|---|---|---|---|
| `zone` | yes | yes | open areas, corridors, seating |
| `deco` | yes | yes | colour only, not searchable |
| `room` | yes | **no**, entered at the end of a route | add `meet:true` or `focus:true` for labels and search category |
| `core` | yes | **no** | stairs, lifts, shafts; the dot is pushed out of these |
| `atrium` | yes | **no** | voids; the dot is pushed out of these |

Draw order matters: later shapes paint over earlier ones, so put background zones first.

### `DEFAULT_GEO`

Paste `fit.room_finder.DEFAULT_GEO` from the export:

```js
const DEFAULT_GEO = {
  anchorPlan: [52.3, 31.0],          // calibration point 1, in plan units
  anchorLL:   [52.0001234, 5.1234567],
  rotDeg: -8.25,                     // clockwise angle of the plan's "up" from true north
  facing: 0,
  mPerUnit: 0.07211,                 // metres per plan unit
  calibrated: true
};
```

Then bump the storage key (`rf.geo.v2` → `rf.geo.v3`), so phones that opened an older version pick up the new alignment.

### Also update

- the header text (`<h1>` and subtitle) and `<title>`,
- the Teams manifest (`teams/manifest.json`): name, URLs and `validDomains`. Generate a new `id` (any GUID) and bump `version`,
- `ROOM_MATCH`, if your calendar uses different room names. It matches room `name`s as whole words.

## Step 5: Test before you trust it

- Open `index.html#demo` on a desktop and click around the plan. The dot should follow, routes should go around voids and cores, and every room should be reachable.
- With simulated GPS (browser dev tools → *Sensors*), set each calibration point's lat/lng. The dot must land within a few metres of where the point is marked on the plan.
- On site: walk a known route and check that the heading-up view turns the right way. If the map turns the wrong way, `rotDeg` has the wrong sign.

## Privacy checklist for your fork

Keep these, and the README's privacy claims stay true:
- [ ] No `fetch`/XHR/WebSocket/beacon that carries position or sensor data.
- [ ] No analytics, tag managers, fonts or scripts from third-party CDNs. Inline everything.
- [ ] No storing of positions (no location history in `localStorage`, IndexedDB or cookies).
- [ ] Served as static files over HTTPS, with no backend.
