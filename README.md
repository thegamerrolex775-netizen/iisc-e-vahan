# IISc e-Vahan Navigator — Free OpenStreetMap Edition

## What this project is

This is a small, installable web app for navigating the Indian Institute of Science campus using the three e-Vahan routes supplied for the project: Red, Green and Blue. It is designed to run from GitHub Pages without a Google Maps API key or paid backend.

The interface has four main inputs:

1. **Your location** — requested only when the user presses the separate **My location** button.
2. **Search** — instant IISc campus suggestions from the bundled route/place list.
3. **Map selection** — tap any point on the map to use that latitude/longitude as the destination.
4. **Recent destinations** — stored in the browser and deletable from Settings.

## What changed in v4

The earlier prototype tried to geocode every e-Vahan stop before it could route. That caused the message “0/25 found; 25 may need map selection” when OpenStreetMap did not recognise the campus stop names. v4 removes that startup dependency. The 25 route stops have local campus anchor coordinates so the route algorithm can run immediately.

The route planner also changed from “nearest stop + one line” to a small **multi-modal graph search**. It enumerates direct e-Vahan journeys and one-transfer journeys, then scores them.

### Cost model

For each candidate journey:

`total = walk-to-stop + first wait + e-Vahan ride time + transfer penalty + second wait + final walk`

The candidate with the lowest total is shown as the **Fastest estimated journey**. Up to three alternatives are also shown.

The e-Vahan ride time is currently estimated from the geographic distance between consecutive stops and a configurable average e-Vahan speed. Waiting time is also an estimate. The exact trip timetable is not embedded yet.

This distinction matters: the app is an algorithmic prototype, not an official real-time e-Vahan dispatch system.

## Project structure

All files are deliberately kept in the repository root so the project can be uploaded directly to GitHub Pages. There is no required assets folder.

- `index.html` — complete page structure and UI.
- `iisc-eVahan.css` — visual design, responsive/mobile styling and colours.
- `iisc-eVahan.js` — search, GPS, map selection, stop graph, fastest-route algorithm and route rendering.
- `app-config.js` — OpenStreetMap tile URL, walking router URL and routing assumptions.
- `manifest.webmanifest` — allows installation as a PWA on supported browsers.
- `service-worker.js` — caches the app shell for faster repeat loading.
- `README.md` — this documentation.
- `.nojekyll` — prevents GitHub Pages/Jekyll processing from interfering with the static files.

## Creating the GitHub repository

Recommended repository name:

`iisc-e-Vahan`

Create a **public** repository. Do not add another README, `.gitignore` or licence during repository creation if you plan to upload this project as-is.

After extracting the ZIP, upload the **contents** of the extracted folder directly to the repository root. `index.html` must appear at the top level. Do not upload the ZIP itself.

Expected GitHub layout:

```text
iisc-e-Vahan/
├── index.html
├── iisc-eVahan.css
├── iisc-eVahan.js
├── app-config.js
├── manifest.webmanifest
├── service-worker.js
├── .nojekyll
└── README.md
```

Then open **Settings → Pages**, select **Deploy from a branch**, choose `main` and `/ (root)`, and save. GitHub Pages will publish the site.

The resulting address normally follows:

`https://YOUR-USERNAME.github.io/iisc-e-Vahan/`

## How to use the website

### 1. Find your starting location

Open the site on your phone and press **My location**. Allow browser location permission. The app does not request GPS until this button is pressed.

### 2. Choose a destination

Type a destination. Suggestions appear after every letter. You can also tap the map directly. A map tap becomes a precise latitude/longitude destination, even when it is not one of the named IISc places.

### 3. Find the fastest e-Vahan journey

Press **Find fastest route**. The algorithm checks the e-Vahan network rather than simply choosing the nearest stop. It can choose a direct ride or a one-transfer journey.

Example conceptual output:

```text
Walk → Red Line → transfer → Green Line → Walk
```

The map draws the walking legs and the e-Vahan route sequence.

### 4. Alternatives

The app displays a few other candidates after the selected route. This is useful when two routes have similar estimated travel times.

### 5. Search history

Using **Save** stores a destination in the current browser. Settings contains **Delete search history**.

## Why map-tap routing is different

A random point on the map is not itself an e-Vahan stop. The algorithm therefore considers the nearest useful boarding stop and the nearest useful alighting stop across all route lines. This is why a destination can be a building, path or open campus point rather than an entry in the stop list.

The current candidate generator allows at most one transfer. That is enough to compare direct journeys and common two-line campus trips without making the browser perform an unnecessarily large search.

## Routing assumptions in v4

The current configuration uses: 

- walking speed: `1.35 m/s`
- estimated e-Vahan speed: `14 km/h`
- estimated initial wait: `4 min`
- estimated transfer penalty: `3 min`
- estimated stop dwell: `0.35 min`

These are **engineering assumptions**, not official e-Vahan timings. Change them in `app-config.js` when better measured values are available.

## Adding the actual timetable later

The next major upgrade is timetable-aware routing. The graph can then use a time-dependent cost:

`arrival time = walking time + next available trip departure + in-vehicle time + transfer wait + final walk`

At that point, the algorithm can choose between, for example, a slightly longer direct route and a shorter two-line route based on the actual next vehicle rather than a fixed waiting estimate.

The route data is already kept as ordered stop arrays in `iisc-eVahan.js`, so timetable data can be added without redesigning the user interface.

## OpenStreetMap services

The map uses Leaflet with OpenStreetMap raster tiles. The walking leg uses an OpenStreetMap-based OSRM foot routing service. OSRM supports route calculations from coordinates and profiles such as foot routing.

Public OSM services are not an unlimited private API. Keep normal interactive usage, do not bulk-download tiles, and keep the required OpenStreetMap attribution visible.

## Coordinate accuracy

The bundled campus anchors are intended to make the prototype routable without a 25-stop geocoding process. Some are approximate campus anchors and should be verified against an authoritative IISc map before treating the app as an operational navigation system.

A good next step is to replace every anchor with a verified coordinate collected once from an authoritative IISc/OpenStreetMap source. Those values can then be committed into `STOP_COORDS` so no runtime geocoding is necessary.

## Common problems

### “My location” does not work

Use the GitHub Pages HTTPS URL, allow location permission in the browser, and check that Location is enabled on the phone.

### The map is blank

Check internet access. OpenStreetMap tiles are loaded from the network.

### A walking route is not drawn

The public pedestrian routing service can be temporarily unavailable. The app still displays the algorithmic journey and its estimated times, but the exact walking geometry may be missing.

### The app still shows an older design

Hard-refresh the GitHub Pages site or clear the browser's cached site data. The service worker intentionally caches the app shell.

## Zero-cost scope

This project is designed to avoid paid Google Maps/Places APIs. GitHub Pages can host the static files, while the map/routing services are open-data services. The project does not require a custom domain, Android Studio, Play Store publishing or a paid server.

## Credits

- OpenStreetMap contributors
- Leaflet
- OSRM / OpenStreetMap-based routing infrastructure
- IISc campus route information supplied for this project
