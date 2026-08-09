# ParkingScout

A lightweight, static web app that shows a city's parking on a map and makes
clear which of it is **free** — part of the [ScoutPlatform](https://bymilossavic.com)
family. Parking comes from OpenStreetMap and EV chargers from Open Charge Map,
prepared per city into JSON files the app reads directly. The app itself never
calls an API at runtime, so it loads instantly and costs nothing to host.

## Live

Deployed on GitHub Pages: `https://milossavicgit.github.io/ParkingScout/`
(and linked from bymilossavic.com).

## What it shows

- **Free** parking (black) — shown first, because that's the question drivers have
- **Paid** parking (purple)
- **Not recorded** (blue) — parking OSM knows about but hasn't priced; never
  assumed free
- **Park & ride** (green)
- **⚡ EV charging** — from Open Charge Map, with operator, charging cost, and
  whether you also pay for parking on top of charging
- **Time-limited** spots carry an amber dashed outline and an hour tag (1h / 2h)
  — a reminder to set the parking disc
- Customers-only **business parking is hidden**, so "free" means free
- City search, Google Maps links, and honest "check the signs" caveats where the
  map data is thin

## Repository layout

```
ParkingScout/
├── index.html            the whole app (no build step, no API keys)
├── cities.json           list of built cities the app shows in the picker
├── cities/               prepared parking per city (from parking_probe / build_italy)
│   ├── rome-41485.json
│   └── copenhagen-2192363.json
├── chargers/             prepared EV chargers per city (from build_chargers)
│   └── <same-slug>.json
├── parking_probe.py      build one city's parking
├── build_italy.py        bulk, resumable parking build for all of Italy
├── build_chargers.py     EV charger layer from Open Charge Map
└── README.md
```

Each city has a matching pair: `cities/<slug>.json` (parking) and, optionally,
`chargers/<slug>.json` (chargers). The app loads a city's chargers by convention
from the same filename under `chargers/`.

## Run it locally

Browsers block a page from `fetch`-ing local files, so serve the folder:

```
python -m http.server
# open http://localhost:8000
```

## The data pipeline

Data is prepared at build time, never at runtime.

**One city's parking** — resolve by OSM relation id to avoid same-name places
(e.g. "Copenhagen" also exists in New York):

```
python parking_probe.py "Copenhagen" --rel 2192363 --save
python parking_probe.py "Lisbon" --country pt --save
```

Then copy `out/<slug>.json` into `cities/` and add a line to `cities.json`
(or let `build_italy.py` manage it).

**All of Italy** — resumable, biggest city first, safe to Ctrl+C and rerun:

```
python build_italy.py                 # everything
python build_italy.py --min-pop 20000 # a smaller first pass
python build_italy.py --status        # progress
```

**EV chargers** — for every city already in `cities/`, pull chargers from Open
Charge Map into `chargers/`:

```
python build_chargers.py --key YOUR_OCM_KEY
# or set OCM_API_KEY once and just run: python build_chargers.py
```

Get a free Open Charge Map key at <https://openchargemap.org/site/develop/>.

### Keeping it fresh

OSM and OCM change slowly; re-running the builders every few months and
redeploying is enough. Each file is stamped with a build date shown in the app.
`--refresh` re-fetches existing cities.

## Deploy (GitHub Pages)

It's a static site — push the repo and turn on Pages:

```
git add .
git commit -m "ParkingScout"
git push
# repo → Settings → Pages → deploy from branch → / (root)
```

The app uses only relative paths, so it works at any URL without changes.

## Data sources & attribution

- Parking: **© OpenStreetMap contributors** (ODbL). Prepared with the Overpass API.
- EV chargers: **© Open Charge Map contributors** (CC BY 4.0).
- Map tiles: OpenStreetMap raster. Map engine: MapLibre GL JS.

## Honesty

ParkingScout only reflects what the open data records. Untagged parking is shown
as *unknown*, never assumed free; customers-only lots are hidden; time limits and
"parking here is paid" are surfaced where known. Where a city is lightly mapped,
the app says so and tells you to check the signs. A wrong "free" costs a ticket.
