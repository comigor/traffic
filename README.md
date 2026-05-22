# Live Traffic ETA Kiosk

A single-file, zero-dependency public display kiosk for showing live Google Maps ETAs from a central HQ to a list of places. Designed for wall displays with a dark, transit-board aesthetic.

## How to run locally

No build step required. Just serve the directory:

```bash
python3 -m http.server 8000
```

Then open your browser to:
`http://localhost:8000/?key=YOUR_GOOGLE_MAPS_API_KEY`

## Configuration

Edit the top of the `<script>` tag in `index.html` to configure your `HQ` and `PLACES`:

```javascript
const HQ = { name: 'HQ', address: '...' };
const PLACES = [
  { name: 'Airport',    icon: 'plane',  address: '...', dir: 'both',    mode: 'driving' },
  { name: "Fabricio's", icon: 'home',   address: '...', dir: 'both',    mode: 'transit' },
  // ...
];
```

### Place fields

| Field | Required | Values | Notes |
|---|---|---|---|
| `name` | yes | any string | Displayed as the card title |
| `icon` | yes | `plane` `park` `soccer` `subway` `home` | Inline SVG; see below to add more |
| `address` | yes | any string | Passed directly to the Routes API |
| `dir` | yes | `'from-hq'` `'to-hq'` `'both'` | Which directions to request and display |
| `mode` | yes | `'driving'` `'transit'` | Travel mode. Driving uses `computeRouteMatrix` (batched, Advanced SKU). Transit uses `computeRoutes` per pair (Essentials SKU, 10k/month free tier) to extract the transit line name and walking time. Each transit place adds ~2 calls per refresh. Transit rows show a line-name chip (colored if the API returns a line color) and walking time in the sub-label. |

### Adding a new icon

1. Find or create a 24x24 SVG path (Material Symbols or similar).
2. Add a `<symbol id="icon-NAME" viewBox="0 0 24 24"><path d="..."/></symbol>` entry inside the `<svg><defs>` sprite block near the top of `<body>` in `index.html`.
3. Reference it with `icon: 'NAME'` in the `PLACES` entry.

## Refresh schedule

The kiosk uses a time-window schedule in **BRT (America/Sao_Paulo)**. Windows and intervals are hardcoded in the `SCHEDULE` constant near the top of `index.html` and are easy to edit.

| Window | Hours (BRT) | Interval |
|---|---|---|
| Peak | 08:00 – 09:30 | `?refresh=` param (default 10 min) |
| Peak | 11:30 – 13:00 | `?refresh=` param (default 10 min) |
| Peak | 18:30 – 21:00 | `?refresh=` param (default 10 min) |
| Off-peak | all other hours within 06:00 – 22:00 | 60 min (hardcoded) |
| Sleep | 22:00 – 06:00 | no API calls |

During the sleep window the kiosk shows the last known data and displays "PAUSED / Resumes 06:00 BRT" in the top bar.

## URL Parameters

*   `key` (Required): Your Google Maps API Key.
*   `refresh`: Peak-window refresh interval in minutes (default `10`, clamped 1–30). Has no effect on the off-peak 60-minute cadence or the sleep window.
*   `units`: Set to `imperial` or `metric`. Default is `metric`.

Example:
`/?key=AIza...&refresh=5&units=imperial`

## Google Maps API Key Setup

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Enable the **Maps JavaScript API** and the **Routes API**.
3. Generate an API Key.
4. **Important**: Restrict your API key to prevent unauthorized use. Under "Application restrictions", choose "HTTP referrers" and add your GitHub Pages domain (e.g., `https://yourusername.github.io/traffic/*`).

## Estimated API cost

Each refresh fires parallel calls. With the current 8-place list (7 driving + 1 transit, all `dir: "both"`):

**Driving** (`computeRouteMatrix`, Advanced SKU — $10/1000 elements):
- 2 calls: 1×7 elements (HQ → driving) + 7×1 elements (driving → HQ) = 14 elements/refresh

**Transit** (`computeRoutes`, Essentials SKU — $5/1000 calls, 10k/month free):
- 2 calls: 1 call each direction per transit place = 2 calls/refresh

At stated rates:

- **Peak** (~4.5 h/day, 10-min interval, ~27 refreshes): 378 driving elements + 54 transit calls/day
- **Off-peak** (~11.5 h/day, 60-min interval, ~11 refreshes): 154 driving elements + 22 transit calls/day
- **Sleep**: 0

**Total: ~532 driving elements/day ≈ $0.0053/day + ~76 transit calls/day well within free tier. (~$1.94/year for driving alone.)**
